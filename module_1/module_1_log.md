Here's the running log, fully updated with this session's findings:

```
CLINIC SCHEMA — FINDINGS & OPEN ITEMS LOG
(nothing here has been implemented — all still pending your decision)

REAL BUGS / GAPS
- validate_owner_is_admin() confirms owner_id belongs to SOME admin, not the
  CORRECT admin for the submitting staff member's chain. Cross-chain
  patient/clinic misassignment possible at root INSERT. Only matters if
  multiple unrelated chains ever share one database.
- No trigger derives patients.owner_id automatically — CONFIRMED not
  implemented. Frontend is fully responsible for computing and submitting
  the correct value; no database backstop beyond "is this some admin."
- patients.blood_type has no CHECK constraint — any string is accepted.
- No index on clinician_id (patients, complaint_courses, visits) — fine now,
  will matter for per-clinician reporting at real scale.
- owner_id models chain identity as a single admin's user id — no support
  for two co-equal admins of the same chain (documented limitation).
- visits and invoices have NO updated_at column or trigger (unlike every
  other clinical table). By design, matching "row exists = done, no
  lifecycle" MVP philosophy — but means any future retroactive-edit
  feature on these two tables leaves literally zero trace today, not even
  a timestamp.

SCHEMA/FRONTEND MISMATCH (confirmed via screenshots, not hypothetical)
- "Current session bill" UI combines charges from multiple complaint
  courses (= multiple visits rows) into one total and one "Create Invoice"
  action. invoices.visit_id is a single, singular FK — no structural way
  for one invoice to legitimately represent multiple visits today.
- No "session"/"encounter" concept exists anywhere in the schema. Only
  patient_id + date can approximate grouping visits together.
- Three options on the table: (A) one invoice per visit, frontend batches
  the creates; (B) real invoice_visits junction table for true one-to-many
  billing; (C) treat "session" as frontend-only, never persisted — likely
  the pragmatic choice for now.
- Procedure-logger screen has no clinician selector at all, and the
  registration form (per available docs, unconfirmed against live code)
  likely has no clinician field either. Both are reasonable, deliberate
  scope decisions for a single-clinician clinic — closing either gap later
  is frontend-only work, zero backend/migration cost, since the relevant
  columns already exist, nullable and unenforced.

OPEN BUSINESS / DESIGN DECISIONS (not yet resolved)
- Should receptionists/clinicians be allowed to register patients directly,
  or admins only? No real RLS insert policy written yet.
- Should patients.phone / patients.address become NOT NULL?
- Should patients.clinician_id become NOT NULL (real enforcement of
  "must be registered under a clinician") — currently just optional data.
- Deployment model: one Supabase project per chain vs. shared multi-tenant
  DB — determines whether the cross-chain gap and a businesses table ever
  actually matter.
- Multiple-admin support: lightweight sub-admin association on profiles,
  vs. full businesses table? Former is cheaper, still undecided.
- Invoice/session design (see above) — pick A, B, or C.
- Scope of retroactive editing: recommend allowing on attribution fields
  (clinician_id, complaint text) but treating financial fields (fees,
  amounts, payment status) with the same "never silently overwritable"
  philosophy already built into grand_total_in_paise/invoice_number. If
  built, add updated_at to visits/invoices first, at minimum.

DECIDED / RESOLVED
- insurer_name: confirmed unused, nullable, no functional impact — safe to
  drop or leave, no urgency.
- Permission matrix (role_permissions table): confirmed overkill for 3
  fixed roles right now. Flat role checks are correct at this stage.
- Cross-chain gap + businesses table: document only, do not build, pending
  the deployment-model decision above.
- complaint_courses.clinician_id / patients.clinician_id: keep both
  immutable ("who started/registered"), do NOT auto-update via a visits
  trigger. "Currently/last treating" is derived on demand from visits
  (DISTINCT ON pattern) rather than stored.
- Multi-clinician-per-different-complaint-course, same patient, same day:
  confirmed already supported by existing schema shape, no change needed.
```