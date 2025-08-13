**Accounting Statement**
**Rate Assumed:** \$80/hour

**Assumptions**

* Scope covers a multi-role marketplace (buyer, vendor, admin) including vendor onboarding, promotions, checkout, logistics/shipping, notifications, payouts, and a Supabase-based backend.
* Backend includes edge functions for payments, shipping carriers, notifications, promotions, and payouts.
* API integration hours are part of the backend total (detailed separately for transparency to avoid double-counting).

---

### **Executive Summary**

* **UI/UX Design:** 140 hours — \$11,200
* **Frontend Development:** 360 hours — \$28,800
* **Backend Development (incl. API integrations):** 260 hours — \$20,800
* **Database Development:** 110 hours — \$8,800
  **Estimated Total:** 870 hours — **\$69,600**

---

### **UI/UX Design**

* **Activities:** Information architecture, buyer/vendor/admin flows, component system, responsive layouts, dashboards, promotion tools, onboarding wizards, empty/error states.
* **Hours:** 140
* **Amount:** \$11,200

---

### **Frontend Development**

* **Activities:** React/Vite/TypeScript setup, routing, state management, protected routes, shadcn/ui components, forms (react-hook-form + zod), notifications (toast/sonner), vendor/admin dashboards, carts/checkout, promotions UI, logistics screens, integration tests setup.
* **Hours:** 360
* **Amount:** \$28,800

---

### **Backend Development (Edge Functions & Business Logic)**

* **Activities:** Stripe checkout/verification, bank transfer verification, payouts processing, shipping rates (UPS/FedEx/DHL/PostNet) & shipment creation, notifications (email/SMS), promotions analytics/lifecycle, support/contact flows, commission tracking, vendor payout records, admin tools.
* **Hours:** 260
* **Amount:** \$20,800
* **Breakdown:** \~206 hours API integrations (see below), \~54 hours custom logic, orchestration, error handling, and admin tools.

---

### **Database Development**

* **Activities:** Schema modeling (orders, items, promotions, payouts, notifications, support), RLS policies, indexes, Supabase types, migrations, seed/test data alignment.
* **Hours:** 110
* **Amount:** \$8,800

---

### **API Integrations** *(included in Backend total; listed for transparency)*

* **Stripe (checkout, verification, customer portal):** 36 hrs — \$2,880
* **Supabase Auth (email/SMS, protected routes, sessions):** 16 hrs — \$1,280
* **Email delivery (auth, welcome, order, support):** 12 hrs — \$960
* **SMS delivery (codes, notifications):** 14 hrs — \$1,120
* **Shipping carriers & logistics:**

  * DHL rates: 12 hrs — \$960
  * FedEx rates: 14 hrs — \$1,120
  * UPS rates (incl. OAuth): 20 hrs — \$1,600
  * FedEx shipment: 10 hrs — \$800
  * UPS shipment: 10 hrs — \$800
  * PostNet rates: 8 hrs — \$640
  * Live rates aggregator: 8 hrs — \$640
    **Subtotal (Shipping):** 82 hrs — \$6,560
* **Promotions engine (recommendations, lifecycle, notifications):** 18 hrs — \$1,440
* **Payouts (escrow release/processing):** 16 hrs — \$1,280
* **Security/UX enhancement (password breach check):** 4 hrs — \$320
* **Support notifications/contact form:** 8 hrs — \$640
  **API Integrations Subtotal:** 206 hrs — \$16,480

---

### **Grand Totals**

* **Total Hours:** 870
* **Total Amount:** **\$69,600**

---

**Notes & Caveats**

* Estimates are based on the current codebase, typical complexity, and similar projects. Actual hours may vary with revisions, testing depth, or scope changes.
* This statement can be reformatted into an invoice-style PDF or adjusted to match internal timesheet allocations if required.

