# Comprehensive Backend Schema & Business Logic Discussion Audit

```text
DOCUMENT PURPOSE & SCOPE:
This file is an authoritative, comprehensive audit of all historical schema iterations, study module logs,
and business logic discussions across the clinic management system backend. It aggregates discussion timelines,
exhaustively catalogs resolved architectural decisions, tracks all partially discussed/open threads with exact
quotes, inventories every undiscussed element from the v11 Supabase migration and study plan (Modules 2–10),
and computes velocity metrics across all dated sessions.
```

---

## Section 1 — Discussion Timeline (chronological)

* **Pre-Migration / Iteration 1: Multi-Location Multi-Tenancy & Role-Based Access Control**
  * **Date:** Pre-migration baseline (Iteration 1)
  * **Topic Covered:** Onboarding scoping requirements (single-clinic vs. multi-tenant SaaS; 3-clinic chain with one admin; front desk vs. clinician vs. admin role access levels; scoping hierarchy `businesses` $\rightarrow$ `clinics` $\rightarrow$ `staff`).
  * **Back-and-Forths (Q&A Cycles):** 2 Q&A cycles.
  * **Resolved:** Flat text check constraints for roles (`'front_desk'`, `'clinician'`, `'admin'`); linking `staff.user_id` to Supabase `auth.users(id)`; introducing a `businesses` table for multi-tenancy.
  * **Flagged as Open:** Whether the product will be sold as isolated standalone copies or multi-tenant SaaS; whether a full granular permission matrix (`role_permissions`) is needed (deferred as premature).

* **v1 $\rightarrow$ v2: Initial Schema Critique & Refinement**
  * **Date:** Pre-migration iteration (Iteration 2)
  * **Topic Covered:** Architectural evaluation of initial draft schema: strengths (`_in_paise` integers, audit columns, `daily_ledger` view, placeholder RLS) vs. critical gaps (`clinic_id` scoping missing across tables/profiles; single-row `clinic_settings`; unreferenced `complaint_catalog`; client-side conditional fields; text primary keys and invoice sequence collisions; un-normalized `timeline_events`).
  * **Back-and-Forths (Q&A Cycles):** 6 critique/review points.
  * **Resolved:** Rejected `businesses`/`staff` separation in favor of a single `profiles` table + `clinics.owner_id`; replaced single-row settings with multi-row `clinics`; converted transactional PKs to UUIDs (`gen_random_uuid()`); moved conditional dependencies into database `CHECK` constraints (`chk_referral_doctor_info`, `chk_consultation_type`); introduced atomic sequential invoice counter trigger.
  * **Flagged as Open:** Normalized table vs. dynamic database view for `timeline_events`.

* **v2 $\rightarrow$ v11: Progressive Schema Iterations & Hardening**
  * **Date:** Pre-migration iterations (Iterations 3 through 12)
  * **Topic Covered:** Migration schema progression: per-clinic pricing overrides (`clinic_service_prices`); unified chain patient identity and branch attendance junction (`patient_clinic_access`); 3-tier record isolation model; intake registration deadlock; `SECURITY DEFINER` cross-chain tenancy enforcement; splitting invoice triggers between INSERT and UPDATE; admin role validation on `owner_id`; migration DDL script execution order.
  * **Back-and-Forths (Q&A Cycles):** 18 review cycles.
  * **Resolved:** Baseline table architecture (v2); pricing override schema and clinic guard triggers (v3); unified patient identity with chain-scoped MRN uniqueness and consolidated invoice triggers (v4); index fixes and admin RLS policy corrections (v5); intake deadlock resolved by shifting auto-provisioning to `complaint_courses` (v6); cross-chain tenancy validation in `SECURITY DEFINER` functions (v7); trigger coverage on `complaint_courses` updates (v8); split invoice triggers into INSERT generation and UPDATE validation with early-exit guards (v9); root `owner_id` admin role validation trigger (v10); DDL script execution order fix (v11).
  * **Flagged as Open:** Verification checklist execution against a real live database instance.

* **[2 August 2026] — Core Patient Table Breakdown & Constraint Basics**
  * **Date:** 2 August 2026 (Module 1, Part 1)
  * **Topic Covered:** Top-to-bottom breakdown of `patients` table: `NOT NULL` rules, default values, inline enum check constraints (`gender`, `referral_mode`), compound constraint `chk_referral_doctor_info`, compound unique key `unique(owner_id, mrn)`, integer paise pattern, foreign key references to `profiles(id)`, hands-on SQL constraint error testing.
  * **Back-and-Forths (Q&A Cycles):** 2 Q&A / instructional cycles.
  * **Resolved:** Mechanics of database-level `chk_referral_doctor_info` enforcement; duplicate rejection under `unique(owner_id, mrn)`; integer paise standard for financial values.
  * **Flagged as Open:** None.

* **[2 August 2026] — Constraints, Enums, Roles & UUID Collision Safety**
  * **Date:** 2 August 2026 (Module 1, Part 2)
  * **Topic Covered:** Chain identity via `owner_id`; unconstrained `patients.blood_type`; status of `insurer_name`; separation of `clinician_id` vs. `created_by`; mathematical safety of `gen_random_uuid()` ($2^{122}$ collision space); permission matrix vs. flat role checks.
  * **Back-and-Forths (Q&A Cycles):** 5 Q&A cycles.
  * **Resolved:** `insurer_name` confirmed unused and harmless; permission matrix rejected as premature for 3 fixed roles; single-admin chain ownership model documented; UUID collision risk confirmed mathematically negligible.
  * **Flagged as Open:** Unconstrained `patients.blood_type`; missing clinician role validation on `patients.clinician_id`.

* **[3 August 2026] — Failure Paths, Retry Strategies & Error Handling**
  * **Date:** 3 August 2026 (Module 1, Part 3)
  * **Topic Covered:** Classification of API write failures: transient network glitches vs. deterministic PostgreSQL constraint rejections; retry policies; React Query/client error boundaries.
  * **Back-and-Forths (Q&A Cycles):** 3 Q&A cycles.
  * **Resolved:** Deterministic constraint rejections must fail fast to user UI; transient network timeouts can be retried.
  * **Flagged as Open:** Frontend error handling implementation (deferred to React client wiring phase).

* **[3 August 2026] — Invoice Batching Options & Multi-Complaint Session Design**
  * **Date:** 3 August 2026 (Module 1, Part 4)
  * **Topic Covered:** Schema-frontend mismatch where UI bundles multiple complaint visits into one session bill, but `invoices.visit_id` is a 1:1 foreign key; evaluation of Options A, B, and C.
  * **Back-and-Forths (Q&A Cycles):** 4 Q&A cycles.
  * **Resolved:** Trade-offs evaluated across Option A (1 invoice per visit, client batches), Option B (`invoice_visits` junction table), and Option C (session is frontend-only, never persisted).
  * **Flagged as Open:** Final architectural selection among Options A, B, and C.

* **[4 August 2026] — Single-Clinician Scope & Attendance-Derived Access**
  * **Date:** 4 August 2026 (Module 1, Part 5)
  * **Topic Covered:** Missing clinician selector in Procedure Logger and Intake Form UI; immutable registering clinician vs. dynamic treating clinician; deriving last treating clinician via `DISTINCT ON (complaint_course_id)` from `visits`.
  * **Back-and-Forths (Q&A Cycles):** 4 Q&A cycles.
  * **Resolved:** `patients.clinician_id` and `complaint_courses.clinician_id` remain immutable registration attribution; "currently treating clinician" is derived on demand from `visits` via `DISTINCT ON`; multi-clinician same-day visits confirmed natively supported.
  * **Flagged as Open:** Adding clinician assignment controls to frontend UI.

* **[4 August 2026] — SMTP Auth Limits & Triggers on `auth.users`**
  * **Date:** 4 August 2026 (Module 1, Part 6)
  * **Topic Covered:** Supabase built-in SMTP rate limits (30 emails/hour); automatic creation of `profiles` rows via `handle_new_user()` trigger on `auth.users`.
  * **Back-and-Forths (Q&A Cycles):** 3 Q&A cycles.
  * **Resolved:** Pattern of `handle_new_user()` trigger for auth profile synchronization.
  * **Flagged as Open:** Custom SMTP provider setup in production.

* **[5 August 2026] — Supabase Org Permissions, `profiles` Triggers & Multi-Chain MRN**
  * **Date:** 5 August 2026 (Module 1, Part 7)
  * **Topic Covered:** Supabase dashboard organization permissions vs. Postgres database roles; manual `profiles` insertion vs. trigger provisioning; cross-chain MRN coexistence.
  * **Back-and-Forths (Q&A Cycles):** 4 Q&A cycles.
  * **Resolved:** Separation of dashboard privileges from database RLS; confirmed `unique(owner_id, mrn)` allows identical MRNs across separate chains.
  * **Flagged as Open:** None.

* **[6 August 2026] — Module 2 Foreign Keys, Junction Tables, Concurrency & Topological Sequence**
  * **Date:** 6 August 2026 (Module 2: M2-01 through M2-04)
  * **Topic Covered:** Foreign key cascades vs. `RESTRICT`; seed tables (`services`, `complaint_catalog`) with text PKs vs. transactional UUID tables; junction table `patient_clinic_access`; nullable FK semantics; `patients.owner_id` derivation gap; `patients.clinician_id` role validation gap; soft deletes vs. cascades; invoice concurrency and `clinics.invoice_counter` row-locking; `SECURITY DEFINER` on invoice generation; circular FK dependency resolution (`clinics` $\leftrightarrow$ `profiles`); `IS DISTINCT FROM` three-valued logic; denormalized display name snapshot drift; 15-step topological table creation sequence.
  * **Back-and-Forths (Q&A Cycles):** 14 Q&A / depth sections.
  * **Resolved:** Scoped cascade deletion strictly to patient-owned child data; resolved circular dependencies via deferred `ALTER TABLE`; verified atomic invoice row-locking; formalized 15-step dependency creation sequence.
  * **Flagged as Open:** `patients.owner_id` missing derivation trigger; `patients.clinician_id` missing clinician role check; `patient_alerts` table redundancy audit; server-side `clinic_id` derivation from `auth.uid()`; denormalized display name snapshot synchronization.

* **[8 August 2026] — Visit-to-Invoice Workflow & UI Filter Boundaries**
  * **Date:** 8 August 2026 (Module 2: M2-05)
  * **Topic Covered:** Standardized module naming (`M{module}-R{number}`); invoice counter increment timing; full visit $\rightarrow$ invoice submission flow; admin clinic-switcher as frontend display filter vs. backend security boundary; repeat-visit procedure pre-selection.
  * **Back-and-Forths (Q&A Cycles):** 5 discussion sections.
  * **Resolved:** `clinics.invoice_counter` increments strictly at final invoice insertion; admin switcher established as UI display filter over full chain access; procedure pre-selection supported via existing queryable relations.
  * **Flagged as Open:** Staff clinic write validation at RLS layer (Module 5); `SECURITY DEFINER` deep dive (Module 6); admin RLS policies for `visits` and `invoices`; frontend verification of single-submit vs. progressive writes.

* **[10 August 2026] — Fee Structure, Visit Types & JSONB Configuration**
  * **Date:** 10 August 2026 (Business Logic, Part 1)
  * **Topic Covered:** Purpose of `clinics.config` JSONB; `visits.visit_type` (`CONSULTATION` vs. `MACHINE_ONLY`); machine-only itemized cart pricing; first-time exam fee (₹350 exam + ₹200/complaint therapy); 10-day gap-return concept; rehab pricing tier; package refund deduction policy; 2D rate table (`clinic_tier_rates`) vs. columns vs. `.env`.
  * **Back-and-Forths (Q&A Cycles):** 6 Q&A cycles.
  * **Resolved:** `clinics.config` is strictly for high-level feature toggles (`inventoryEnabled`, `appointmentsEnabled`), not pricing; `visit_type` is selected per visit row; machine-only visits are itemized via `visit_services` + `clinic_service_prices`; first-time exam is ₹350 exam-only + ₹200 per complaint for therapy; package refunds deduct consumed days at regular daily rates; `.env` and monolithic settings tables rejected in favor of dedicated `clinic_tier_rates`.
  * **Flagged as Open:** Final naming for non-itemized doctor-directed visit type; whether 10-day gap penalty applies once or per complaint; whether ₹350 exam fee varies by Rehab tier.

* **[10 August 2026] — Additive Pricing Engine & Waiver Approval Flow**
  * **Date:** 10 August 2026 (Business Logic, Part 2)
  * **Topic Covered:** Blast radius of mixed visit types (confirmed zero); additive penalty model vs. combinatorial branching formulas; fee waiver approval workflow UX (scrap-and-redo vs. password prompt vs. request-and-approve); tier-aware exam fee column.
  * **Back-and-Forths (Q&A Cycles):** 5 Q&A cycles.
  * **Resolved:** Additive multi-rule calculation model selected; request-and-approve workflow chosen for fee waivers (asynchronous admin approval via `override_by`); exam fee column added to tier rate table (defaulting ₹350 across all tiers).
  * **Flagged as Open:** None.

* **[13 August 2026] — Server-Side Fee Computation & Price Override Auditing**
  * **Date:** 13 August 2026 (Business Logic, Part 3 & 4)
  * **Topic Covered:** Origin of fee numbers (client vs. server calculation); gap-return fee formula correction (₹150 flat penalty); manual price override mechanism (`computed_amount_in_paise`, `actual_amount_in_paise`, `override_reason`, `override_by`); admin authorization mechanics; standing discounts (₹180-forever); promotional discounts; precedent correction (`derive_owner_id_from_patient()`); `patient_flags` table; unapproved pending overrides not blocking visits; `patient_rate_overrides` design.
  * **Back-and-Forths (Q&A Cycles):** 11 Q&A / topic turns.
  * **Resolved:** Gap penalty formula corrected to normal complaint therapy fees + flat ₹150 penalty once per visit; backend must strictly calculate totals from rate tables; override audit columns established on `visits` and `packages`; shared secret codes rejected; standing and promo discounts routed through standard admin override mechanism; `patient_flags` separated from `patient_alerts`.
  * **Flagged as Open:** Authentication transport for in-person admin approval; whether `patient_flags` creation is admin-only or open to staff; whether standing ₹180 rate is flat total or component-based.

* **[14 August 2026] — PL/pgSQL Triggers, Pre-fill, Specialty Services & Component Rates**
  * **Date:** 14 August 2026 (Module 2: M2-06 & Business Logic)
  * **Topic Covered:** PL/pgSQL BEFORE INSERT trigger `NEW` candidate row lifecycle; multi-complaint procedure pre-fill using `DISTINCT ON (complaint_course_id)` in SQL (2 queries); rejection of frontend history filtering; `clinic_default_services` table; specialty services (Dry Needling, Cupping, Laser, Taping) pricing and carve-outs; nullable `final_amount_in_paise`; 3-way override CHECK constraints; adding `visits.package_id`; component-based `patient_rate_overrides` (`CONSULTATION_FEE`, `THERAPY_RATE`, `FLAT_TOTAL`).
  * **Back-and-Forths (Q&A Cycles):** 14 discussion and Q&A sections.
  * **Resolved:** Pre-fill resolved via SQL `DISTINCT ON`; prescription table rejected for `clinic_default_services`; specialty services categorized as `PREMIUM` in `services` + `clinic_service_prices` and billed as line items in `visit_services`; `final_amount_in_paise` made nullable; `visits.package_id` foreign key adopted.
  * **Flagged as Open:** `clinic_default_services` clinic-wide vs. clinic+clinician scope; specialty service duration/machine limits per visit; `missed_days` calculation mechanics; `day_log` single source of truth consolidation.

* **[16 August 2026] — Package Management, Day Logs & Attendance Tracking**
  * **Date:** 16 August 2026 (Business Logic)
  * **Topic Covered:** Package lifecycle (`Active`, `Completed`, `Expired`); `day_log` JSONB array structure; attendance counters (`attended_days`, `missed_days`); Sunday exclusions (`exclude_sundays`); mid-package complaint addition.
  * **Back-and-Forths (Q&A Cycles):** 5 Q&A cycles.
  * **Resolved:** Package tracking schema using `attended_days`, `missed_days`, and `packages.day_log` JSONB array; Sunday exclusion inherited from clinic defaults; auto-completion at duration limit; mid-package complaint addition creates separate complaint course.
  * **Flagged as Open:** Cross-branch package utilization within chain; automatic expiry background cron vs. lazy on-read evaluation.

* **[17 August 2026] — Views, RLS Bypass, and Session Risk Analysis**
  * **Date:** 17 August 2026 (Business Logic & Package Logic)
  * **Topic Covered:** Generated column limitations in Postgres (cannot reference generated columns); view `visits_with_effective_charge`; JWT `auth.uid()` admin role derivation for overrides; `day_log` JSONB audit; transitive join vs. direct `visits.package_id`; read-time `attended_days` computation; subscription vs. attendance package models; reactive `clinic_closures` table; `package_day_waivers` table; lazy status expiration; 3-layer package configurability.
  * **Back-and-Forths (Q&A Cycles):** 9 architectural / Q&A sections.
  * **Resolved:** Generated columns cannot reference generated columns; view `visits_with_effective_charge` adopted; `visits.package_id` confirmed mandatory; `attended_days` computed at read-time; package days computed from calendar minus closures and waivers; `clinic_closures` and `package_day_waivers` tables designed; lazy status expiration adopted.
  * **Flagged as Open:** Invoicing Option A/B/C billing correctness fork; outer time boundary on attendance-driven packages; whether patient package mode hint is binding or advisory; final naming of package modes.

* **[18 August 2026] — Security Invoker, Constraints, Four-Bucket Split, Package Expiry & Closures**
  * **Date:** 18 August 2026 (Business Logic, Package Logic & Handover)
  * **Topic Covered:** Mandatory `WITH (security_invoker = true)` on Postgres 15+ views to prevent RLS bypass; downstream view blast radius; partial unique index `idx_one_active_package_per_complaint`; partial unique index on `patient_rate_overrides`; four-bucket money column redesign (`exam_fee_in_paise`, `lapse_penalty_in_paise`, `therapy_fee_in_paise`, `services_total_in_paise`); same-day flat fee dedup trigger (`patient_id + date`); Om's ₹180 case resolved; package mode renamed `VISIT_BASED`; unique constraint `uq_one_visit_per_complaint_per_day`; `clinics.recurring_closed_weekdays integer[]` generalizing Sunday exclusion; synthesis into handover documents (`handover-fee-computation.md`, `handover-packages.md`, `handover-pricing-integrity-overrides.md`, `handover-index.md`).
  * **Back-and-Forths (Q&A Cycles):** 17 topic and synthesis sections.
  * **Resolved:** `WITH (security_invoker = true)` made mandatory on all views; four-bucket money redesign adopted; same-day flat fee dedup trigger adopted; Om's ₹180 case structured as component override; package mode renamed `VISIT_BASED`; `recurring_closed_weekdays` adopted; migration roadmap synthesized.
  * **Flagged as Open:** Invoices amount disconnection from visit overrides; enforcement of active status on complaint course before visit insert; Hold/Cancel receptionist approval workflows; converting logged visit to day 1 of new package; formal migration script generation.

* **[19 August 2026] — Scope Configurability, Joining Fees & Package Waivers Refinement**
  * **Date:** 19 August 2026 (Business Logic & Package Logic)
  * **Topic Covered:** `clinics.consultation_fee_scope` (`PER_VISIT_DAY` vs. `PER_COMPLAINT`); partial unique index for therapy-charging visits; same-day 2-complaint race condition; "joining fee" vs. "per-complaint exam fee" tension; `clinic_fee_scopes` table; multi-complaint specialty representation (`visit_service_complaint_links`); package exclusion of specialty services; server-side `package_id` derivation trigger; `package_day_waivers` 3-question model; stress-testing 4 closure approaches (auto-detect login absence REJECTED, daily onboarding prompt ADOPTED); mode switch non-retroactivity; blocking actions during pending requests.
  * **Back-and-Forths (Q&A Cycles):** 9 technical and architectural sections.
  * **Resolved:** `clinics.consultation_fee_scope` column added; examination fee modeled as catalog row with scope table; specialty multi-complaint linking recommended via `visit_service_complaint_links`; auto-detect login absence rejected in favor of daily login prompt; create-at-default pattern adopted; mode switch made strictly forward-looking; Cancel/Refund pending blocks visit draws.
  * **Flagged as Open:** Joining fee vs. per-complaint exam fee tension; multi-tier fee scope variation; parallel visit creation race condition; Hold pending immediate pause vs. retroactive backdating; final naming of `CALENDAR_BASED` / `VISIT_BASED` and `duration_days` vs. `visits_promised`; binding vs. advisory patient package hint; patient-level default hint for `visit_type`.

* **[27 August 2026] — SPECIALTY-03 & SPECIALTY-04: Specialty Treatments, UI Workflow, Junctions & Invoicing Line Items**
  * **Date:** 27 August 2026 (Fee Calculation Logic)
  * **Topic Covered:** Structural package exclusion correction (separate money buckets); artificial anchor complaint issue; standalone `special_treatment_events` vs. junction `visit_service_complaint_links`; parent `sessions` table evaluation; frontend Procedure Logger multi-select; retraction of "zero linked complaints" (minimum 1 required); `invoices.visit_id` non-enforcement as 1:1; `invoice_line_items` child table linking to `visits_with_effective_charge`; downstream override blast radius resolution.
  * **Back-and-Forths (Q&A Cycles):** 5 sequenced architectural sections.
  * **Resolved:** Package exclusion confirmed structural; `sessions` table confirmed not to replace `patient_id + date` dedup scoping; specialty events require $\ge 1$ linked complaint; `invoices.visit_id` clarified as nullable non-1:1; proposed `invoice_line_items` child table; procedure logger UI requires only multi-select addition.
  * **Flagged as Open:** Junction table vs. `uuid[]` array column for treatment events; formal confirmation of `sessions` and `invoice_line_items` tables; production of dedicated `WORKFLOW.md` document.

---

## Section 2 — Decided/Closed Items

* Monetary values are stored exclusively as integer paise (`_in_paise` suffix) across all financial columns to prevent floating-point calculation errors.
* User roles (`'receptionist'`, `'clinician'`, `'admin'`) are stored as a text column on `profiles` with a `CHECK` constraint instead of a complex permission matrix.
* Multi-branch tenancy is modeled via `clinics.owner_id` pointing to an admin user's profile ID rather than introducing a separate `businesses` table.
* Transactional tables use UUID primary keys (`gen_random_uuid()`), while static seed/catalog tables (`services`, `complaint_catalog`) use human-readable text primary keys.
* Conditional referral doctor information is enforced at the database level via `chk_referral_doctor_info` on `patients`.
* Conditional consultation type information is enforced at the database level via `chk_consultation_type` on `visits`.
* `visits.grand_total_in_paise` is implemented as a stored generated column summing `consultation_fee_in_paise` and `services_total_in_paise`.
* Sequential, per-clinic human-readable invoice numbers (`INV-YYYY-XXXX`) are generated atomically via the `process_new_invoice()` trigger using `clinics.invoice_counter`.
* Multi-branch patient identity is unified chain-wide via `patients.owner_id`, with branch-specific attendance tracked through `patient_clinic_access`.
* Patient MRN uniqueness is constrained per chain via `UNIQUE (owner_id, mrn)`.
* Per-clinic service pricing overrides are managed via `clinic_service_prices` with application-level `COALESCE` fallback to `services.standalone_price_in_paise`.
* Patient-global tables (`patient_alerts`, `patient_vitals`, `clinical_notes`, `timeline_events`) automatically derive `owner_id` from `patients.owner_id` via triggers.
* `visit_services.clinic_id` is automatically derived from the parent `visits.clinic_id` via a database trigger.
* Branch attendance rows in `patient_clinic_access` are auto-provisioned inside the `process_new_complaint_course()` trigger on first complaint registration.
* Direct client writes (INSERT/UPDATE/DELETE) on `patient_clinic_access` are completely prohibited, reserving write access exclusively to `SECURITY DEFINER` triggers.
* Cross-chain tenancy isolation is verified inside `process_new_complaint_course()` by asserting `patient_owner IS NOT DISTINCT FROM clinic_owner`.
* The `complaint_courses_process_new` trigger fires `BEFORE INSERT OR UPDATE` to prevent unvalidated branch or chain reassignments on updates.
* Invoice trigger logic is split into `invoices_process_new` (INSERT-only number generation) and `invoices_validate_clinic_id` (UPDATE-only tenancy checks) to protect counter integrity.
* Trivial updates to `complaint_courses` and `invoices` short-circuit execution via `IS NOT DISTINCT FROM` checks on `clinic_id` and `patient_id`.
* Root ownership columns `patients.owner_id` and `clinics.owner_id` are validated via `validate_owner_is_admin()` to ensure they reference a profile with `role = 'admin'`.
* `clinics.owner_id` is defined as `NOT NULL`.
* DDL migration execution order defines the `validate_owner_is_admin()` trigger on `patients` immediately following the `patients` table definition.
* `insurer_name` is confirmed as an optional, unused nullable column that is safe to leave in place without urgent schema migration.
* `patients.clinician_id` and `complaint_courses.clinician_id` remain immutable registration records, while the currently treating clinician is derived on demand from `visits` via `DISTINCT ON`.
* Multi-clinician treatments for different complaint courses on the same day are natively supported by the existing schema without structural modifications.
* Transient network failures are classified for automatic client retries, while deterministic database constraint rejections must fail fast to user UI boundaries.
* `ON DELETE CASCADE` is scoped strictly to patient-ownership chains; organizational and reference chains (`profiles`, `clinics`, `complaint_courses`) enforce `RESTRICT`.
* Soft delete is handled by `patients.is_active boolean not null default true`; hard deletes via cascade are reserved for administrative erasure.
* `services` and `complaint_catalog` tables in the migration are unseeded DDL structures requiring separate data population.
* `profiles.id` shares PK with `auth.users(id)` enabling direct `owner_id = auth.uid()` comparisons without extra joins.
* Circular FK dependency between `clinics` and `profiles` is resolved by creating `clinics.owner_id` bare and adding the constraint via `ALTER TABLE` after `profiles` exists.
* Multiple FKs to `profiles` on a single table require distinct aliased joins (`owner`, `clinician`, `creator`).
* `IS DISTINCT FROM` is mandatory in PL/pgSQL trigger comparisons to handle NULL values correctly under SQL three-valued logic.
* Composite UNIQUE constraints (`owner_id, mrn`, `clinic_id, invoice_number`, `clinic_id, service_id`) scope uniqueness per owner or clinic boundary.
* Full 15-step topological schema creation sequence is strictly ordered by foreign key dependencies.
* Standardized response naming convention (`M{module}-R{number}`) adopted across all study modules.
* `clinics.invoice_counter` increments strictly at final submit when the `invoices` row is inserted.
* Admin clinic-switcher is a client-side display filter layered on full-chain RLS access, not a backend security boundary.
* Repeat-visit procedure pre-selection is supported via queryable joins on `visits` and `visit_services` without schema changes.
* `clinics.config` JSONB column is reserved for low-frequency UI feature toggles (`inventoryEnabled`, `appointmentsEnabled`), not billing rules.
* `visits.visit_type` lives on `visits`, allowing patients to freely mix `CONSULTATION` and `MACHINE_ONLY` visits.
* `MACHINE_ONLY` visits map directly to existing `services` + `visit_services` + `clinic_service_prices` catalog architecture.
* Package cancellation refund formula deducts attended days at the regular non-discounted daily rate (₹810 transcript example rejected).
* Environment files (`.env`) are rejected for business pricing; all rates must live in queryable database tables.
* Locality-based rate variations are handled via `clinic_id` scoping on pricing tables.
* First-time patient examinations are priced at ₹350 for the exam only, with same-day physical therapy billed separately at ₹200 per treated complaint course.
* Follow-up physical therapy visits are billed at a flat rate of ₹200 per active treated complaint course.
* Gap-return fee is structured as an additive flat ₹150 lapse penalty on top of standard per-complaint therapy rates, not a bundled ₹350.
* Fee calculation engine is structured as additive rules rather than branching formulas.
* Request-and-approve workflow is selected for fee waivers over password prompts or scrap-and-redo.
* `clinic_tier_rates` table stores daily rates, package rates, and tier-aware exam fees (both Regular and Rehab seeded at ₹350).
* Precedent citation corrected from `derive_clinic_id_from_patient()` to `derive_owner_id_from_patient()`.
* Override fields (`final_amount_in_paise`, `override_reason`, `override_by`, `override_status`) live directly on `visits` and `packages` without a separate table.
* Button label for package termination is confirmed as "Cancel Package".
* Reassigning package days is handled by honest ₹0 cancellation + new complaint course with waived visits.
* Build `clinics.default_visit_type` as a form pre-fill convenience; skip visit-type locking permissions.
* `patient_flags` established as a dedicated table separate from `patient_alerts`.
* Package refund preview uses the exact same backend calculation function as real cancellation.
* Pending fee overrides do not block visit creation (billed at full computed amount, adjusted retroactively upon approval).
* In plpgsql BEFORE INSERT triggers, `NEW` is an in-memory mutable candidate row evaluated before constraint checks.
* Multi-complaint procedure pre-fill is resolved in 2 queries using `DISTINCT ON (complaint_course_id)` across `visits` and `visit_services`.
* Patient-wise prescription table is rejected in favor of optional `clinic_default_services`.
* `final_amount_in_paise` must be nullable (not defaulted to computed total) to avoid silent staleness on data entry corrections.
* Three-way CHECK constraint governs override state transitions (`none`, `pending`, `approved`).
* Override authorization trigger must verify `auth.uid()` admin role and derive `override_by` directly from JWT identity.
* `visits.package_id` must be added as a nullable foreign key linking visits to packages.
* `patient_rate_overrides` table is redesigned with component-based rows (`CONSULTATION_FEE`, `THERAPY_RATE`, `FLAT_TOTAL`).
* Generated columns cannot reference other generated columns in Postgres.
* Query ergonomics for effective visit charge are solved via view `visits_with_effective_charge`.
* `visits.consultation_type` CHECK constraint to be widened to include `'GAP_RETURN'`.
* `attended_days` on packages is computed live at read time (`COUNT(*)`), not via a trigger-maintained counter.
* Package days used are computed live from calendar days minus closures and waivers (packages never mutate stored `expiry_date`).
* Reactive `clinic_closures` table and `package_day_waivers` table adopted for attendance math.
* Status flip (Active $\to$ Expired) is executed lazily at visit creation or detail view opening.
* Mandatory `WITH (security_invoker = true)` clause is required on all views in Postgres 15+ to prevent RLS bypass.
* Partial unique index `idx_one_active_package_per_complaint` enforces at most one active package per complaint course.
* Partial unique index on `patient_rate_overrides(patient_id, override_type) WHERE is_active` prevents duplicate active grants.
* Same-day multiple visits for a patient are handled via soft frontend warnings, not database CHECK constraints.
* `visits` money columns redesigned into four buckets: `exam_fee_in_paise`, `lapse_penalty_in_paise`, `therapy_fee_in_paise`, `services_total_in_paise`.
* Same-day flat fee dedup trigger is scoped to `patient_id + date`.
* Om's ₹180 case is resolved as a component override (`CONSULTATION_FEE = 0`, `THERAPY_RATE = 180`).
* `packages.expiry_date` is nullable/optional for both package modes (null represents indefinite duration).
* Package mode renamed from `SESSION_BASED` to `VISIT_BASED` to eliminate naming collision with `visit_type = 'CONSULTATION'`.
* Non-default package creation and overrides remain admin-only-final.
* Unique constraint `uq_one_visit_per_complaint_per_day` adopted on `visits(complaint_course_id, date)`.
* `exclude_sundays` boolean generalized to per-clinic recurring array `clinics.recurring_closed_weekdays integer[]` (default `{0}`).
* `clinics.consultation_fee_scope` column added with values `'PER_VISIT_DAY'` (default) and `'PER_COMPLAINT'`.
* Partial unique index `visits(complaint_course_id, date) WHERE therapy_fee_in_paise > 0` limits therapy visits to one per day.
* Examination fee modeled as a service catalog row with `clinic_fee_scopes` table separating amount from sharing scope.
* Specialty services multi-complaint linking recommended via `visit_service_complaint_links` junction table.
* Specialty pricing mode (flat-once vs per-complaint) configured per service or via `clinic_service_prices`.
* Package-id derivation trigger on `visits` matches active package, zeroes `therapy_fee_in_paise`, and increments `attended_days`.
* Auto-detecting clinic closure from staff login absence (Approach C(i)) is evaluated and rejected.
* Daily login onboarding prompt with `closure_type` (`FULL` / `NO_PACKAGE_PATIENTS`) adopted as UI layer on `clinic_closures`.
* Package override workflow creates package at clinic default immediately and applies override upon admin approval.
* Package mode switch takes effect strictly going forward upon approval without retroactively reinterpreting past days.
* Pending Cancel/Refund requests block new visits from drawing against that package.
* CORRECTION: Package exclusion of specialty treatments is structural (different money buckets), not a category filter.
* `sessions` table does not replace `patient_id + date` fee-dedup scoping for flat fees.
* CORRECTION: Specialty treatment events require at least one linked complaint by default (zero-row allowance retracted).
* `invoices.visit_id` was never schema-enforced 1:1 with visits (nullable, no unique constraint).
* Proposed `invoice_line_items` child table with per-complaint attribution reading from `visits_with_effective_charge` view.
* Frontend Procedure Logger requires no new screen/route for specialty logging (adds complaint multi-select on specialty selection).

---

## Section 3 — Partially Discussed / Open Threads

* **`patients.owner_id` Derivation Gap:**
  > *"No trigger derives `patients.owner_id` automatically. Frontend is fully responsible for computing and submitting the correct value; no database backstop beyond 'is this some admin.'"*

* **`patients.clinician_id` Role Validation Gap:**
  > *"`patients.clinician_id uuid references profiles(id)` checks only that the id exists in `profiles`. Nothing checks that the profile it points to actually has `role = 'clinician'`."*

* **`patient_alerts` Table Redundancy vs Frontend Mockup:**
  > *"Patient_alerts — Om doesn't remember a real frontend form for this and suspects it leaked in from an old mockup type... Possibly fully redundant with `clinical_notes` + `is_critical`. Not resolvable from the docs alone — needs a direct check against the actual frontend code."*

* **Staff Clinic ID Derivation from `auth.uid()`:**
  > *"Should `clinic_id` on staff-submitted writes (visits, complaint_courses, packages) be derived server-side from the submitting user's own `profiles.clinic_id` (via `auth.uid()`), instead of trusted-then-validated from client input... Complication: `profiles.clinic_id` is NULL for admins by design... Complication: `auth.uid()` is NULL when running raw SQL directly in the Supabase SQL Editor."*

* **Admin RLS Policies for Visits and Invoices:**
  > *"One honest gap here: the docs I have only draft the admin RLS policy for patients; there's no equivalent drafted yet for invoices or visits. Worth remembering this is genuinely undesigned territory, not something already quietly solved."*

* **Denormalized Snapshot Validation Gap:**
  > *"the database guarantees the relationship exists — never that the denormalized copy of it is accurate. That's entirely the frontend's job... nothing downstream catches it."*

* **Clinic Default Services Configuration Granularity:**
  > *"Clinic-wide, or clinic + clinician? You said it yourself — 'doctors usually have a default go-to,' which is really a claim about doctors, not clinics as a whole... Worth deciding deliberately once you have a better read on whether your actual clinicians' habits really diverge enough to matter."*

* **`patient_flags` Authorization Level:**
  > *"should flagging be admin-only, like Hold/Cancel, or can any staff member do it? Front-desk staff are the ones who actually hit the friction this feature exists for — but it's also a weighty, permanent, visible judgment about a specific person. Your call."*

* **Joining Fee vs Per-Complaint Exam Fee Conflict:**
  > *"is the January/June behavior still what you want (in which case the fee is really scoped to 'each complaint's own first touchpoint,' not 'this patient's first-ever visit,' and 'joining fee' needs a different name) — or are you actually reconsidering that rule now, in which case a returning patient with a new complaint would need to stop paying a fresh exam fee, which is a real behavior change from what's currently logged?"*

* **`packages.missed_days` Read-Time Derivation Rules:**
  > *"missed_days is genuinely harder — a missed day has no visit row to count... depends on exactly how 'missed' gets decided in the real workflow (only once a day is unambiguously past with nothing logged, or can staff mark a day missed in advance?), which isn't clear enough yet to commit to one design."*

* **`packages.day_log` Source of Truth Consolidation:**
  > *"day_log has the same underlying issue as attended_days/missed_days — untouched by any trigger, a third independent representation of the same fact. Worth eventually picking one source of truth... rather than three peers that can silently disagree."*

* **Hold Pending Approval Workflow Behavior:**
  > *"either (a) a hold request takes immediate, provisional effect the moment it's submitted — which does protect the days in real time, but means a receptionist can unilaterally pause a package's clock pending review... or (b) nothing happens while pending, and if approved, the admin's approval retroactively backdates the waiver across however many days were pending..."*

* **Final Naming for Package Modes & Counters:**
  > *"Final naming for CALENDAR_BASED / VISIT_BASED (and duration_days vs. visits_promised) — Om's call."*

* **Enforcement of `patients.default_package_mode_hint`:**
  > *"Whether patients.default_package_mode_hint should ever be enforced/binding rather than purely advisory."*

* **Conversion of Logged Visit to Day 1 of Package:**
  > *"what if a patient gets treated today, pays the normal ₹200 fee, and then tomorrow decides they want to buy a 10-day package and have yesterday's visit count as Day 1?... Deferred for now — design the baseline package logic first."*

* **Cross-Branch Package Redemption:**
  > *"Package purchased at Clinic A used at Clinic B: not discussed. The schema enforces `packages.clinic_id` matching `visits.clinic_id`, meaning a package is tied to one location... If cross-branch packages are ever needed, that's a new conversation."*

* **Downstream Invoice Blast Radius from Visit Overrides:**
  > *"Invoices are created as a downstream step from visits. If a visit has an override... what amount goes onto the invoice?... If an invoice is generated before an override is approved, does the invoice mutate when the visit is updated?"*

* **Enforcement of Active Status on Complaint Courses:**
  > *"Should `visits` insert be blocked if the referenced `complaint_courses` row has `status = 'Completed'`?"*

* **Registration permissions via RLS:**
  > *"Should receptionists/clinicians be allowed to register patients directly via RLS insert policy, or admins only?"*

* **Mandatory demographics constraints:**
  > *"Should `patients.phone` / `patients.address` / `patients.clinician_id` become `NOT NULL` to enforce registration completeness?"*

* **Multi-tenant deployment architecture:**
  > *"One Supabase project per clinic chain vs. shared multi-tenant database (determines whether the cross-chain gap and a `businesses` table ever matter)."*

* **Multi-admin chain support:**
  > *"Multiple-admin support: Lightweight sub-admin association on `profiles` vs. full `businesses` table (former is cheaper)."*

* **Scope of retroactive visit editing and audit trail:**
  > *"Scope of retroactive editing: Recommend allowing edits on attribution fields (`clinician_id`, `complaint` text) while keeping financial fields (`fees`, `amounts`, `payment_status`) immutable (matching `grand_total_in_paise` / `invoice_number` philosophy). If built, add `updated_at` to `visits` / `invoices` first at minimum."*

* **Unconstrained patient blood type:**
  > *"Unconstrained blood type: `patients.blood_type` has no `CHECK` constraint — any arbitrary string is accepted."*

* **Staff clinic write scoping enforcement:**
  > *"No enforcement exists yet — at any layer — that a receptionist/clinician can only write records at their own assigned clinic... Today, a Clinic 1 receptionist can submit `clinic_id = Clinic 2` on a write and the trigger layer will validate it against the patient's access, never against the submitting staff member's own clinic."*

* **Visit-to-invoice submission coordination:**
  > *"Whether the visit→invoice write happens as one coordinated backend action (complaint_course if new → visit → visit_services → invoice, all fired together at a single 'submit' point) or as several progressive writes across the UI flow is not confirmed against real frontend/API code."*

* **Non-itemized visit type naming:**
  > *"The other visit type still needs a name — you flagged this yourself, 'consultation therapy' isn't final. This is the non-itemized path: the clinician picks whichever machines a complaint needs, all under one flat fee, no per-machine pricing."*

* **Rehab tier exam fee independence:**
  > *"Does the ₹350 exam fee itself change for a rehab complaint, or does only the therapy/package side change and the exam stays flat regardless of tier?"*

* **Referring doctors data representation:**
  > *"Are referring doctors worth their own table, or does free text stay correct forever?"*

* **Daily front-desk visit lifecycle realities:**
  > *"Does 'a visit row existing = session complete and paid' really match how your front desk works, or are there real in-between states?"*

* **Complaint and procedure catalog usability:**
  > *"Is the complaint/procedure catalog wired the way you actually want it used day to day?"*

* **Paper verification checklist execution:**
  > *"Verification Checklist (Not Yet Executed): Everything above is paper review. None of it has been run against a real database yet. This checklist is the actual finish line — check each box only after watching it happen, not after reading the trigger code again."*

---

## Section 4 — Not Yet Discussed

* **Tables & Columns Present in Migration with Zero Discussion Log Coverage:**
  * **`complaint_catalog`**: Detailed anatomy of region values (`'Spine'`, `'Shoulder'`, `'Knee'`, `'Hip'`, `'Elbow'`, `'Ankle'`, `'Neuro'`), soft-deletion lifecycle via `is_active`, seed dataset ingestion mechanics, and text primary key conventions (`CAT_S01`).
  * **`services`**: Category partitioning (`'STANDARD'` vs. `'PREMIUM'`), baseline standalone price defaults, seed dataset ingestion mechanics, and text primary key conventions (`SVC-01`).
  * **`patient_vitals`**: Individual vital types (`'BP'`, `'HR'`, `'TEMP'`, `'SPO2'`), vital trends (`'NORMAL'`, `'HIGH'`, `'LOW'`), unit formatting, composite text readings (e.g. `"118/76"`), and timestamp recording via `recorded_at`.
  * **`clinical_notes`**: Categorization schema (`category`), observation text structure, `is_critical` boolean weighting, and staff author attribution (`created_by`).
  * **`timeline_events`**: Event classification categories (`'PHYSIO'`, `'CONSULT'`, `'LAB'`, `'MEDICATION'`, `'SURGERY'`, `'NOTE'`), polymorphic-style dual foreign key nullable links (`visit_id`, `note_id`), and default text descriptions.
  * **`clinics` Unused Columns**: `default_currency` (`'INR'`), `default_timezone` (`'Asia/Kolkata'`), `payment_methods_accepted` (`text[]` array `['CASH', 'UPI', 'CARD']`), `multi_therapist` boolean flag, and `color_scheme` string identifier.
  * **`patients` Columns**: `photo_url`, `is_active` soft-delete flag, `date_of_birth` vs. age calculation, and `last_visit_at` timestamp maintenance.

* **Triggers & Database Functions from Migration Not Yet Covered in Study Sessions:**
  * **`derive_owner_id_from_patient()`**: PL/pgSQL function internals and execution mechanics across `patient_alerts`, `patient_vitals`, `clinical_notes`, and `timeline_events` (scheduled for Module 4).
  * **`validate_clinic_id_from_access()`**: PL/pgSQL validation execution mechanics on `packages` (scheduled for Module 2/4).
  * **`derive_clinic_id_from_visit()`**: PL/pgSQL derivation execution mechanics on `visit_services`.
  * **`trigger_set_timestamp()`**: PL/pgSQL reusable timestamp function attached to `clinics`, `profiles`, `patients`, `complaint_courses`, `clinic_service_prices`, and `packages` (scheduled for Module 4).

* **Database Engine Features, Extensions & Constraints Not Yet Covered:**
  * **PostgreSQL Extension `pg_trgm` & GIN Trigram Indexing**: `CREATE EXTENSION IF NOT EXISTS "pg_trgm"` and `idx_patients_full_name_trgm` (`patients USING gin (full_name gin_trgm_ops)`) for fuzzy text search.
  * **B-Tree Secondary Indexes**: The 26 specific single-column and composite B-Tree indexes across the schema (`idx_patients_phone`, `idx_patient_vitals_patient_id`, `idx_visits_date`, `idx_invoices_date`, `idx_visit_services_visit_id`, etc.).
  * **Array Data Types & Constraints**: Array storage and validation for `clinics.payment_methods_accepted` (`text[]`).
  * **Database View Mechanics**: `daily_ledger` SQL query projection, column casting (`v.created_at::time as time`), execution performance, and virtual query vs. stored table behavior (scheduled for Module 3).
  * **Foreign Key `ON DELETE` Cascade Architecture**: The 11 specific `ON DELETE CASCADE` paths vs. `NO ACTION`/`RESTRICT` integrity protections across profiles and clinics (scheduled for Module 2).

* **Study Plan Modules 2–10 Topics Not Yet Discussed:**
  * **Module 2**: Hands-on relational integrity exercises; `ON DELETE CASCADE` behavioral verification; seed/reference table semantics vs. transactional table lifecycle.
  * **Module 3**: Hands-on generated column write-rejection testing (`visits.grand_total_in_paise`); `daily_ledger` query plan analysis vs. manual `JOIN`.
  * **Module 4**: PL/pgSQL trigger execution anatomy; `BEFORE` vs. `AFTER` event phases; `NEW` vs. `OLD` record manipulation; silent field mutation vs. exception raising.
  * **Module 5**: Direct-to-Postgres security architecture; request-scoped JWT token extraction via `auth.uid()`; `USING` (read filter) vs. `WITH CHECK` (write validation); transitioning from `v1_allow_all` development policies to production policies (`staff_patients`, `admin_patients`).
  * **Module 6**: `SECURITY DEFINER` security boundary implications; RLS bypass mechanics; manual testing of trigger-level cross-chain exceptions.
  * **Module 7**: End-to-end trace from receptionist authentication $\rightarrow$ JWT $\rightarrow$ RLS policy evaluation $\rightarrow$ database row filtering.
  * **Module 8a**: Live mechanical verification of the 5-point checklist on a real Supabase test environment.
  * **Module 8b**: Domain stress-testing and formal resolution of the 4 open business questions (referring doctors table vs. text, visit write lifecycle realism, catalog daily usage ergonomics, cross-branch patient identity validation).
  * **Module 9**: Migration versioning discipline using Supabase CLI (`supabase migration new`); safe vs. planned vs. destructive schema changes; structural (🔴) vs. safety-net (🟡) risk classification.
  * **Module 10**: Postgres structured exception mapping (`RAISE EXCEPTION`, CHECK/FK failures) through the Supabase JavaScript client into frontend error boundaries.

---

## Section 5 — Velocity Data

* **Total Discussion Sessions:** 25 distinct discussion passes and sessions (7 in Module 1, 6 in Module 2, 12 in Business Logic / Fee / Package / Specialty logic) across 21 dated sessions and 12 pre-migration review iterations.
* **Calendar Days Span:** 14 active calendar days with recorded discussions (Aug 2, Aug 3, Aug 4, Aug 5, Aug 6, Aug 8, Aug 10, Aug 13, Aug 14, Aug 16, Aug 17, Aug 18, Aug 19, Aug 27, 2026) spanning a 26-calendar-day window (2 August 2026 – 27 August 2026).
* **Meaningfully Resolved Topics:** 83 distinct architectural, relational, and business pricing decisions resolved across all audited files.
* **Rough Average Resolved Topics per Session:** $\approx 3.32$ meaningfully resolved topics per discussion session ($\approx 5.93$ resolved topics per active calendar day).
