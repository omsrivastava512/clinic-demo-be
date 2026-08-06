# Module 1 — Architecture Decisions & Design Resolves

Key architectural decisions, trade-offs evaluated, and design patterns resolved during Module 1:

## Decided / Resolved
- **`insurer_name`**: Confirmed unused and nullable with no functional impact — safe to drop or leave, no urgency.
- **Permission matrix (`role_permissions` table)**: Confirmed overkill for 3 fixed roles right now. Hardcoded flat role checks in RLS policies are appropriate for this stage.
- **Cross-chain security gap & `businesses` table**: Documented only; do not build until multi-tenant deployment model is explicitly chosen.
- **`complaint_courses.clinician_id` / `patients.clinician_id`**: Keep both immutable ("who started/registered the patient/course"). Do **NOT** auto-update via a `visits` trigger. "Currently/last treating clinician" is derived on demand from `visits` (using Postgres `DISTINCT ON` pattern) rather than stored redundantly.
- **Multi-clinician support (different complaint courses, same patient, same day)**: Confirmed already natively supported by existing schema shape (`visits.complaint_course_id` + `visits.clinician_id`), no schema change needed.

## Open Design & Business Decisions (Pending)
- **Registration permissions**: Should receptionists/clinicians be allowed to register patients directly via RLS insert policy, or admins only?
- **Column constraints**: Should `patients.phone` / `patients.address` / `patients.clinician_id` become `NOT NULL` to enforce registration completeness?
- **Deployment model**: One Supabase project per clinic chain vs. shared multi-tenant database (determines whether the cross-chain gap and a `businesses` table ever matter).
- **Multiple-admin support**: Lightweight sub-admin association on `profiles` vs. full `businesses` table (former is cheaper).
- **Invoice/session design**: Choosing between:
  - *(Option A)*: One invoice per visit, frontend batches the creations.
  - *(Option B)*: Real `invoice_visits` junction table for true one-to-many billing.
  - *(Option C)*: Treat "session" as frontend-only, never persisted (pragmatic choice for now).
- **Scope of retroactive editing**: Recommend allowing edits on attribution fields (`clinician_id`, `complaint` text) while keeping financial fields (`fees`, `amounts`, `payment_status`) immutable (matching `grand_total_in_paise` / `invoice_number` philosophy). If built, add `updated_at` to `visits` / `invoices` first at minimum.
