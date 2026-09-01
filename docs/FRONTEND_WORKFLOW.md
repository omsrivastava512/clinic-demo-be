# WORKFLOW.md — Frontend Workflow Documentation (5-Step OPD Visit Flow)

**Scope:** Daily Ledger → New Patient Registration → Complaint Selector → Procedure Logger → Invoice & Payment. This does **not** cover Patient Profile or Dashboard — those are separate modules, undocumented here.

**Basis:** the workflow description + 5 screenshots (Aug 14 2026), cross-referenced against `supabase_migration.md` (v11, canonical), `README.md` (client business-rules log), and the existing `schema_reconciliation_audit.md` / `frontend-schema-audit-handover.md` findings. Where this document adds something *new* beyond those two audits, it's explicitly marked — screenshots and prompt prose are weaker evidence than the line-numbered component audit, since I don't have the actual component code for these five screens.

**Wiring status:** 100% mock data end to end. Nothing below is connected to Supabase yet.

**Legend:** `MOCK` = catalog-phase, hardcoded/simulated · `NOT YET WIRED` = no live Supabase call exists · `PLANNED` = named as upcoming in the workflow description, not built · `CONFIRMED` = matches a finding already in `schema_reconciliation_audit.md` · `NEW` = surfaced by this pass, not previously documented · 🔴/🟡/🟢 = structural / safety-net / performance, same tagging convention as `backend-schema-iterations.md`.

---

## Section 1: Workflow Overview

| Step | Screen | Initiated By | Reads (SELECT) | Writes (INSERT/UPDATE) | Status |
|---|---|---|---|---|---|
| 0 | Daily Ledger (home) | App load / receptionist | `patients` (search), `visits` (today's rows, implied) | — (read-only) | `MOCK` — array of ~22 patients + hardcoded ledger rows |
| 1 | New Patient Registration | "Register New Patient" from ledger search | — | `patients` (INSERT), `clinical_notes` (INSERT, 0+ rows) | `MOCK` — form submits to local state, no persistence |
| 2 | Complaint Selector | Existing patient selected, or returned from Step 1 | `complaint_courses` (active, this patient), `complaint_catalog` (search) | `complaint_courses` (INSERT for new complaint) | `MOCK` |
| 3 | Procedure Logger | Complaint(s) confirmed in Step 2 | `services` (+ `clinic_service_prices`, unconfirmed) | `visits` (INSERT, timing TBD), `visit_services` (INSERT, timing TBD) | `MOCK` — billing math explicitly disclaimed as inaccurate in the workflow description itself |
| 4 | Invoice & Payment | "Create Invoice" from Step 3 | Carried-forward Step 3 state | `invoices` (INSERT, timing = open Decision 1) | `MOCK` — "most underdeveloped part of the frontend" per the workflow description |

---

## Section 2: Step-by-Step Detail

### Step 0 — Daily Ledger (Home Screen)
*Screenshot 1: "Daily Register", Dr. Sharma's Clinic, OPD Count 22, live search panel.*

**Current State:** Static table of today's entries (time, patient, treatment, status badge) styled as a paper logbook. A bottom search bar filters a mock patient array in real time and offers "Register New Patient: '{typed text}'" when nothing matches.

**Intended Behavior:** Live, clinic-scoped view of the day's OPD activity; search resolves to an existing patient (→ Step 2) or confirms a new one (→ Step 1).

**Backend Interaction Points:** Patient search maps cleanly onto `idx_patients_full_name_trgm` (pg_trgm) and `idx_patients_phone` — the two search modes named in the catalog features line up with the two indexes that already exist for exactly this purpose. "OPD Count" is presumably `count(*) from visits where clinic_id = X and date = current_date` — reasonable, not yet confirmed clinic-scoped. Full read/write list in §3.

**Frontend Assumptions:**
- Assumes any patient relevant to this branch is findable by a flat search — see Decision 4, this doesn't hold once the real staff RLS policy (drafted, commented out) goes live.
- The three status badges shown (`Waiting`, `In Therapy`, `Paid`) imply a trackable in-progress state that has no home in the schema at all — see Decision 3, the standout finding of this pass.
- "Register New Patient" copy says "Create new MRN" — implies MRN auto-generation. No such mechanism exists in the schema today (see Decision 5).

**Open Questions:** Is OPD Count clinic-scoped or chain-wide? Does "grey-out for patients who've already completed a visit today" (named as upcoming) read `visits` directly, or something else? → Decisions 3, 4, 5.

---

### Step 1 — New Patient Registration
*Screenshot 2: Full Legal Name, Mobile/WhatsApp, Age, Sex (M/F/X), Address, Referral Source (Walk-In selected).*

**Current State:** Form fields visible: name, phone, **age** (not DOB), sex (M/F/X buttons), address, referral source (Walk-In / Google Maps / Dr. Referral). "Create Profile & Proceed" returns the user straight to Step 2 for the new patient. Catalog description also names a "Clinical Notes Builder" on this screen with auto-categorized observations, including "fall risk."

**Intended Behavior:** `patients` INSERT, optionally followed by `clinical_notes` INSERT(s) for whatever the Notes Builder captured.

**Backend Interaction Points:** Maps to `patients.full_name`, `patients.phone`, `patients.address`, `patients.gender`, `patients.referral_mode`. `clinical_notes` needs `patient_id` — meaning the notes write can only fire *after* the patients INSERT returns an id, not as one flat payload. Full list in §3.

**Frontend Assumptions:**
- **`CONFIRMED`** — captures a 2-digit **age**, not `date_of_birth date`. This is already documented in `schema_reconciliation_audit.md` (`DemographicsSection.tsx` L45–55, "Blocks integration"). The screenshot confirms it's still the live state, and the workflow description's own "Upcoming: DOB picker" line shows this is already a known, planned fix — good sign, not a fresh discovery.
- Sex buttons show M/F/X, which *does* match `patients.gender`'s CHECK values exactly — that part is fine. The already-documented issue is one level deeper: the underlying form field is named `sex`, not `gender` (`patient/types/index.tsx` L11) — invisible in a screenshot, but worth restating here since this is where it happens.
- No clinician selector anywhere on this form. **`CONFIRMED`** — matches `GAPS.md`: "intake registration form has no clinician assignment field." Every new patient will insert with `clinician_id = NULL` under the current shape — an accepted scope decision per that doc, not a bug, but worth restating for anyone wiring this form.
- **`NEW`** — "Clinical Notes Builder" auto-categorizing observations including "fall risk" is meaningful evidence for the open question in `LOGS.md`: whether `patient_alerts` (which has its own `type = 'FALL_RISK'` enum value) has *any* live write path, or whether everything routes through `clinical_notes.category` as free text instead. The workflow description names this a *Notes* builder, not an *Alerts* builder, and "fall risk" is listed as one of its auto-categorized labels — that's evidence pointing toward `clinical_notes`, not proof. Worth a direct check against the actual component when it's available.

**Open Questions:** Does selecting "Dr. Referral" reveal a required text input for `referral_doctor_info`? Not visible in this screenshot (Walk-In is selected) — needed to satisfy `chk_referral_doctor_info`, which requires it non-null exactly when `referral_mode = 'DOCTOR'`.

---

### Step 2 — Complaint Selector
*Screenshot 3: Priya Kapoor, MRN-9921, 3 active complaints, "SELECT VISIT COMPLAINTS (MULTIPLE ALLOWED)", catalog search with region chips.*

**Current State:** Lists this patient's `status = 'Active'` complaint courses with checkboxes — confirmed, the screenshot shows only Active-labelled ("Active Plan") complaints, consistent with the catalog description's "resolved complaints excluded." Multiple can be checked at once. A catalog search below offers existing `complaint_catalog` entries, filterable by region chips that match `complaint_catalog.region`'s CHECK values exactly: Spine, Shoulder, Knee, Hip, Elbow, Ankle, Neuro — **checked, no mismatch found** on the display side.

**Intended Behavior:** Confirming this screen should, per the trigger-ordering rule, fire a `complaint_courses` INSERT for any brand-new complaint *before* the flow can legally proceed — that INSERT is what silently provisions `patient_clinic_access` via `process_new_complaint_course()`.

**Backend Interaction Points:** `complaint_courses` (read: active ones for `patient_id`; write: new ones), `complaint_catalog` (read, search). Full list in §3.

**Frontend Assumptions:**
- **`CONFIRMED`, now with concrete visual proof** — "MULTIPLE ALLOWED" is the literal UI copy, and Screenshot 4 (Step 3) shows a real 3-complaint session flowing through together. `invoices.visit_id` is a single FK; three complaints selected here means three separate `visits` rows downstream, feeding one invoice. This is exactly the divergence already flagged in `GAPS.md` and `schema_reconciliation_audit.md` — this screen is where that "session" actually gets defined, and nothing downstream of it persists that grouping anywhere. See Decision 2.
- Whether the new-complaint write actually fires *at* this screen, or is deferred to a later combined submit, is unconfirmed — this is the direct, concrete instance of the already-flagged "Blocks integration" finding (`ComplaintSelector/index.tsx`, `vistitworkflow.tsx`): if it's deferred, Step 3/4 will attempt `visits` inserts against a patient+clinic pair with no `patient_clinic_access` row yet, and get a raw trigger exception instead of a friendly error.
- **`CONFIRMED`** — catalog-selected complaints are already known to drop `complaint_catalog_id` (`handleCatalogSelect`, `ComplaintSelector/index.tsx` L183–193). Not visible in a screenshot (the catalog id, e.g. `CAT_S01`, is never shown to the user) — restating here because this is the exact screen where it happens.
- "Returning Patient • Last visit: over 2 years ago" reads from `patients.last_visit_at`, which is chain-wide, not branch-specific (patients have no `clinic_id` — location-specific data lives on `visits`/`complaint_courses`). A receptionist could reasonably misread this as "last visit at my branch."

**Open Questions:** Confirm write timing → Decision 2. Confirm whether "last visit" framing is made explicitly chain-wide in the UI copy.

---

### Step 3 — Procedure Logger
*Screenshot 4: "Log Today's Procedures for Amit Trivedi", 3 complaint sections, 8 procedures incl. "Consultation ₹500", running bill = ₹1,700.*

**Current State:** One numbered section per selected complaint, each with its own procedure checklist pulled from a shared "Common Procedures" grid. A live "Current Session Bill" panel sums checked items grouped by complaint. The workflow description itself states this billing math is a placeholder summation and doesn't reflect real fee logic — that disclaimer is already given, not something this pass is discovering.

**Intended Behavior:** Per complaint, log which services were used; ultimately produce `visits` + `visit_services` rows with correct `is_charged` / `charged_amount_in_paise`, informed by the real fee rules in `README.md` §2 once those are implemented.

**Backend Interaction Points:** `services` (procedure list + base prices), possibly `clinic_service_prices` for per-clinic overrides (not visibly reflected — can't confirm from the screenshot alone). Writes: `visits` (one per complaint being treated), `visit_services` (one per logged procedure). Full list in §3.

**Frontend Assumptions:**
- **`CONFIRMED`** — the composite `complaintId::procedureId` keys and the `validateInvoiceItems` client-ID matching problem are already documented (`useProcedureLogger.tsx` L39–46, `ProcedureLogger/index.tsx` L11–16): if a complaint's client-side `crypto.randomUUID()` isn't reconciled with the real `complaint_courses.id` before this screen runs, logged items silently drop during validation.
- **`NEW`** — the billing panel treats **"Consultation"** as an ordinary selectable procedure, scoped per complaint, structurally identical to Ultrasonic/IFT/TENS/etc. In the screenshot, "Post-Op ACL Rehab" and "Femur Fracture" *each* carry their own separate ₹500 Consultation line — two consultation charges in one session. But `visits.consultation_fee_in_paise` is a dedicated snapshot column on `visits`, entirely separate from the `visit_services` junction — consultation was never meant to be "just another catalog service." This gap is now sharper than a generic "billing is WIP" note: `README.md` §2's new fee rule requires the exam fee to be charged **once per session, shared across every complaint treated that day** (₹350 once, then ₹200/300 per complaint) — the exact opposite of what's rendered here. See Decision 6.
- No visible signal of clinic-specific price overrides — can't confirm whether `clinic_service_prices` is even consulted, or whether every clinic sees identical `services.standalone_price_in_paise` values in this mock.

**Open Questions:** Confirmed by Decision 6 — how should "Consultation" be represented once the real fee rules land? Does this screen read `clinic_service_prices` at all today?

---

### Step 4 — Invoice & Payment
*Screenshot 5: `INV-2026-0306-088`, Amitabh Bachchan, ₹550, 2 procedures across 2 complaints, payment mode UPI/Cash/Card.*

**Current State:** Displays a pre-built invoice number, itemized procedure/cost breakdown (spanning more than one complaint), and a payment-mode selector, ending in "Confirm Payment." Named in the workflow description as the least-built screen. (Note: this screenshot's patient/total don't match Screenshot 4's — these are separate illustrative mock states, not one continuous session.)

**Intended Behavior:** Resolve Decision 1 below, then fire `invoices` INSERT with the confirmed line items, total, and payment mode.

**Backend Interaction Points:** Write: `invoices` (`amount_in_paise`, `payment_mode`, `payment_status`, `visit_id`). `invoice_number` and `clinic_id` are both trigger-owned, not client-supplied. Full list in §3.

**Frontend Assumptions:**
- **`CONFIRMED`, and the exact same value** — `schema_reconciliation_audit.md` already flags `vistitworkflow.tsx` L146 as hardcoding `'INV-2026-0306-088'`, and that is the literal string shown in this screenshot. This screen displays an invoice number *before* any DB write, when `process_new_invoice()` is what mints it, atomically, on INSERT.
  - **`NEW`, additional layer** — the mock string's *shape* doesn't match the trigger's output either. `process_new_invoice()` produces `'INV-' || year || '-' || 4-digit-zero-padded-counter`, e.g. `INV-2026-0001`. The mock value `INV-2026-0306-088` has an extra hyphenated segment — even once the timing question is fixed, the display format itself needs updating to match.
- **`CONFIRMED`** — the breakdown spans multiple complaints in one invoice, directly reinforcing the already-documented `invoices.visit_id`-is-singular divergence (`InvoiceBuilder/index.tsx`, `pages/vistitworkflow.tsx` L146–153).
- **`CONFIRMED`** — payment mode is visibly selectable (UPI/QR pre-selected, Cash, Card — matching `clinics.payment_methods_accepted`'s default array exactly), but the existing audit already found `FooterActions.tsx` L11 doesn't thread the selected `paymentMode` state through the confirm handler. The screenshot shows the *display* layer is fine; the gap is purely in the data flow to submission.
- "Pay later" (named as upcoming) would need to actually exercise `payment_status = 'Pending'`/`'Overdue'` — CHECK values that exist in the schema today but aren't touched by the MVP `'Paid'`-default flow.

**Open Questions:** Decision 1, directly.

---

## Section 3: Data Flow Map

| Moment in Workflow | DB Table | R/W | Triggered By | Status |
|---|---|---|---|---|
| Ledger renders | `patients`, `visits` (implied, for rows) | Read | Page load | NOT YET WIRED |
| Typing in ledger search | `patients` (trgm/phone index) | Read | Keystroke | NOT YET WIRED |
| "Register New Patient" click | — | — | Click | Navigation only |
| Submit registration form | `patients` | Write (INSERT) | "Create Profile & Proceed" | NOT YET WIRED |
| Submit registration form (if notes captured) | `clinical_notes` | Write (INSERT) | Same submit; must sequence *after* the patients INSERT resolves | NOT YET WIRED |
| Complaint Selector opens | `complaint_courses` | Read (active, this patient) | Screen mount | NOT YET WIRED |
| Typing in complaint search | `complaint_catalog` | Read | Keystroke | NOT YET WIRED |
| Confirming a new/custom complaint | `complaint_courses` | Write (INSERT) | Checkmark/confirm — **exact moment is Decision 2** | NOT YET WIRED |
| *(DB-side effect of the above)* | `patient_clinic_access` | Write (INSERT, trigger only) | `process_new_complaint_course()` | Never app-initiated — client roles never write here directly |
| Procedure Logger opens | `services`, `clinic_service_prices` (unconfirmed) | Read | Screen mount | NOT YET WIRED |
| Logging / confirming procedures | `visits` | Write (INSERT) | "Create Invoice" click, or earlier — **Decision 2** | NOT YET WIRED |
| Logging / confirming procedures | `visit_services` | Write (INSERT, one per procedure) | Same moment as the `visits` insert | NOT YET WIRED |
| Invoice screen renders | Carried-forward local state (not a fresh read) | Read (local) | Screen mount | NOT YET WIRED |
| "Confirm Payment" click | `invoices` | Write (INSERT) | Click — **exact moment is Decision 1** | NOT YET WIRED |
| *(DB-side effect of the above)* | `clinics.invoice_counter` | Write (UPDATE, atomic increment, trigger only) | `process_new_invoice()` | Trigger only |
| "Grey out patients seen today" *(planned)* | `visits` | Read (`patient_id` + `date = today`) | Ledger render | PLANNED, not built |

---

## Section 4: Unresolved Design Decisions

### Decision 1 — When does the invoice write actually fire?
**Priority: BLOCKER**

- **Question:** Does "Create Invoice" (end of Step 3) INSERT the `invoices` row immediately, with Step 4 as a post-write preview — or does Step 4 stay a pure preview until "Confirm Payment" does the actual INSERT?
- **What the frontend currently implies:** Neither, explicitly — this is named as an open question in the workflow description itself, not something the current mock UI has committed to either way.
- **What the backend needs to support either option:** `process_new_invoice()` is INSERT-only, `SECURITY DEFINER`, and atomically increments `clinics.invoice_counter` the instant the row is written — there's no decrement path anywhere in the schema. If the write fires on the Step 3 click and the user then backs out or abandons the flow, the counter has already burned a number: a gap appears in the clinic's invoice sequence (`INV-2026-0004` exists, orphaned, next real invoice becomes `INV-2026-0005`), and nothing short of a manual DELETE — which isn't discussed anywhere in the schema or RLS docs — removes the row, and even a DELETE wouldn't restore the counter.
- **Leaning:** Firing the write on final "Confirm Payment" is more consistent with how the rest of the schema already behaves — MVP philosophy throughout is "row exists = complete and paid," `payment_status` defaults to `'Paid'` immediately, and there's no draft/pending invoice state anywhere. This also matches the already-suggested fix for the `invoice_number` display problem (§2, Step 4): show a placeholder like "Auto-generated on confirmation" and only reveal the real number after the mutation response comes back.

---

### Decision 2 — When do `complaint_courses` and `visits` actually get written — one coordinated write, or progressively across Steps 2–4?
**Priority: BLOCKER**

- **Question:** Restates and sharpens the open item already logged in `LOGS.md`: "whether the visit→invoice write happens as one coordinated backend action... or as several progressive writes across the UI flow." That was previously unconfirmable against real code. It's still unconfirmed against real code — but the concrete 5-step flow now makes the shape of the question much sharper than it was in the abstract.
- **What the frontend currently implies:** The trigger-ordering rule is absolute — `complaint_courses` must exist before `visits` can insert for that patient+clinic pair, because that INSERT is what silently provisions `patient_clinic_access`. If Step 2's confirm action doesn't fire that INSERT immediately, every complaint selected there is just local state until some later combined submit — and the earliest that submit could safely include `visits` is once Step 3's procedure logging (which supplies `consultation_fee_in_paise` / `services_total_in_paise` — both `NOT NULL DEFAULT 0`, not nullable) is complete.
- **What the backend needs to know:** Whichever shape is chosen, `visits` cannot legally be written before `complaint_courses` for a first-time patient+clinic pair — that's not a preference, it's what `validate_visit_clinic_id()` enforces. This is also the direct, concrete version of the already-flagged "Blocks integration" finding around `ComplaintSelector`/`vistitworkflow.tsx` skipping an explicit `complaint_courses` write step.
- **Connects to:** the "session" question in `GAPS.md`/`README.md` §5 — Screenshot 3's "MULTIPLE ALLOWED" and Screenshot 4's 3-complaint bill are concrete proof that one Complaint Selector sitting produces *multiple* `visits` rows with no schema-level grouping between them at all today.

---

### Decision 3 — Does the Daily Ledger's Waiting → In Therapy → Paid status have anywhere to live?
**Priority: LATER (relative to core Steps 1–4 wiring) — but structurally significant, worth deciding deliberately rather than by accident**
**Tag: 🔴 Structural, if pursued**

- **Question:** The ledger's most prominent visual feature is a three-state status badge per row. The schema's MVP philosophy, stated directly in `supabase_migration.md`, is the opposite: *"Row existing = session complete + paid. No status lifecycle."* The `daily_ledger` view hardcodes `'Complete' as status` — a constant, not a derived value. Under the current model, a patient who is merely "Waiting" or "In Therapy" has **no row anywhere in the transactional schema** — they haven't reached Step 4 yet, and nothing gets written until they do.
- **What the frontend currently implies:** Either this status is tracked ephemerally, client-side only (meaning a page refresh or a second receptionist's device wouldn't see it — undermining the whole point of a shared, live OPD ledger), or it's simply hardcoded/randomized in the current mock with no real mechanism behind it at all.
- **What the backend needs to know to support either option:** A real trackable in-progress state means adding an actual status lifecycle — most likely a new `visits.status` column (or a lightweight parallel table for in-progress-but-not-yet-billed encounters) — which is a genuine 🔴 structural change, not a safety-net tweak, and cuts against the "row exists = done" assumption that `grand_total_in_paise`, `invoice_number`, and the whole single-touch MVP design are built around. The alternative — no durable backing at all — means the ledger's core promise (a shared, real-time view of who's where) can't actually be delivered once more than one device is involved.
- **This is the single most structurally significant finding in this pass** — everything else here is a mapping/sequencing problem; this one is a genuine gap between what the home screen promises and what the schema is designed to represent.

---

### Decision 4 — How does a receptionist find a chain-wide existing patient who hasn't been seen at *this* branch before?
**Priority: BLOCKER before real RLS ships (not before initial wiring under the current `v1_allow_all` policy)**

- **Question:** The drafted "real" staff RLS policy (commented out in `supabase_migration.md`) scopes `patients` visibility to rows where a `patient_clinic_access` entry already exists for the staff member's own `clinic_id`. But `patient_clinic_access` rows are only created by `process_new_complaint_course()` — which fires during Step 2, *after* a patient has already been found and selected in Step 0. For a patient who's an existing chain member (registered at Branch A) walking into Branch B for the first time, the Daily Ledger search at Branch B would find nothing for them under that policy — no access row exists yet, and none can exist until they're already selected.
- **What the frontend currently implies:** The Step 0 search flow only has two outcomes — select an existing (found) patient, or register a brand-new one. There's no third path for "this patient exists somewhere in the chain but not here yet." Under a naive real-RLS rollout, a Branch B receptionist searching for that patient would see nothing and likely register a duplicate — directly undermining the whole point of `owner_id`-based shared identity and `unique(owner_id, mrn)`.
- **What the backend needs to know:** Either the real staff policy needs a different shape than currently drafted (e.g., search itself runs through a narrower, elevated-privilege function rather than a plain RLS-scoped `SELECT`), or Step 0's search needs to explicitly search chain-wide and surface "existing patient, new branch" as its own third outcome, distinct from both existing paths. Not resolvable from the docs alone — this is a genuine gap between the drafted policy and the UI it's meant to serve, not previously flagged anywhere in `GAPS.md`/`LOGS.md`.

---

### Decision 5 — How does MRN actually get generated?
**Priority: LATER**

- **Question:** The registration screen's subtitle reads "Create digital ID & generate MRN," implying automatic generation. No trigger, sequence, or counter for MRN exists anywhere in `supabase_migration.md` — `patients.mrn` is a plain `text not null` column with only `unique(owner_id, mrn)` enforced at insert time.
- **What the frontend currently implies:** Unconfirmed from the docs alone whether MRN is client-generated (e.g. sequential "next available MED-XXX," which would carry the same race-condition risk `invoice_counter` was specifically built to avoid) or left for later.
- **What the backend needs to know:** If sequential, chain-scoped MRNs are wanted, the natural precedent already exists in the schema — an `invoice_counter`-style column on a chain-level entity plus a small trigger, mirroring `process_new_invoice()`. If free text is fine, nothing needs to change, but that should be a stated decision, not a default.

---

### Decision 6 — How should "Consultation" be represented once the shared-per-session exam fee lands?
**Priority: BLOCKER for wiring Procedure Logger — converges with the billing work already in progress**

- **Question:** Today, Procedure Logger treats "Consultation" as an ordinary per-complaint checkbox, priced like any other service. `README.md` §2's confirmed new rule charges the exam fee **once per session** (₹350, shared across however many complaints are treated that day), fully decoupled from the per-complaint therapy rate (₹200/300).
- **What the frontend currently implies:** Two complaints checked in one session currently produces two separate Consultation charges (Screenshot 4: ₹500 + ₹500) — structurally the opposite of "once, shared."
- **What the backend needs to know:** This is the same open problem already named in `README.md` §5 as tangled up with the invoice/session question — something needs to know "is this visit the first billing touchpoint of today's session" to decide whether the shared fee applies, which is exactly the kind of state the "session as frontend-only, never persisted" option (Option C in `DECISIONS.md`) says shouldn't exist anywhere. Given the Q1 decision already reached (audited `BEFORE INSERT` trigger, option B extended) and the pending CHECK-constraint-widening migration for gap-return billing, this decision is best made as part of that same active thread, not independently — Procedure Logger's rebuild should wait on it rather than the reverse.

---

## Section 5: Frontend ↔ Backend Divergences

### A. Already documented in `schema_reconciliation_audit.md` — restated here only where these screenshots add something
1. **Multi-complaint → one invoice.** `vistitworkflow.tsx` L146–153, `ProcedureLogger/index.tsx` L48–76, `InvoiceBuilder/index.tsx`. `invoices.visit_id` is a single nullable FK; Screenshots 3+4 now give this concrete numbers — 3 complaints, ₹1,700, one "Create Invoice" button. Severity as already assigned: Blocks integration.
2. **`invoice_number` expected pre-insert.** `InvoiceBuilder/index.tsx` L15/20/32, `InvoiceHeader.tsx` L14, `vistitworkflow.tsx` L146. Screenshot 5 shows the exact hardcoded value already on file (`INV-2026-0306-088`) — plus the new observation that its format doesn't structurally match `process_new_invoice()`'s actual output (§2, Step 4).
3. **Age captured, not `date_of_birth`.** `DemographicsSection.tsx` L45–55. Screenshot 2 confirms it's still live; the workflow description's own "Upcoming: DOB picker" line confirms this is already a known fix, not a fresh find.
4. **`complaint_catalog_id` dropped on catalog selection.** `ComplaintSelector/index.tsx` L183–193. Not visible in a screenshot by nature (the catalog id is never shown), restated here because Screenshot 3 is exactly where this happens.
5. **`complaint_courses` write timing bypassed in the stepper.** `ComplaintSelector/index.tsx`, `vistitworkflow.tsx`. Directly informs Decision 2 above.
6. **Payment mode not threaded through submission.** `FooterActions.tsx` L11. Screenshot 5 shows the selector *is* visually present and matches `clinics.payment_methods_accepted` — confirms the gap is in data flow, not UI design.

### B. New — surfaced by this pass (screenshot/description evidence only, not yet checked against component code)
1. Daily Ledger's `Waiting`/`In Therapy`/`Paid` status badges vs. the schema's explicit no-status-lifecycle MVP design and `daily_ledger`'s hardcoded `'Complete'`. → Decision 3.
2. "Consultation" modeled as a per-complaint optional `visit_services`-style item vs. `visits.consultation_fee_in_paise` as a distinct snapshot column — now sharpened by the new shared-once-per-session exam fee rule. → Decision 6.
3. Cross-branch patient discoverability gap between the Step 0 search flow and the drafted real staff RLS policy on `patients`. → Decision 4.
4. "Create new MRN" implied auto-generation with no corresponding trigger/counter in the schema, unlike the parallel `invoice_number` pattern. → Decision 5.
5. "Clinical Notes Builder" auto-categorizing "fall risk" is meaningful (not conclusive) evidence toward resolving `LOGS.md`'s open question on whether `patient_alerts` has any live write path at all.
6. `patients.last_visit_at` is chain-wide; "Returning Patient • Last visit: over 2 years ago" doesn't visibly disclose that it isn't branch-specific.

---

## Closing Summary

The workflow is fully specified on the frontend side and fully specified on the schema side — the two were designed somewhat independently, and this pass is mostly about making the seams between them explicit before wiring starts. Six of the fourteen previously-documented mismatches sit directly inside this 5-step flow and now have concrete visual confirmation. Of the six new items, two are low-stakes naming/format questions (Decisions 5, and part of divergence B6); one converges with billing work already underway (Decision 6); and two are genuinely structural, not previously written down anywhere (Decisions 3 and 4) — the ledger's status model has no schema backing at all, and the cross-branch search flow doesn't obviously survive the transition to real RLS. Both are worth resolving deliberately before Steps 0 and 2 get wired, even though neither blocks starting on Steps 1/3/4 first.

---

## Sources
- `supabase_migration.md` (v11) — canonical schema, all table/column/trigger citations
- `README.md` — client business-rules log (fee restructuring, patient flags, package handling)
- `schema_reconciliation_audit.md` — the 14 previously-confirmed frontend↔schema findings, cited by file/line throughout §5-A
- `frontend-schema-audit-handover.md` — trigger-ordering rule, location-specific vs. patient-global table split
- `GAPS.md`, `LOGS.md`, `DECISIONS.md` — session entity absence, invoice/session Option A/B/C, patient_alerts open question
- Screenshots 1–5 (Daily Ledger, New Patient Registration, Complaint Selector, Procedure Logger, Invoice & Payment), Aug 14 2026
