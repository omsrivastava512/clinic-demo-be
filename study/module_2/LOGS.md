# Running Gaps & Findings Log — Module 2 onward

Continues the "Running findings / open items log" from module-1-handover.md.
Append-only as we go. Something appearing here means "flagged," not "resolved" —
resolution happens later, deliberately, not by writing it down.

---

## REAL BUGS / GAPS (confirmed against the schema as written)

- `patients.clinician_id` — bare FK to `profiles(id)`, checks existence only.
  Nothing enforces that the referenced profile actually has `role = 'clinician'`.
  Same category of issue as the `owner_id`/admin gap already logged in Module 1,
  just on a different column, with no equivalent guard trigger anywhere.

- No enforcement exists yet — at any layer — that a receptionist/clinician can
  only write records at their own assigned clinic. Confirmed by reading the
  live RLS section directly: every table currently runs on `v1_allow_all`
  (any authenticated user, any row). The one "real" staff-scoped policy shown
  in the SQL is commented out, covers `patients` only (not visits/complaint_
  courses/invoices), and even that draft only scopes reads via `using` — no
  insert-time (`with check`) restriction on submitted `clinic_id` is drafted
  anywhere. Today, a Clinic 1 receptionist can submit `clinic_id = Clinic 2`
  on a write and the trigger layer will validate it against the *patient's*
  access, never against the *submitting staff member's* own clinic.

- None of the denormalized display-name snapshot columns (`complaint_courses.
  complaint_name`, `visit_services.service_name`, `packages.linked_complaint_
  name`) are derived or validated against their paired FK by any trigger.
  Each is trusted verbatim from client input, same as any other plain text
  column — nothing checks that `complaint_name` actually matches what
  `complaint_catalog_id` points to, etc. A row can have a structurally valid
  FK and a completely wrong display name at the same time, with zero error.
  Frontend is fully responsible for keeping these in sync at write time;
  nothing on the backend will ever catch drift.

---

## OPEN QUESTIONS (unconfirmed — need real frontend code, not just schema docs)

- `patient_alerts` — Om doesn't remember a real frontend form for this and
  suspects it leaked in from an old mockup type (`types/index.tsx L71-74`,
  per the migration's own source comment) without becoming a real feature.
  Possibly fully redundant with `clinical_notes` + `is_critical`. Not
  resolvable from the docs alone — needs a direct check against the actual
  frontend code. Same category of question as Module 8b's "is the catalog
  wired the way you actually want it used" — revisit there.

- General version of the above, as a standing principle: a table/column with
  no corresponding frontend form is not automatically a gap — could be
  deliberate future scope (much of the frontend is still mockup). Needs an
  eventual table-by-table audit against real usage, not an assumption either
  way in the meantime.

- Whether the visit→invoice write happens as one coordinated backend action
  (complaint_course if new → visit → visit_services → invoice, all fired
  together at a single "submit" point) or as several progressive writes
  across the UI flow is not confirmed against real frontend/API code —
  codebase-context.md describes the app as still running on local mock data,
  with real Supabase integration described as "upcoming." Strongest available
  signal (the "final invoice payload" assembled before one submit; the MVP
  "row existing = complete + paid" philosophy on visits/invoices) points
  toward one coordinated write at the end, not progressive writes. Confirm
  directly once real integration code exists — check `VisitWorkflow`,
  `ProcedureLogger/index.tsx`, and whatever submit handler wires them up.

---

## OPEN BUSINESS / DESIGN DECISIONS

- Should `clinic_id` on staff-submitted writes (visits, complaint_courses,
  packages) be **derived** server-side from the submitting user's own
  `profiles.clinic_id` (via `auth.uid()`), instead of trusted-then-validated
  from client input — same pattern as `derive_owner_id_from_patient()`, just
  pointed at the staff member instead of the patient. Would structurally
  close the receptionist-crosses-clinics gap above (can't submit a false
  value at all, vs. today: submit one and have it checked against the
  patient, not the submitter).
  - Complication: `profiles.clinic_id` is `NULL` for admins by design — a
    pure derive rule has nothing to fall back to for admin-submitted writes.
    Would need a role branch (derive for staff, validate-against-chain for
    admins).
  - Complication: `auth.uid()` is `NULL` when running raw SQL directly in the
    Supabase SQL Editor (no request JWT outside a real app request — this is
    exactly what Module 5 explains). A derive-from-`auth.uid()` trigger would
    behave correctly through the real app but differently under manual SQL
    Editor testing — relevant since that's exactly how Module 1/2 exercises
    are being run.
  - Not decided. Worth resolving deliberately, likely around Module 5/6.