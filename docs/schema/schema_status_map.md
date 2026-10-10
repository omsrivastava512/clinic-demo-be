# Schema Status Map — August 30, 2026
*Built from: module_2/LOGS.md, TODO.md, TODO2.md | business_logic/general/README.md | migration/handover-*.md | fee_calculation_logic/specialty-decision-log.md | packages_logic/README.md*

> This is a one-time read, not a living doc. Every item here either has a status (DECIDED/OPEN/DEFERRED) or a call to action. Nothing is assumed or inferred.

---

## 1. DECIDED — No further discussion needed, these go directly into SQL

### Money & Billing Architecture
- **Four-bucket visits money columns**: split `consultation_fee_in_paise` into:
  - `exam_fee_in_paise` — 350, fires at most once per patient per day
  - `lapse_penalty_in_paise` — 150, fires at most once per patient per day (gap-return only)
  - `therapy_fee_in_paise` — 200/300 per complaint per visit (no day-level cap)
  - `services_total_in_paise` — unchanged, itemized add-ons
  - `grand_total_in_paise` stays as GENERATED sum of all four
- **Three visit states**: `visits.consultation_type` CHECK needs a third value: `'GAP_RETURN'` added alongside `'FIRST'` / `'SUBSEQUENT'`
- **Same-day dedup rule**: BEFORE INSERT trigger on visits checks `patient_id + date` for whether exam_fee or lapse_penalty was already charged by another visit row that day — if yes, this row gets 0 for that column. Scoped by patient+date, NOT by complaint.
- **Rehab tier confirmed**: separate `clinic_tier_rates` table needed for tier-dependent daily visit and package pricing. Shape: `(clinic_id, tier, daily_rate_in_paise, package_rate_in_paise, exam_fee_in_paise)`.
- **Exam fee is tier-independent at 350 for now** — but `clinic_tier_rates` builds the column anyway to avoid a future migration.
- **MACHINE_ONLY billing** already solved: seeds machines as `services` rows, uses `clinic_service_prices` for per-clinic overrides, `visit_services` for what was used. No new structure.
- **`clinics.consultation_fee_first_in_paise` / `consultation_fee_subsequent_in_paise`** are confirmed obsolete under the new model.

### Pricing Integrity & Overrides
- **Override columns on `visits`**: `final_amount_in_paise` (nullable int), `override_reason` (text), `override_by` (uuid → profiles), `override_status` CHECK `('none','pending','approved')` default `'none'`
- **Override columns on `packages`**: `computed_refund_in_paise`, `final_refund_in_paise`, `override_reason`, `override_by`, `override_status` — same shape
- **`packages.status`**: add `'Cancelled'` to the CHECK constraint
- **Three-state CHECK constraint on override columns**: status=none requires all others NULL; status=pending requires reason filled, final_amount still NULL; status=approved requires all filled. Trigger separately validates `auth.uid()` has `role='admin'` before allowing status→approved.
- **`visits_with_effective_charge` view**: exposes `COALESCE(final_amount_in_paise, grand_total_in_paise)` as `effective_charge_in_paise`. Must be created with `security_invoker = true`.
- **All views going forward**: `WITH (security_invoker = true)` is a standing rule, not case-by-case.
- **Request-and-approve workflow** (not TOTP) for all waivers.
- **Refund formula**: `package_total - (days_used × regular_rate)`. NOT the discounted per-day rate.
- **Per-patient standing overrides**: `patient_rate_overrides` table with component model:
  - `(patient_id, override_type CHECK('CONSULTATION_FEE','THERAPY_RATE','FLAT_TOTAL'), override_amount_in_paise, reason, granted_by, granted_at, expires_at, is_active)`
  - Partial unique index: `(patient_id, override_type) WHERE is_active`
  - Om's confirmed case: `CONSULTATION_FEE = 0` + `THERAPY_RATE = 180`, scales per complaint

### Package System
- **`visits.package_id uuid references packages(id)`** — nullable, add it
- **`attended_days`** — computed at read time via `COUNT(visits WHERE package_id = X)`. Not trigger-maintained.
- **Partial unique index**: at most one Active package per complaint — `ON packages(linked_complaint_id) WHERE status = 'Active'`
- **Unique constraint**: one visit per complaint per day — `UNIQUE (complaint_course_id, date)` on visits
- **Package modes**: `CALENDAR_BASED` vs `VISIT_BASED`
  - `clinics.default_package_mode` — pre-fill default
  - `patients.default_package_mode_hint` — advisory only
  - `packages.package_mode NOT NULL` — the binding decision, snapshotted at sale time
  - `packages.visits_promised integer` — for VISIT_BASED packages
  - `packages.duration_days` — made nullable (only required for CALENDAR_BASED)
  - Conditional CHECK: `(package_mode='CALENDAR_BASED' AND duration_days IS NOT NULL) OR (package_mode='VISIT_BASED' AND visits_promised IS NOT NULL)`
  - `expiry_date` stays nullable under both modes (optional ceiling)
- **VISIT_BASED packages** can optionally carry expiry_date (null = indefinite)
- **`clinic_closures` table** (reactive, not maintained calendar):
  - `(id, clinic_id, closure_date, reason, created_by, created_at, closure_type CHECK('FULL','NO_PACKAGE_PATIENTS'))`
  - `UNIQUE (clinic_id, closure_date)`
- **`package_day_waivers` table** (per-patient exception):
  - `(id, package_id, waived_date, reason, granted_by, created_at, UNIQUE(package_id, waived_date))`
  - Admin-gated (granted_by must be role='admin')
- **`exclude_sundays` boolean → `clinics.recurring_closed_weekdays`**: generalize from hardcoded Sunday to a per-clinic weekday set. `clinic_closures` handles one-off dates.
- **Three-question per-day logic** (in priority order): (1) visit logged? → attended. (2) waiver row exists? → waived. (3) otherwise → missed.
- **Package status flip (Active → Expired)**: lazy, checked at two touchpoints only: visit-logging attempt + package detail view open.
- **Package mode override workflow**: package created immediately at clinic default (fully valid transaction). Override request is a separate pending layer on top — same create-at-default pattern as fee waivers.
- **Hold pending / Cancel pending**: while pending, Cancel blocks new visits from drawing against the package. Hold: leaning toward retroactive-waiver-on-approval (option b).

### Patient Flags
- **Dedicated `patient_flags` table** (separate from `patient_alerts`):
  - `(patient_id, owner_id, flagged_by, reason, created_at)`
  - Uses `derive_owner_id_from_patient()` pattern — flag is chain-wide
  - Manual only, never auto-flagged
  - Display: red marker on patient profile wherever it renders; "red zone" list = `SELECT patients WITH ANY flag row`

### Specialty Treatments (partial)
- **Package exclusion is structural** (specialty charges land in `services_total_in_paise`, which the package trigger never touches) — NOT a category filter on `visit_services`
- **Specialty events require at least one linked complaint** — enforced at API layer, not a DB constraint
- **No new frontend screen needed for specialty logging** — existing Procedure Logger gets a complaint multi-select added when a specialty service is selected

### Module 2 Schema Gaps (confirmed bugs, not decisions)
- `patients.clinician_id` — bare FK, no enforcement that profile has `role='clinician'`
- All tables still on `v1_allow_all` RLS — real policies are Module 5/6 work
- Staff-submitted writes' `clinic_id` trusted from client, not derived from `auth.uid()` — Module 5/6 decision
- Denormalized snapshot columns (`complaint_name`, `service_name`, `linked_complaint_name`) are never validated against their paired FKs — entirely frontend's responsibility

---

## 2. OPEN — Needs Om's explicit call before SQL can be written

These are real forks. Building without deciding creates tech debt that costs a migration later.

### The Biggest Fork (affects specialty, invoices, and session design simultaneously)
**1. Session/Encounter Architecture** — what do you actually want?
- The schema has no `session` concept. A visit row is one complaint × one day.
- This causes friction in four places: (a) multi-visit invoice grouping, (b) exam/lapse-penalty firing once per session across multiple visit rows, (c) one-visit-per-complaint-per-day constraint vs. real same-day split encounters, (d) specialty treatments covering multiple complaints needing an anchor.
- The four-bucket trigger and partial-unique-index workarounds are buildable WITHOUT a session table. But they're workarounds. A real `sessions` table would simplify the exam/lapse-penalty dedup from "scan today's visit rows" to "did this session already have an exam fee."
- **Options still open:**
  - Option A: stay sessionless, use the workarounds (four-bucket dedup trigger + visits.package_id + invoice_line_items fleshed out as Option B below)
  - Option B: build a narrow `sessions` table scoped only to specialty anchoring + invoice assembly. Does NOT replace the patient+date exam/lapse dedup logic.
  - Option C: session as frontend-only concept, never persisted — ruled out by specialty, since a specialty event needs a real FK target if it's a standalone table

**2. Specialty Treatment Table Design** — depends on decision 1 above
- **Option A**: embed in `visit_services` + junction table `visit_service_complaint_links(visit_service_id, complaint_course_id)` — no new table, reuses existing structure
- **Option B**: standalone `special_treatment_events` table with its own FK to sessions or to patient+date — clean separation, new table
- Both options require at least one linked complaint per event (already decided)
- Junction table preferred over array column for complaint linking (leaning, not confirmed)

**3. Invoice/Session Model (Option A/B/C)** — depends on decisions 1 & 2
- **Option A**: one invoice per visit row, frontend batches display. Simple. Leaves `invoices.visit_id` as-is.
- **Option B** (`invoice_line_items` table): real child table of `invoices`, one row per visit/specialty event, each carrying `complaint_course_id` and `amount`. This is what the existing frontend `InvoiceBuilder` component already expects (`items` array with `complaintId/complaintName/cost`). Also resolves the "override on a visit doesn't flow to invoice" gap.
- **Option C**: session as frontend-only. Ruled out by specialty needing a real FK.
- The frontend evidence (InvoiceBuilder items array, "Current Session Bill" label) independently points at Option B.

### Smaller Calls (don't block the big schema work, just need answers)
**4. Joining fee question**: a returning patient opens a brand-new complaint. Do they pay:
  - The full ₹350 exam fee again (per-complaint, re-triggered by new complaint)?
  - Or a true once-ever joining fee that never re-triggers?
  - *Resolved in FEECOMP-01 / mvp-index.md:* **PATIENT_LIFETIME** (first-ever exam fee paid once, subsequent complaints pay ₹0 exam fee).

**5. `patient_flags` — who can flag?**
  - Admin-only (same as Hold/Cancel)
  - Any staff member can flag (front-desk staff are most likely to encounter the friction)
  - Simple call. No schema difference — just an RLS policy.

**6. Hold pending — blocking behaviour**
  - Option (a): hold takes provisional effect immediately on request (receptionist can unilaterally pause a package clock pending review)
  - Option (b): nothing changes until approved; approval retroactively backdates waivers for the pending window (keeps strict "receptionist can't change anything alone" rule, leaning direction)

**7. `patients.default_package_mode_hint` — advisory or ever binding?**
  - Advisory only (no enforcement, just a pre-fill suggestion)
  - Binding for certain complaint types
  - Currently designed as advisory. Confirm that's what you want.

**8. `clinic_tier_rates` naming**: `CALENDAR_BASED` / `VISIT_BASED` for package modes and `duration_days` / `visits_promised` for the two mode fields.

---

## 3. SCHEMA IMPLICATIONS — What SQL is implied by the DECIDED items

These can be written now, without waiting for the OPEN decisions above (except where noted).

### New Tables (6)
```sql
-- 1. Patient flags
CREATE TABLE patient_flags (
  id          uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id  uuid NOT NULL REFERENCES patients(id),
  owner_id    uuid NOT NULL REFERENCES profiles(id),
  flagged_by  uuid NOT NULL REFERENCES profiles(id),
  reason      text NOT NULL,
  created_at  timestamptz NOT NULL DEFAULT now()
);

-- 2. Clinic tier rates
CREATE TABLE clinic_tier_rates (
  clinic_id          uuid NOT NULL REFERENCES clinics(id),
  tier               text NOT NULL CHECK (tier IN ('REGULAR','REHAB')),
  exam_fee_in_paise  integer NOT NULL DEFAULT 35000,
  daily_rate_in_paise integer NOT NULL,
  package_rate_in_paise integer NOT NULL,
  PRIMARY KEY (clinic_id, tier)
);

-- 3. Patient rate overrides (component model)
CREATE TABLE patient_rate_overrides (
  id                    uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  patient_id            uuid NOT NULL REFERENCES patients(id),
  override_type         text NOT NULL CHECK (override_type IN ('CONSULTATION_FEE','THERAPY_RATE','FLAT_TOTAL')),
  override_amount_in_paise integer NOT NULL,
  reason                text NOT NULL,
  granted_by            uuid NOT NULL REFERENCES profiles(id),
  granted_at            timestamptz NOT NULL DEFAULT now(),
  expires_at            timestamptz,
  is_active             boolean NOT NULL DEFAULT true
);
CREATE UNIQUE INDEX uq_one_active_override_per_type
  ON patient_rate_overrides(patient_id, override_type) WHERE is_active;

-- 4. Clinic closures (reactive, not maintained calendar)
CREATE TABLE clinic_closures (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  clinic_id    uuid NOT NULL REFERENCES clinics(id),
  closure_date date NOT NULL,
  closure_type text NOT NULL DEFAULT 'FULL' CHECK (closure_type IN ('FULL','NO_PACKAGE_PATIENTS')),
  reason       text,
  created_by   uuid REFERENCES profiles(id),
  created_at   timestamptz NOT NULL DEFAULT now(),
  UNIQUE (clinic_id, closure_date)
);

-- 5. Package day waivers (per-patient exceptions)
CREATE TABLE package_day_waivers (
  id           uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  package_id   uuid NOT NULL REFERENCES packages(id),
  waived_date  date NOT NULL,
  reason       text,
  granted_by   uuid REFERENCES profiles(id),
  created_at   timestamptz NOT NULL DEFAULT now(),
  UNIQUE (package_id, waived_date)
);

-- 6. invoice_line_items — BLOCKED on Open Decision #3 (invoice/session model)
-- Do not write until Option A/B/C is decided.
```

### Columns to Add / Modify (5 tables)

```sql
-- visits: four-bucket money split + override columns + package link + GAP_RETURN state
ALTER TABLE visits
  ADD COLUMN exam_fee_in_paise       integer NOT NULL DEFAULT 0,
  ADD COLUMN lapse_penalty_in_paise  integer NOT NULL DEFAULT 0,
  ADD COLUMN therapy_fee_in_paise    integer NOT NULL DEFAULT 0,
  ADD COLUMN package_id              uuid REFERENCES packages(id),
  ADD COLUMN final_amount_in_paise   integer,
  ADD COLUMN override_reason         text,
  ADD COLUMN override_by             uuid REFERENCES profiles(id),
  ADD COLUMN override_status         text NOT NULL DEFAULT 'none'
    CHECK (override_status IN ('none','pending','approved'));
-- Drop consultation_fee_in_paise after migrating existing data
-- Widen consultation_type CHECK to include 'GAP_RETURN'
-- grand_total_in_paise formula: update to sum all four fee columns

-- packages: mode, override columns, Cancelled status
ALTER TABLE packages
  ADD COLUMN package_mode        text NOT NULL DEFAULT 'CALENDAR_BASED'
    CHECK (package_mode IN ('CALENDAR_BASED','VISIT_BASED')),
  ADD COLUMN visits_promised     integer,
  ALTER COLUMN duration_days     DROP NOT NULL,
  ADD COLUMN computed_refund_in_paise integer,
  ADD COLUMN final_refund_in_paise    integer,
  ADD COLUMN override_reason     text,
  ADD COLUMN override_by         uuid REFERENCES profiles(id),
  ADD COLUMN override_status     text NOT NULL DEFAULT 'none'
    CHECK (override_status IN ('none','pending','approved'));
-- Add 'Cancelled' to packages.status CHECK
-- Add conditional CHECK on package_mode fields

-- clinics: tier defaults, package defaults, recurring closures
ALTER TABLE clinics
  ADD COLUMN default_visit_type          text,
  ADD COLUMN default_package_mode        text DEFAULT 'CALENDAR_BASED'
    CHECK (default_package_mode IN ('CALENDAR_BASED','VISIT_BASED')),
  ADD COLUMN recurring_closed_weekdays   integer[] DEFAULT '{}';
-- Drop or deprecate consultation_fee_first_in_paise / consultation_fee_subsequent_in_paise

-- patients: package mode hint
ALTER TABLE patients
  ADD COLUMN default_package_mode_hint text
    CHECK (default_package_mode_hint IN ('CALENDAR_BASED','VISIT_BASED'));
```

### New Indexes
```sql
-- One active package per complaint at a time
CREATE UNIQUE INDEX uq_one_active_package_per_complaint
  ON packages(linked_complaint_id) WHERE status = 'Active';

-- One visit per complaint per day
ALTER TABLE visits
  ADD CONSTRAINT uq_one_visit_per_complaint_per_day
  UNIQUE (complaint_course_id, date);
```

### New View
```sql
CREATE VIEW visits_with_effective_charge
  WITH (security_invoker = true) AS
SELECT
  *,
  COALESCE(final_amount_in_paise, grand_total_in_paise) AS effective_charge_in_paise
FROM visits;
```

### New Triggers
1. **Same-day dedup** — BEFORE INSERT on visits: check `patient_id + date` for existing exam_fee or lapse_penalty; zero those columns on new row if already charged
2. **Override authorization** — BEFORE UPDATE on visits/packages: when `override_status` moves to `'approved'`, verify `auth.uid()` has `role='admin'`, derive `override_by` from `auth.uid()`

---

## 4. WHAT IS BLOCKING NEXT STEPS

### Hard Blockers (nothing downstream can be written without these)

| Open Decision | What it blocks |
|---|---|
| **#1 Session/encounter** | Specialty table design (Open #2), invoice model (Open #3), `invoice_line_items` table |
| **#3 Invoice/session model (A/B/C)** | The shape of invoices, how the billing UI submits, whether `invoice_line_items` gets built |
| **#2 Specialty treatment table** | Any schema for specialty treatments (`special_treatment_events` or junction table on `visit_services`) |

### Non-Blockers (can be written now, independent of the above)
All items in Section 3 above except `invoice_line_items`.

Specifically, these are buildable immediately:
- `clinic_tier_rates`, `patient_flags`, `patient_rate_overrides`, `clinic_closures`, `package_day_waivers`
- All `visits` column additions (four-bucket split, package_id, override columns, GAP_RETURN state)
- All `packages` column additions (package_mode, visits_promised, override columns, Cancelled status)
- All `clinics` and `patients` column additions
- Both new indexes
- `visits_with_effective_charge` view
- Both new triggers
