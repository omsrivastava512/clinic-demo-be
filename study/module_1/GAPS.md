# Module 1 — Gaps, Bugs & Mismatches

Technical debt, missing constraints, schema-frontend mismatches, and deferred features identified in Module 1:

## Real Bugs & Schema Gaps
- **Cross-chain admin validation**: `validate_owner_is_admin()` confirms `owner_id` belongs to *some* admin, not strictly the *correct* admin for the submitting staff member's chain. Cross-chain patient/clinic misassignment is possible at root `INSERT`. Only matters if multiple unrelated chains share one database.
- **Missing `owner_id` derivation trigger**: No trigger derives `patients.owner_id` automatically. Frontend is fully responsible for computing and submitting the correct value; no database backstop beyond "is this some admin."
- **Unconstrained blood type**: `patients.blood_type` has no `CHECK` constraint — any arbitrary string is accepted.
- **Missing clinician indexes**: No index on `clinician_id` across `patients`, `complaint_courses`, and `visits`. Fine for current volume, but will matter for per-clinician reporting at scale.
- **Single-admin chain model**: `owner_id` models chain identity as a single admin's user ID — no native support for two co-equal admins of the same chain (documented limitation).
- **Missing `updated_at` on write-once tables**: `visits` and `invoices` have no `updated_at` column or timestamp trigger (unlike all other clinical tables). By design, matches the "row exists = done, no status lifecycle" MVP philosophy, but means any future retroactive-edit feature on these two tables leaves zero trace today.

## Schema / Frontend Mismatches
- **Multi-visit invoice mismatch**: "Current session bill" UI combines charges from multiple complaint courses (= multiple `visits` rows) into one total and one "Create Invoice" action. However, `invoices.visit_id` is a single singular foreign key — no structural way for one invoice to represent multiple visits today.
- **Absence of "session" entity**: No "session" or "encounter" concept exists in the database schema. Only `(patient_id, date)` can approximate grouping visits together.
- **Missing clinician selector in UI**: Procedure-logger screen has no clinician selector, and intake registration form has no clinician assignment field. Both are acceptable scope decisions for a single-clinician setup — closing either later is frontend-only work with zero backend/migration cost since columns already exist nullable.
