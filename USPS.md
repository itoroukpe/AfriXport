Awesome—here’s a **clear, do-this-then-that** plan to wire USPS into Afrixport for the three APIs you named. I’m keeping it vendor-blessed (new USPS Developer Portal, OAuth 2.0) and calling out anything you must set up first.

# 0) Prereqs (one-time)

1. **Create a USPS Developer Portal account**, then **register an app**. ([developer.usps.com][1])
2. **Enroll in USPS Ship** and **get an Enterprise Payment Account (EPA)**; Labels won’t work without these. (USPS says Labels APIs require Ship + EPA.) ([developers.usps.com][2])
3. **Get OAuth 2.0 credentials** (client id/secret) in the portal. All new USPS APIs use OAuth Bearer tokens. ([developer.usps.com][3])
4. Note that the legacy “Web Tools” platform is being retired; build on the **new** APIs only. ([USPS][4])

---

# 1) International Labels 3.0 (create label + customs forms)

**What it does:** Creates USPS international labels (e.g., **Priority Mail Express International**, **Priority Mail International**, **First-Class Package International Service**), calculates postage, and generates the required **Shipping Services File** (Publication 199). Output label formats: **PDF/TIFF**. ([developer.usps.com][5], [developers.usps.com][6])

## Steps

A) **Collect required data from the order**

* Shipper (from/EPA account), recipient (to), phone/email
* Package: weight (lbs/oz or kg/g), dimensions, value, insurance flag
* Service: `PRIORITY_MAIL_EXPRESS_INTERNATIONAL` (or PMI/FCPI)
* **Customs line items**: description, quantity, unit value, **HS/HTS code**, country of origin, total weight per item
* Contents type (merchandise/gift/etc.), **incoterm** (if you model it), non-delivery option, and **export control** fields when applicable (e.g., EEI exemption)
  (USPS public customs guidance confirms customs form requirements; Labels 3.0 will generate the proper CN22/CN23 data in the label stream.) ([USPS][7])

B) **Get an OAuth token** and cache it (e.g., 55 min TTL). All calls include `Authorization: Bearer <token>`. ([developer.usps.com][3])

C) **Call International Labels 3.0 – Create Label**

* Request body: shipper, recipient, package, service, customs details.
* Response: base64/PDF (or URL) for the label, **Tracking/IMpb** number, **postage**, and **Shipping Services File** metadata. (Per API catalog.) ([developer.usps.com][5], [developers.usps.com][6])

D) **Persist & deliver**

* Store: label pdf (or URL), tracking number, SSF id, postage, and request/response JSON.
* Expose a **“Print Label”** link in Afrixport + attach tracking to the order.

> Tip: If you must reference field names while you wait for portal access, USPS publishes “Web Tools → New API mapping” spreadsheets. They help translate old request fields to the new schemas. ([USPS][8])

---

# 2) Carrier Pickup 3.0 (schedule pickups for Express/international)

**What it does:** Schedules a carrier pickup at the ship-from address for eligible services (including international/Express). (Modern replacement for legacy “Package Pickup” APIs.) ([developers.usps.com][9])

## Steps

A) **Confirm eligibility** (pickup ZIP + service) via the Carrier Pickup 3.0 check endpoints (visible after login). ([developers.usps.com][9])
B) **Create a pickup** for a date window with: contact, phone/email, package location (“front porch”, “reception”), special instructions, **count of packages by class** (incl. Express/Intl), and optional earliest/latest times.
C) **Store pickupId** returned by USPS; surface “Cancel/Modify pickup” actions in Afrixport.

> Need request shape while gated? The legacy **Package Pickup API** PDF shows the core fields (Schedule, Availability, Cancel). The new API is similar conceptually; use this only as a field checklist while building your UI. ([USPS][10])

---

# 3) Tracking 3.0 (Track/Confirm without an aggregator)

**What it does:** Returns **summary or detailed scan events** (date/time/location) for a USPS package. Use it to power your **order timeline** and notifications. ([developer.usps.com][11])

## Steps

A) **When you create a label**, capture the **tracking number**.
B) **Polling plan:** call Tracking 3.0 at key intervals (e.g., every 6h until first movement, then every 12–24h; stop on “Delivered/Final”).
C) **Map events → Afrixport statuses** (Acceptance, Departure, Arrival, Customs, Out for Delivery, Delivered, Exception). Store scan array + lastEventAt for diffing.
D) **Notify buyers/vendors** on status transitions (webhooks → email/SMS/in-app).

> If later you want multi-carrier/EMS networks in one feed, you can still layer an aggregator; but this is the official USPS endpoint for USPS-origin traffic. ([developer.usps.com][11])

---

# 4) Recommended Afrixport implementation (concrete)

## A. Config & secrets

* `USPS_CLIENT_ID`, `USPS_CLIENT_SECRET`, `USPS_BASE_URL`, `USPS_SCOPES`
* `USPS_EPA_ACCOUNT` (Enterprise Payment Account id)
* Keep **sandbox vs production** by environment variable. (The dev portal provides both.) ([developer.usps.com][1])

## B. Minimal data model changes

* `shipment_labels(id, order_id, carrier, service, tracking, label_url_or_b64, postage_cents, ssf_id, usps_shipment_id, status, created_at)`
* `shipment_customs_lines(label_id, description, hs_code, origin_country, qty, unit_value_cents, net_weight_grams)`
* `carrier_pickups(id, shipper_address_id, date, earliest, latest, package_location, instructions, pickup_id, status)`
* `tracking_events(id, tracking, status_code, status_text, city, state, country, event_time, raw_json)`

## C. API flow (pseudo)

1. **Auth**: `POST /oauth/token` (portal provides URL) → cache token. ([developer.usps.com][3])
2. **Create intl label**: `POST /shipping/international/labels` → save label + tracking. (Per API catalog description.) ([developer.usps.com][5])
3. **Schedule pickup**: `POST /carrier-pickup/requests` → save `pickupId`. ([developers.usps.com][9])
4. **Track**: `GET /tracking/v3/{trackingNumber}` → upsert events. ([developer.usps.com][11])

---

# 5) Example server handlers (TypeScript/Node—drop-in pattern)

> These mirror the four calls above. Replace paths/field names with the exact ones from the portal when your app is approved.

```ts
// usps.ts
import fetch from "node-fetch";

const BASE = process.env.USPS_BASE_URL!;
const ID = process.env.USPS_CLIENT_ID!;
const SECRET = process.env.USPS_CLIENT_SECRET!;

let tokenCache: { token?: string; exp?: number } = {};

async function getToken() {
  const now = Math.floor(Date.now() / 1000);
  if (tokenCache.token && tokenCache.exp && tokenCache.exp - now > 60) return tokenCache.token;

  const resp = await fetch(`${BASE}/oauth/token`, {
    method: "POST",
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
    body: new URLSearchParams({
      grant_type: "client_credentials",
      client_id: ID,
      client_secret: SECRET,
      // scope if required by your USPS app; leave off if not issued
    }),
  });
  const data = await resp.json();
  tokenCache = { token: data.access_token, exp: now + (data.expires_in ?? 3600) };
  return tokenCache.token!;
}

export async function createIntlLabel(payload: any) {
  const token = await getToken();
  const resp = await fetch(`${BASE}/shipping/international/labels`, {
    method: "POST",
    headers: { Authorization: `Bearer ${token}`, "Content-Type": "application/json" },
    body: JSON.stringify(payload),
  });
  if (!resp.ok) throw new Error(await resp.text());
  return resp.json(); // expect label (PDF/TIFF), tracking, postage, SSF metadata
}

export async function schedulePickup(payload: any) {
  const token = await getToken();
  const resp = await fetch(`${BASE}/carrier-pickup/requests`, {
    method: "POST",
    headers: { Authorization: `Bearer ${token}`, "Content-Type": "application/json" },
    body: JSON.stringify(payload),
  });
  if (!resp.ok) throw new Error(await resp.text());
  return resp.json(); // expect pickupId, status
}

export async function trackUSPS(tracking: string) {
  const token = await getToken();
  const resp = await fetch(`${BASE}/tracking/v3/${encodeURIComponent(tracking)}`, {
    headers: { Authorization: `Bearer ${token}` },
  });
  if (!resp.ok) throw new Error(await resp.text());
  return resp.json(); // expect events[]
}
```

**Example label payload skeleton** (fill from your order/customs UI):

```json
{
  "shipper": { "name": "Afrixport Fulfillment", "address1": "...", "city": "...", "state": "WA", "zip": "98101", "phone": "206...", "email": "ops@afrixport.com", "epaAccount": "EPA123456" },
  "recipient": { "name": "Kobi Health", "address1": "...", "city": "Lagos", "country": "NG", "phone": "+234..." },
  "package": { "weight": { "unit": "lb", "value": 2.4 }, "dimensions": { "unit": "in", "l": 12, "w": 9, "h": 4 } },
  "service": "PRIORITY_MAIL_EXPRESS_INTERNATIONAL",
  "customs": {
    "contentsType": "MERCHANDISE",
    "nonDeliveryOption": "RETURN",
    "items": [
      { "description": "Cotton shirts", "hsCode": "610510", "originCountry": "US", "quantity": 3, "unitValue": 25.00, "netWeight": { "unit": "g", "value": 600 } }
    ],
    "totalValue": 75.00
  }
}
```

**Example pickup payload skeleton:**

```json
{
  "address": { "name": "Afrixport Fulfillment", "address1": "...", "city": "Seattle", "state": "WA", "zip": "98101", "phone": "206..." },
  "date": "2025-09-08",
  "earliestTime": "09:00",
  "latestTime": "17:00",
  "packageLocation": "Front Desk",
  "specialInstructions": "Ring bell at Suite 200",
  "packageCounts": { "priorityMailExpress": 1, "priorityMailInternational": 0, "firstClassPackageInternationalService": 0 }
}
```

---

# 6) Operational best practices

* **Validation**: Require **HS codes** + detailed descriptions in your seller UI to reduce customs exceptions. USPS stresses detailed item descriptions on customs forms. ([USPS][7])
* **Idempotency**: Use your `orderId` as an idempotency key when creating labels; if a call times out, don’t double-charge.
* **Label lifecycle**: Allow **void/refund** flows (expose in your admin).
* **Tracking cadence**: Poll often in the first 48h, then back off. Store the full scan history. ([developer.usps.com][11])
* **Migration note**: Avoid starting new work on legacy Web Tools; USPS has a formal retirement timeline. ([USPS][4])

---

# 7) Where to click in USPS docs (for your approval flow)

* **International Labels 3.0** (API catalog entry; details unlocked after login) — feature overview & Publication 199 output. ([developer.usps.com][5], [developers.usps.com][6])
* **Carrier Pickup 3.0** (overview; endpoints behind login). ([developers.usps.com][9])
* **Tracking 3.0** (public overview page). ([developer.usps.com][11])
* **OAuth 2.0** (how to obtain tokens; app registration). ([developer.usps.com][3])
* **Getting Started** (Ship + EPA requirement for Labels). ([developers.usps.com][2])
* **Web Tools Mapping** (handy for field naming during buildout). ([USPS][8])

---

If you want, I can tailor the payloads and DB migrations to your current Afrixport stack (Supabase/Edge functions vs. Spring Boot) and provide ready-to-paste code for either path.

[1]: https://developer.usps.com/home?utm_source=chatgpt.com "USPS Developer Portal"
[2]: https://developers.usps.com/getting-started?utm_source=chatgpt.com "Getting Started | devportal"
[3]: https://developer.usps.com/oauth "OAuth 2.0 | devportal"
[4]: https://www.usps.com/business/web-tools-apis/documentation-updates.htm?utm_source=chatgpt.com "Web Tools Documentation & Updates"
[5]: https://developer.usps.com/apis?utm_source=chatgpt.com "API Catalog | devportal"
[6]: https://developers.usps.com/apis?utm_source=chatgpt.com "API Catalog | devportal"
[7]: https://www.usps.com/international/customs-forms.htm?utm_source=chatgpt.com "U.S. Customs Forms"
[8]: https://www.usps.com/business/web-tools-apis/?utm_source=chatgpt.com "Web Tools APIs"
[9]: https://developers.usps.com/carrierpickupv3?utm_source=chatgpt.com "Carrier Pickup 3.0 | devportal"
[10]: https://www.usps.com/business/web-tools-apis/package-pickup-api.pdf?utm_source=chatgpt.com "Package Pickup API"
[11]: https://developer.usps.com/trackingv3?utm_source=chatgpt.com "Tracking 3.0 | devportal"
