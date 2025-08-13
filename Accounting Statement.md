**Accounting Statement**
**Rate:** \$80/hour
**Basis:** Estimates reflect the current size and complexity of the codebase, integrations, and related deliverables.

---

### **Summary (USD)**

| Area                 | Hours   | Rate    | Amount       |
| -------------------- | ------- | ------- | ------------ |
| UI/UX                | 100     | \$80/hr | \$8,000      |
| Frontend Development | 240     | \$80/hr | \$19,200     |
| Backend Development  | 200     | \$80/hr | \$16,000     |
| Database Development | 60      | \$80/hr | \$4,800      |
| **Total**            | **600** | —       | **\$48,000** |

---

### **Detailed Breakdown**

**UI/UX (100h = \$8,000)**

* Information architecture, wireframes, component library usage
* Responsive layouts, accessibility, theming
* Onboarding flows, analytics/report UX

**Frontend (240h = \$19,200)**

* React pages/components, forms & validation, i18n
* Charts/dashboards, budgeting/deductions/reports
* Routing, error boundaries, offline/mobile optimizations
* Performance tracking, PDF/receipt handling UI, payments UI

**Backend (200h = \$16,000)**

* Supabase Edge Functions, webhook integrations (Plaid, Yodlee, Twilio, payments)
* Scheduled jobs (reminders), monitoring hooks
* File processing (OCR/PDF), fraud detection heuristics, authentication flows

**Database (60h = \$4,800)**

* Schema design, migrations, indexing
* Row-Level Security (RLS) policies, storage buckets, performance tuning

---

### **API & Platform Integrations**

(*Included in Frontend/Backend totals; shown for clarity*)

* Supabase (Auth, DB, Storage, Edge Functions): 60h = \$4,800
* Payments (Stripe Checkout/subscriptions): 32h = \$2,560
* Plaid: 36h = \$2,880
* Yodlee: 28h = \$2,240
* Twilio SMS: 20h = \$1,600
* Monitoring/telemetry (New Relic, Sentry): 18h = \$1,440
* OCR & document parsing (Tesseract.js, PDF.js): 36h = \$2,880
* i18n (4 locales): 20h = \$1,600
* Analytics dashboards: 16h = \$1,280
* Capacitor mobile shell (iOS/Android): 16h = \$1,280
* Voiceover/TTS service integration: 16h = \$1,280
* Calendars/reminders email functions: 12h = \$960
* Fraud detection heuristics: 10h = \$800
* Bank data import parsers (CSV, PDF): 20h = \$1,600

---

### **Assumptions**

* Scope is based on the current repository: multi-module frontend, numerous Supabase Edge Functions, multiple external integrations.
* Hours include build, testing, and documentation for each area.
* Ongoing maintenance, infrastructure costs, and third-party service fees (e.g., Stripe, Twilio, New Relic) are excluded and billed separately.

