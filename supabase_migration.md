# Supabase Migration SQL — Clinic Demo v2

> **How to use**: Paste the entire SQL block into Supabase SQL Editor → New Query → Run.
>
> **Cross-reference**: Every column is justified in [schema_cross_reference.md](file:///C:/Users/HP/.gemini/antigravity/brain/3545fb0f-2af4-42f0-a0aa-010e4af11f9d/schema_cross_reference.md).

---

## What Changed from v1 → v2 → v3

| Change | Why |
|---|---|
| **New `clinics` table** | 3 locations, one owner. Every clinical table needs `clinic_id` for isolation. |
| **`clinic_settings` removed** | Merged into `clinics`. Each location has its own fees, Sunday policy, color scheme. |
| **`clinic_id` on every clinical table** | Without it, RLS policies can't isolate location data. |
| **`profiles.clinic_id` nullable** | Admins span all locations. Receptionists/clinicians are scoped to one. |
| **`complaint_catalog_id` FK on `complaint_courses`** | Catalog table was dead weight with no connection to real patient records. |
| **Conditional CHECK constraints** | `referral_doctor_info` / `consultation_type` enforced at DB level. |
| **`grand_total_in_paise` GENERATED** | Cannot drift — always `consultation_fee + services_total`. |
| **UUID PKs on transactional tables** | Eliminates app-side collision-free ID generation. |
| **`invoice_number` + `invoice_counter`** | Sequential, per-clinic, collision-free via trigger. |
| **`timeline_events` FK fixed** | `doctor_name`/`doctor_initials` replaced with `clinician_id` FK. |
| **(v3) `clinic_id` guard triggers** | `clinic_id` derived from parent row — never trusted from client. |
| **(v3) `visit_services.clinic_id`** | Missing in v2. Added, derived from `visits.clinic_id` via trigger. |
| **(v3) `clinic_service_prices` table** | Per-clinic price overrides with fallback to base price. |
| **(v4 BUGFIX) `timeline_events` guard trigger** | Was missing. Now `owner_id` derived from `patient_id` like every other patient-global table. |
| **(v4 BUGFIX) Invoice triggers merged** | Two triggers merged into one `process_new_invoice()`. Removes fragile alphabetical-ordering dependency. |
| **(v4) Patient cross-branch identity** | `patients.clinic_id` removed. `owner_id` (chain-level) + `patient_clinic_access` junction. |
| **(v5 BUGFIX) Stale index on removed column** | `idx_patients_clinic_id` referenced removed `patients.clinic_id`. Migration would fail. Fixed to `idx_patients_owner_id`. |
| **(v5 BUGFIX) No-op guard triggers restored** | `derive_clinic_id_from_patient()` was gutted to a no-op in v4. `visits`, `complaint_courses`, `packages` had zero validation. Replaced with real trigger functions. |
| **(v5 BUGFIX) Admin RLS policy** | Was broken — queried `profiles.clinic_id` which is NULL for admins. Simplified to `owner_id = auth.uid()`. |
| **(v5) Auto-provision `patient_clinic_access`** | `process_new_visit()` trigger upserts access row on every visit insert. App never needs a separate step. |
| **(v6 BUGFIX) Deadlock in first-visit flow** | `complaint_courses` validated against `patient_clinic_access` which didn't exist yet; `visits` needed a complaint_course that couldn't be created. Circular. Fixed: `complaint_courses` now auto-provisions the access row (`process_new_complaint_course`). `visits` is simplified to validate-only. |

---

## Conventions

| Convention | Rule |
|---|---|
| **Money** | Integer paise. `_in_paise` suffix on every money column. ₹300 = `30000`. |
| **Multi-tenancy** | Every clinical table has `clinic_id uuid references clinics(id)`. RLS policies use this to isolate locations. |
| **PKs** | Transactional tables: `uuid default gen_random_uuid()`. Reference/seed tables: `text` (matches existing catalog IDs). |
| **Enums** | `text` + `CHECK` constraint — not Postgres `CREATE TYPE`. Easier to add values later without migrations. |
| **Audit** | Clinical entities: `clinician_id`. All writes: `created_by`. |
| **Timestamps** | All `timestamptz`, stored UTC, displayed as `Asia/Kolkata` in frontend. |

---

## The SQL

```sql
-- ============================================================================
-- 0. UTILITY FUNCTION: updated_at auto-stamp
-- Source: mock_data.tsx L871-875
-- ============================================================================

create or replace function trigger_set_timestamp()
returns trigger as $$
begin
  new.updated_at = now();
  return new;
end;
$$ language plpgsql;


-- ============================================================================
-- 1. CLINICS
-- NEW in v2. One row per clinic location.
-- Source: User context — "chain of 3 with one owner/admin"
-- Absorbs clinic_settings from v1 (dropped). Each location has its own
-- fees, timezone, Sunday policy, color scheme, and feature flags.
-- ============================================================================

create table clinics (
  id           uuid primary key default gen_random_uuid(),
  -- Human-readable name for each location
  name         text not null,
  address      text not null default '',
  phone        text not null default '',

  -- Owner/chain grouping. All 3 locations share the same owner_id.
  -- This is the handle for admin-level cross-clinic queries.
  -- Source: "one owner/admin" context.
  -- DECISION (v10): NOT NULL. A clinic must always have an owner from creation.
  -- Was nullable only to work around DDL ordering (clinics table created before
  -- profiles). The deferred FK (alter table add constraint below) handles that
  -- ordering concern without requiring the column itself to be nullable.
  owner_id     uuid not null,  -- FK to profiles(id) wired below after profiles exists

  -- Per-location settings (previously clinic_settings)
  -- Source: comment_1.md clinicConfig
  default_currency            text not null default 'INR',
  default_timezone            text not null default 'Asia/Kolkata',

  -- Source: Visit type L141 — fees may differ per location
  -- AR7: paise. ₹300 = 30000.
  consultation_fee_first_in_paise      integer not null default 30000,
  consultation_fee_subsequent_in_paise integer not null default 20000,

  -- Source: PackageRecord.excludeSundays — clinics vary on Sunday policy
  exclude_sundays_default boolean not null default true,

  -- Source: Issue #70 body
  payment_methods_accepted text[] not null default array['CASH', 'UPI', 'CARD'],

  -- Source: comment_1.md — multiTherapist feature flag. Per-location.
  multi_therapist boolean not null default false,

  -- Source: User request — color blindness accommodation. Per-location.
  color_scheme text not null default 'default',

  -- DECISION: Sequential invoice counter per clinic.
  -- Incremented atomically by the generate_invoice_number() trigger on invoices.
  -- Prevents two concurrent inserts from generating the same invoice number.
  invoice_counter integer not null default 0,

  -- Additional feature flags as JSONB (rarely accessed, low-frequency reads)
  -- e.g. { "inventoryEnabled": false, "appointmentsEnabled": false }
  config jsonb not null default '{}'::jsonb,

  is_active  boolean not null default true,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create trigger clinics_updated_at
  before update on clinics
  for each row execute procedure trigger_set_timestamp();




-- ============================================================================
-- 2. PROFILES (extends Supabase auth.users)
-- Source: clinic_more_issues.md §3 — "users table has role enum"
-- v2 change: clinic_id added. Nullable because admins span all locations.
-- ============================================================================

create table profiles (
  id           uuid primary key references auth.users(id) on delete cascade,
  display_name text not null default '',
  role         text not null default 'receptionist'
                 check (role in ('receptionist', 'clinician', 'admin')),

  -- DECISION (v2): clinic_id is nullable.
  -- Receptionists and clinicians are scoped to ONE location (clinic_id set).
  -- Admins span all locations (clinic_id null — they see everything).
  -- Source: "staff/receptionist won't see other clinic's records, but the admin"
  clinic_id    uuid references clinics(id),

  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now()
);

create trigger profiles_updated_at
  before update on profiles
  for each row execute procedure trigger_set_timestamp();

-- Now wire the clinics.owner_id FK (profiles exists now).
-- DECISION: Deferred FK rather than forward-declare. clinics is created before
-- profiles to avoid a circular dependency (profiles.clinic_id references clinics).
-- clinics.owner_id is NOT NULL but the FK constraint is added here, after profiles.
alter table clinics
  add constraint clinics_owner_id_fkey
  foreign key (owner_id) references profiles(id);


-- ============================================================================
-- VALIDATE OWNER IS ADMIN
-- DECISION (v10): owner_id on both patients and clinics is the root of the
-- entire tenancy model. The admin RLS policy (owner_id = auth.uid()) only
-- works correctly if owner_id actually points to a profile with role='admin'.
-- Without this trigger, a client could set owner_id to a receptionist or
-- clinician's profile. No error at write time — the patient would silently
-- disappear from the admin's view because owner_id = auth.uid() would never
-- match. Impossible to diagnose without knowing to check this column.
--
-- Reused on both tables: same column name, same validation rule.
-- ============================================================================

create or replace function validate_owner_is_admin()
returns trigger as $$
begin
  if not exists (
    select 1 from profiles
    where id = new.owner_id
      and role = 'admin'
  ) then
    raise exception
      'owner_id % must reference a profile with role = ''admin''', new.owner_id;
  end if;
  return new;
end;
$$ language plpgsql;

create trigger clinics_owner_is_admin
  before insert or update on clinics
  for each row execute procedure validate_owner_is_admin();

-- patients_owner_is_admin is defined AFTER the patients table below.
-- Postgres requires the table to exist before a trigger can be attached to it.


-- ============================================================================
-- 3. PATIENTS
-- Source: Patient type (types/index.tsx L11-27), PatientProfile type (L95-102)
-- v4 change: PATIENT CROSS-BRANCH IDENTITY REDESIGN
--
-- DECISION (v4): A patient is one shared identity across all branches of the
-- same chain. User decision: "1 shared identity per business so admin can see
-- one patient record, but staff should only see records of their limited branch".
--
-- Implementation:
--   - patients.clinic_id REMOVED. patients now belongs to a chain (owner_id),
--     not to a single clinic.
--   - patient_clinic_access junction table (added below) tracks which branches
--     a patient has actually attended.
--   - Staff RLS: patient visible if patient_clinic_access has a row for that
--     patient at the staff member's clinic.
--   - Admin RLS: patient visible if patients.owner_id matches the admin's chain.
--   - MRN is now unique per chain (unique(owner_id, mrn)), not per clinic.
--
-- Records that are LOCATION-SPECIFIC (tied to a branch):
--   visits, complaint_courses, packages, invoices, visit_services
--   → still have clinic_id, meaning "recorded at this branch"
--
-- Records that are PATIENT-GLOBAL (belong to the patient, not a branch):
--   patient_alerts, patient_vitals, clinical_notes, timeline_events
--   → lose clinic_id, gain owner_id for chain-level isolation
-- ============================================================================

create table patients (
  id uuid primary key default gen_random_uuid(),

  -- DECISION (v4): owner_id replaces clinic_id.
  -- Identifies which chain/owner this patient belongs to.
  -- FK to profiles(id) where role = 'admin'. Set at registration time.
  -- Allows cross-branch admin queries: "show me all patients in my chain."
  --
  -- ⚠ MODEL LIMITATION (noted v7): "chain identity" = a single admin's user id.
  -- Nothing in the schema constrains owner_id to profiles where role = 'admin'.
  -- Works correctly for one admin per chain. If a second admin for the same chain
  -- is ever needed, this model (one owner_id per chain) needs revisiting.
  -- Not urgent today — noted so it doesn't surprise the next schema revision.
  owner_id uuid not null references profiles(id),

  -- MRN unique within a chain, not per-clinic.
  -- DECISION (v4): changed from unique(clinic_id, mrn) to unique(owner_id, mrn).
  -- Same patient can't accidentally get two different MRNs at two branches.
  mrn text not null,

  full_name     text not null,
  phone         text,
  address       text,
  date_of_birth date,
  -- DECISION: 'M'|'F'|'X'. Mock data 'female'/'male' mapped at seeding.
  gender        text check (gender in ('M', 'F', 'X')),
  last_visit_at timestamptz,
  is_active     boolean not null default true,

  referral_mode text check (referral_mode in ('WALKIN', 'GOOGLE', 'DOCTOR')),
  referral_doctor_info text,
  constraint chk_referral_doctor_info check (
    (referral_mode = 'DOCTOR' and referral_doctor_info is not null)
    or (referral_mode != 'DOCTOR' and referral_doctor_info is null)
    or referral_mode is null
  ),

  blood_type   text,
  insurer_name text,
  photo_url    text,

  clinician_id uuid references profiles(id), -- registering clinician
  created_by   uuid references profiles(id),

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),

  unique (owner_id, mrn)
);

create trigger patients_updated_at
  before update on patients
  for each row execute procedure trigger_set_timestamp();

-- DECISION (v10/v11): trigger placed here (not with the function definition above)
-- because CREATE TRIGGER requires the table to already exist. The function
-- validate_owner_is_admin() is defined earlier and shared with clinics.
create trigger patients_owner_is_admin
  before insert or update on patients
  for each row execute procedure validate_owner_is_admin();


-- ============================================================================
-- 3a. PATIENT_CLINIC_ACCESS (junction: which branches a patient has attended)
-- NEW in v4. The mechanism that lets staff at Branch A see ONLY patients who
-- have come to Branch A, even though the patient record is chain-level.
--
-- A row is inserted here when:
--   (a) A new patient is registered at a branch (app-side, on registration)
--   (b) An existing patient from another branch is seen at a new branch
--       (receptionist confirms the shared identity, then creates an access row)
--
-- RLS policy for staff reads patients as:
--   patient_id IN (
--     SELECT patient_id FROM patient_clinic_access
--     WHERE clinic_id = (SELECT clinic_id FROM profiles WHERE id = auth.uid())
--   )
-- RLS policy for admin reads patients as:
--   owner_id = (SELECT owner_id_of_this_admin)
-- ============================================================================

create table patient_clinic_access (
  id         uuid primary key default gen_random_uuid(),
  patient_id uuid not null references patients(id) on delete cascade,
  clinic_id  uuid not null references clinics(id),
  -- Date this patient first attended this branch
  first_visit_date date not null default current_date,
  created_at timestamptz not null default now(),
  unique (patient_id, clinic_id)
);

create index idx_patient_clinic_access_clinic   on patient_clinic_access(clinic_id);
create index idx_patient_clinic_access_patient  on patient_clinic_access(patient_id);



-- ============================================================================
-- 4. PATIENT_ALERTS
-- Source: PatientAlert type (types/index.tsx L71-74)
-- v4 change: PATIENT-GLOBAL record. clinic_id removed, owner_id added.
-- Alerts belong to the patient (e.g. Penicillin allergy), not a specific branch.
-- All staff in the chain who can see the patient can see their alerts.
-- ============================================================================

create table patient_alerts (
  id         uuid primary key default gen_random_uuid(),
  -- owner_id: chain-level isolation. Derived from patient.owner_id by trigger.
  owner_id   uuid not null references profiles(id),
  patient_id uuid not null references patients(id) on delete cascade,
  type       text not null check (type in ('ALLERGY', 'FALL_RISK', 'DNR', 'OTHER')),
  label      text not null,
  created_at timestamptz not null default now()
);

-- ============================================================================
-- GUARD FUNCTIONS
--
-- Two families:
--   (A) derive_owner_id_from_patient  — for patient-global tables (alerts, vitals,
--       notes, timeline). Derives owner_id from patients.owner_id.
--   (B) Validation / access-provisioning for location-specific tables:
--       • process_new_complaint_course  — complaint_courses (auto-provision access)
--       • validate_visit_clinic_id      — visits (validate-only; access already exists)
--       • validate_clinic_id_from_access — packages (validate-only)
--
-- visit_services uses derive_clinic_id_from_visit (derives from parent visit).
-- invoices use process_new_invoice (validates + generates invoice number).
--
-- DEPENDENCY ORDER for a new patient's first visit at a branch:
--   1. patients (INSERT) — no clinic dependency
--   2. complaint_courses (INSERT) — process_new_complaint_course creates access row
--   3. visits (INSERT) — access row now exists, validate_visit_clinic_id passes
--   4. packages, invoices, visit_services — all validate against existing access
-- ============================================================================

create or replace function derive_owner_id_from_patient()
returns trigger as $$
begin
  new.owner_id := (select owner_id from patients where id = new.patient_id);
  return new;
end;
$$ language plpgsql;

create trigger patient_alerts_owner_id_guard
  before insert or update on patient_alerts
  for each row execute procedure derive_owner_id_from_patient();


-- validate_clinic_id_from_access: pure validation, used by packages.
-- Rejects if no patient_clinic_access row exists for (patient_id, clinic_id).
-- Only called on tables where the access row is guaranteed to already exist
-- (complaint_course was created first, which created the access row).
create or replace function validate_clinic_id_from_access()
returns trigger as $$
begin
  if not exists (
    select 1 from patient_clinic_access
    where patient_id = new.patient_id
      and clinic_id = new.clinic_id
  ) then
    raise exception
      'Patient % has no access record at clinic %. Create a complaint course first.',
      new.patient_id, new.clinic_id;
  end if;
  return new;
end;
$$ language plpgsql;


-- ============================================================================
-- 5. PATIENT_VITALS
-- Source: VitalSign type (types/index.tsx L76-82)
-- v4 change: PATIENT-GLOBAL. clinic_id removed, owner_id added.
-- Vitals belong to the patient, not a specific branch.
-- ============================================================================

create table patient_vitals (
  id         uuid primary key default gen_random_uuid(),
  owner_id   uuid not null references profiles(id), -- derived by trigger
  patient_id uuid not null references patients(id) on delete cascade,
  type       text not null check (type in ('BP', 'HR', 'TEMP', 'SPO2')),
  value      text not null,  -- text: BP is "118/76"
  unit       text not null,
  recorded_at timestamptz not null,
  trend      text not null check (trend in ('NORMAL', 'HIGH', 'LOW')),
  created_at timestamptz not null default now()
);

create trigger patient_vitals_owner_id_guard
  before insert or update on patient_vitals
  for each row execute procedure derive_owner_id_from_patient();


-- ============================================================================
-- 6. CLINICAL_NOTES
-- Source: ClinicalNote type (features/patient/types/index.tsx L18-22)
-- v4 change: PATIENT-GLOBAL. clinic_id removed, owner_id added.
-- Notes belong to the patient's medical record, visible across the chain.
-- ============================================================================

create table clinical_notes (
  id         uuid primary key default gen_random_uuid(),
  owner_id   uuid not null references profiles(id), -- derived by trigger
  patient_id uuid not null references patients(id) on delete cascade,
  category    text not null,
  observation text not null,
  is_critical boolean not null default false,
  created_by  uuid references profiles(id),
  created_at  timestamptz not null default now()
);

create trigger clinical_notes_owner_id_guard
  before insert or update on clinical_notes
  for each row execute procedure derive_owner_id_from_patient();


-- ============================================================================
-- 7. COMPLAINT_CATALOG (reference/seed table — NOT patient-specific)
-- Source: complaints_catalog.ts, MedicalComplaint type (types/index.tsx L30-39)
-- Kept as text PK — this is seed/reference data, seeded once, never mutated.
-- ============================================================================

create table complaint_catalog (
  -- Source: MedicalComplaint.id (L31) — "CAT_S01" etc.
  -- DECISION: text PK is appropriate here. This is reference data, seeded once,
  -- not generated by clinic operations. No collision risk.
  id        text primary key,
  -- Source: MedicalComplaint.title (L32)
  title     text not null,
  -- Source: MedicalComplaint.region (L38), from CATALOG_REGIONS in complaints_catalog.ts
  region    text check (region in ('Spine','Shoulder','Knee','Hip','Elbow','Ankle','Neuro')),
  -- Source: MedicalComplaint.isActive (L34)
  is_active boolean not null default true,
  created_at timestamptz not null default now()
);


-- ============================================================================
-- 8. COMPLAINT_COURSES (patient-specific treatment arcs)
-- Source: ComplaintCourse type (types/index.tsx L161-169)
-- v2 changes:
--   - UUID PK (was text in v1)
--   - clinic_id added
--   - complaint_catalog_id FK added — wires catalog to real patient records
-- ============================================================================

create table complaint_courses (
  id         uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null references clinics(id),
  -- Tenancy validated + patient_clinic_access auto-provisioned on INSERT by
  -- process_new_complaint_course (SECURITY DEFINER). UPDATE coverage added v8
  -- so a branch correction can't silently bypass the cross-chain check.
  patient_id uuid not null references patients(id) on delete cascade,

  -- Denormalized display snapshot. Kept because:
  -- (a) free-text custom complaints have no catalog entry
  -- (b) catalog complaints can be renamed without breaking history
  complaint_name       text not null,
  complaint_catalog_id text references complaint_catalog(id), -- nullable

  start_date     date not null,
  last_date      date not null,
  total_sessions integer not null default 0,
  status         text not null default 'Active'
                   check (status in ('Active', 'Completed')),
  clinician_id   uuid references profiles(id),

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create trigger complaint_courses_updated_at
  before update on complaint_courses
  for each row execute procedure trigger_set_timestamp();

-- DECISION (v6): complaint_courses is the true first clinical touchpoint.
-- A complaint course is registered before any visit can be created
-- (visits.complaint_course_id is NOT NULL). This makes complaint_courses
-- the correct and only place to auto-provision the patient_clinic_access row.
--
-- v5 put this on visits, which caused a deadlock: complaint_courses fired
-- validate_clinic_id_from_access() (requiring access to already exist),
-- but visits is what created the access row, and visits requires a complaint_course.
-- No legal order of operations existed for a new patient.
--
-- SECURITY DEFINER: needed to INSERT into patient_clinic_access when the
-- calling user (receptionist) may not have direct INSERT rights on that table.
--
-- DECISION (v7): Added cross-chain tenancy check (patient_owner vs clinic_owner).
-- This function runs SECURITY DEFINER, which means it bypasses RLS policies.
-- That makes it the enforcement boundary — it must validate tenancy itself.
-- Without this check, a receptionist at Chain B's clinic could create a
-- complaint_course for a Chain A patient (if they somehow had the patient UUID),
-- and this trigger would silently grant Chain B branch-level access to a
-- cross-chain patient via patient_clinic_access. RLS upstream won't save you
-- here because SECURITY DEFINER already bypassed it.
create or replace function process_new_complaint_course()
returns trigger as $$
declare
  patient_owner uuid;
  clinic_owner  uuid;
begin
  -- DECISION (v9): Early-exit on UPDATE when tenancy-relevant columns unchanged.
  -- Every trivial UPDATE (e.g. bumping total_sessions, updating status) previously
  -- ran two SELECTs + an upsert attempt even when clinic_id/patient_id didn't change.
  -- IS NOT DISTINCT FROM handles NULL safely (NULL = NULL is true here).
  if TG_OP = 'UPDATE'
     and new.clinic_id   is not distinct from old.clinic_id
     and new.patient_id  is not distinct from old.patient_id then
    return new;
  end if;

  -- Derive owner_id for both patient and clinic to enforce tenancy.
  select owner_id into patient_owner from patients where id = new.patient_id;
  select owner_id into clinic_owner  from clinics  where id = new.clinic_id;

  -- Clinic existence check: if owner_id is null, clinic doesn't exist.
  if clinic_owner is null then
    raise exception 'Clinic % does not exist', new.clinic_id;
  end if;

  -- Cross-chain tenancy check: clinic must belong to the same chain as patient.
  -- DECISION (v7): Using IS DISTINCT FROM instead of != to handle NULL safely.
  if patient_owner is distinct from clinic_owner then
    raise exception
      'Cross-chain violation: patient % belongs to a different chain than clinic %',
      new.patient_id, new.clinic_id;
  end if;

  -- Auto-register this patient at this clinic if it's their first complaint here.
  -- ON CONFLICT DO NOTHING: idempotent — second complaint at same branch doesn't error.
  insert into patient_clinic_access (patient_id, clinic_id)
  values (new.patient_id, new.clinic_id)
  on conflict (patient_id, clinic_id) do nothing;

  return new;
end;
$$ language plpgsql security definer;

-- DECISION (v8): Extended to `before insert or update`.
-- Every other guard trigger in this schema fires on both INSERT and UPDATE.
-- complaint_courses was the only exception — UPDATE was silently unguarded.
-- A "fix wrong branch" edit would bypass the cross-chain tenancy check and
-- skip the patient_clinic_access upsert. The function body handles both
-- operations correctly without change.
create trigger complaint_courses_process_new
  before insert or update on complaint_courses
  for each row execute procedure process_new_complaint_course();


-- ============================================================================
-- 9. SERVICES (global service/procedure catalogue)
-- Source: Service type (types/index.tsx L109-114), MOCK_SERVICES (13 entries)
-- Kept as text PK — seed data, seeded once.
-- ============================================================================

create table services (
  -- text PK: "SVC-01" through "SVC-13". Reference data, seeded once.
  id   text primary key,
  name text not null,
  -- AR7: paise. Base price — applies to all clinics unless overridden.
  standalone_price_in_paise integer not null,
  category text not null check (category in ('STANDARD', 'PREMIUM')),
  created_at timestamptz not null default now()
);


-- ============================================================================
-- 9a. CLINIC_SERVICE_PRICES (per-clinic price overrides)
-- NEW in v3. User confirmed locations may charge different prices per service.
-- Pattern: fall back to services.standalone_price_in_paise when no override row.
-- The app resolves effective price as:
--   COALESCE(clinic_service_prices.price_in_paise, services.standalone_price_in_paise)
-- ============================================================================

create table clinic_service_prices (
  id         uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null references clinics(id),
  service_id text not null references services(id),
  -- AR7: paise. Overrides the base price for this clinic only.
  price_in_paise integer not null,
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now(),
  -- One override row per (clinic, service) pair
  unique (clinic_id, service_id)
);

create trigger clinic_service_prices_updated_at
  before update on clinic_service_prices
  for each row execute procedure trigger_set_timestamp();


-- ============================================================================
-- 10. VISITS (one per complaint per session — V2 billing model)
-- Source: Visit type (types/index.tsx L130-145), MOCK_VISITS_V2
-- v2 changes:
--   - UUID PK (was text in v1)
--   - clinic_id added
--   - Conditional CHECK on consultation_type
--   - grand_total_in_paise is now a GENERATED column
-- ============================================================================

create table visits (
  id uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null references clinics(id), -- validated by trigger; access row guaranteed by complaint_course
  patient_id uuid not null references patients(id) on delete cascade,
  complaint_course_id uuid not null references complaint_courses(id),

  date      date not null,
  complaint text not null, -- denormalized display snapshot

  visit_type text not null check (visit_type in ('CONSULTATION', 'MACHINE_ONLY')),

  -- CHECK: consultation_type must be non-null iff visit_type = 'CONSULTATION'
  consultation_type text check (consultation_type in ('FIRST', 'SUBSEQUENT')),
  constraint chk_consultation_type check (
    (visit_type = 'CONSULTATION' and consultation_type is not null)
    or (visit_type = 'MACHINE_ONLY' and consultation_type is null)
  ),

  -- AR7: paise. Snapshot of fee at time of visit.
  consultation_fee_in_paise integer not null default 0,
  services_total_in_paise   integer not null default 0,
  -- GENERATED: can never drift from the sum of the two components
  grand_total_in_paise integer generated always as
    (consultation_fee_in_paise + services_total_in_paise) stored,

  -- MVP WORKFLOW: Row existing = session complete + paid. No status lifecycle.
  clinician_id uuid references profiles(id),
  created_by   uuid references profiles(id),
  created_at   timestamptz not null default now()
);

-- DECISION (v6): visits no longer auto-provisions patient_clinic_access.
-- That responsibility moved to complaint_courses (process_new_complaint_course),
-- which is always inserted before a visit (visits.complaint_course_id NOT NULL).
-- visits just validates that access already exists.
create or replace function validate_visit_clinic_id()
returns trigger as $$
begin
  if not exists (
    select 1 from patient_clinic_access
    where patient_id = new.patient_id
      and clinic_id = new.clinic_id
  ) then
    raise exception
      'Patient % has no access record at clinic %. Create a complaint course first.',
      new.patient_id, new.clinic_id;
  end if;
  return new;
end;
$$ language plpgsql;

create trigger visits_validate_clinic_id
  before insert or update on visits
  for each row execute procedure validate_visit_clinic_id();


-- ============================================================================
-- 11. VISIT_SERVICES (junction: which services used in a visit)
-- Source: VisitService type (types/index.tsx L117-127)
-- v2 change: UUID PK (was text in v1).
-- ============================================================================

create table visit_services (
  id uuid primary key default gen_random_uuid(),

  -- DECISION (v3): clinic_id added. Missing in v2. Derived from visits.clinic_id
  -- by the guard trigger below — never trusted from client.
  -- Needed because visit_services has no direct patient_id; joining through
  -- visits to scope it by location breaks the "every clinical table has clinic_id" rule.
  clinic_id  uuid not null references clinics(id),
  visit_id   uuid not null references visits(id) on delete cascade,
  service_id text not null references services(id),

  service_name     text not null, -- denormalized for display
  service_category text not null check (service_category in ('STANDARD', 'PREMIUM')),
  is_charged       boolean not null,
  charged_amount_in_paise integer not null default 0
);

-- Guard trigger: derive clinic_id from the parent visit row
create or replace function derive_clinic_id_from_visit()
returns trigger as $$
begin
  new.clinic_id := (select clinic_id from visits where id = new.visit_id);
  return new;
end;
$$ language plpgsql;

create trigger visit_services_clinic_id_guard
  before insert or update on visit_services
  for each row execute procedure derive_clinic_id_from_visit();


-- ============================================================================
-- 12. PACKAGES (time-based therapy packages)
-- Source: PackageRecord type (types/index.tsx L177-193), MOCK_PACKAGES
-- v2 changes:
--   - UUID PK (was text in v1)
--   - clinic_id added
--   - linked_complaint_id is now uuid FK (complaint_courses.id is uuid)
-- ============================================================================

create table packages (
  id uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null references clinics(id), -- validated by trigger against patient_clinic_access
  patient_id uuid not null references patients(id) on delete cascade,
  linked_complaint_id   uuid not null references complaint_courses(id),
  linked_complaint_name text not null, -- denormalized snapshot

  package_name    text not null,
  purchase_date   date not null,
  duration_days   integer not null,
  expiry_date     date not null,
  exclude_sundays boolean not null default true,
  attended_days   integer not null default 0,
  missed_days     integer not null default 0,
  amount_paid_in_paise integer not null default 0,

  status text not null default 'Active'
           check (status in ('Active', 'Completed', 'Expired')),

  -- JSONB array max 30 elements: ["attended","missed","upcoming",...]
  day_log jsonb not null default '[]'::jsonb,

  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);

create trigger packages_updated_at
  before update on packages
  for each row execute procedure trigger_set_timestamp();

create trigger packages_clinic_id_guard
  before insert or update on packages
  for each row execute procedure validate_clinic_id_from_access();


-- ============================================================================
-- 13. INVOICES
-- Source: InvoiceRecord type (types/index.tsx L195-201)
-- v2 changes:
--   - UUID PK (was text in v1)
--   - clinic_id added
--   - invoice_number: sequential, per-clinic, human-readable (e.g. INV-2026-0001)
--   - payment_status defaults to 'Paid' (MVP single-touch workflow)
--   - visit_id is now uuid FK
-- ============================================================================

create table invoices (
  id uuid primary key default gen_random_uuid(),
  clinic_id  uuid not null references clinics(id),
  -- INSERT: validated + number generated by process_new_invoice (SECURITY DEFINER, INSERT-only).
  -- UPDATE: tenancy re-validated by validate_invoice_clinic_id (UPDATE-only, no side effects).
  -- The two triggers are intentionally separate — merging them would re-mint invoice numbers on update.
  patient_id uuid not null references patients(id) on delete cascade,

  -- Sequential per-clinic human-readable number. Format: INV-2026-0001
  -- Generated atomically by trigger below. Unique per clinic (not globally).
  invoice_number text not null,
  unique (clinic_id, invoice_number),

  amount_in_paise integer not null,
  date            date not null,

  -- MVP: default 'Paid'. Receptionist creates one entry after session done + paid.
  -- 'Pending'/'Overdue' kept for future credit/package billing.
  payment_status text not null default 'Paid'
                   check (payment_status in ('Paid', 'Pending', 'Overdue')),
  payment_mode text,

  visit_id   uuid references visits(id), -- nullable: not all invoices link to a visit
  created_by uuid references profiles(id),
  created_at timestamptz not null default now()
);

-- DECISION (v5): invoices_clinic_id_guard removed. The old no-op trigger is
-- gone. Validation is handled entirely by process_new_invoice() below, which
-- validates clinic_id against patient_clinic_access AND generates the number.


-- ============================================================================
-- INVOICE NUMBER + CLINIC_ID COMBINED TRIGGER
-- DECISION (v4): Merged the former two separate triggers into ONE function.
-- Previous v3 had:
--   (1) invoices_clinic_id_guard  (c...) — derives clinic_id from patient
--   (2) invoices_generate_number  (g...) — reads new.clinic_id to increment counter
-- Postgres fires same-event BEFORE triggers in alphabetical order, so 'c' ran
-- before 'g' — but that was an INVISIBLE dependency. Rename either trigger and
-- the invoice counter silently increments against the wrong clinic with no error.
-- Merged into one function so the ordering is EXPLICIT in code, not the alphabet.
--
-- SECURITY NOTE: generate_invoice_number is marked SECURITY DEFINER so it can
-- update clinics.invoice_counter even after real RLS policies restrict direct
-- UPDATE on clinics to admin-only. Without SECURITY DEFINER, a receptionist
-- INSERT on invoices would fail when the trigger tries to UPDATE clinics.
-- ============================================================================

create or replace function process_new_invoice()
returns trigger as $$
declare
  next_num integer;
begin
  -- Step 1: Derive clinic_id from the patient row (guard against client tampering).
  -- For invoices, clinic_id represents which branch issued this invoice.
  -- We keep it explicit here rather than a separate trigger to ensure ordering.
  new.clinic_id := (select clinic_id
                    from patient_clinic_access
                    where patient_id = new.patient_id
                      and clinic_id = new.clinic_id
                    limit 1);

  if new.clinic_id is null then
    raise exception 'Patient % has no access record for clinic %', new.patient_id, new.clinic_id;
  end if;

  -- Step 2: Atomically increment the invoice counter for this clinic.
  update clinics
  set invoice_counter = invoice_counter + 1
  where id = new.clinic_id
  returning invoice_counter into next_num;

  -- Step 3: Format the human-readable invoice number.
  -- Format: INV-{year}-{zero-padded 4-digit counter per clinic}
  new.invoice_number := 'INV-' || to_char(now(), 'YYYY') || '-' || lpad(next_num::text, 4, '0');

  return new;
end;
$$ language plpgsql security definer; -- SECURITY DEFINER: allows trigger to UPDATE clinics even when caller can't

create trigger invoices_process_new
  before insert on invoices
  for each row execute procedure process_new_invoice();

-- DECISION (v9): separate UPDATE-only trigger for tenancy validation.
-- Cannot extend invoices_process_new to `before insert or update` — the
-- function increments invoice_counter and overwrites invoice_number on every
-- call. Running it on UPDATE would re-mint the invoice number whenever any
-- field (payment_status, payment_mode, etc.) is changed — corrupting the
-- counter sequence and detaching the printed number from the DB row.
--
-- Instead: keep INSERT handling in process_new_invoice (number generation,
-- counter increment), and add this side-effect-free UPDATE-only guard.
-- It short-circuits immediately when neither clinic_id nor patient_id changed
-- (the common case: marking an invoice paid does zero work).
create or replace function validate_invoice_clinic_id()
returns trigger as $$
begin
  -- Short-circuit: nothing tenancy-relevant changed, skip all checks.
  if new.clinic_id   is not distinct from old.clinic_id
     and new.patient_id is not distinct from old.patient_id then
    return new;
  end if;

  -- If clinic_id or patient_id changed, validate the new pair against access table.
  if not exists (
    select 1 from patient_clinic_access
    where patient_id = new.patient_id
      and clinic_id  = new.clinic_id
  ) then
    raise exception
      'Patient % has no access record at clinic %', new.patient_id, new.clinic_id;
  end if;

  return new;
end;
$$ language plpgsql;

create trigger invoices_validate_clinic_id
  before update on invoices
  for each row execute procedure validate_invoice_clinic_id();


-- ============================================================================
-- 14. TIMELINE_EVENTS (patient clinical timeline)
-- Source: TimelineEvent type (types/index.tsx L84-92)
-- v4 change: PATIENT-GLOBAL. clinic_id removed, owner_id added.
-- A timeline is the patient's full history across all branches of the chain.
--
-- DECISION: Kept as a table (not a view) because:
--   - Timeline can include manual clinical logs not tied to a visit or note
--   - The MVP writes a timeline event when a visit is completed (app-side)
--   - A future trigger or edge function can auto-populate it without schema change
--
-- v4 BUGFIX: Guard trigger added. Was missing in v3 despite every other child
-- table having one. clinic_id (now owner_id) was fully trusted from client.
-- ============================================================================

create table timeline_events (
  id uuid primary key default gen_random_uuid(),

  -- PATIENT-GLOBAL: owner_id (chain-level isolation), not clinic_id.
  -- Derived from patient.owner_id by guard trigger.
  owner_id   uuid not null references profiles(id),
  patient_id uuid not null references patients(id) on delete cascade,

  title       text not null,
  description text not null default '',
  timestamp   timestamptz not null,

  -- clinician_id FK replaces doctor_name / doctor_initials strings.
  -- Display name derived from profiles.display_name at read time.
  clinician_id uuid references profiles(id),

  category text not null default 'PHYSIO'
             check (category in ('PHYSIO','CONSULT','LAB','MEDICATION','SURGERY','NOTE')),

  -- Optional back-links to source records (nullable: manual logs may have none)
  visit_id uuid references visits(id),
  note_id  uuid references clinical_notes(id),

  created_at timestamptz not null default now()
);

-- v4 BUGFIX: this trigger was MISSING in v3. timeline_events is a patient-global
-- record, so it derives owner_id (not clinic_id) from the patient.
create trigger timeline_events_owner_id_guard
  before insert or update on timeline_events
  for each row execute procedure derive_owner_id_from_patient();


-- ============================================================================
-- 15. DAILY LEDGER VIEW
-- Source: LedgerEntry type (types/index.tsx L56-65)
-- DECISION: View, not a table. Derived from visits + patients.
--
-- MVP WORKFLOW: Every visit row = session complete + paid. Status = 'Complete'.
-- Added clinic_id to the view so the frontend can filter by location.
-- ============================================================================

create or replace view daily_ledger as
select
  v.id,
  v.clinic_id,
  v.created_at::time as time,
  p.full_name        as patient_name,
  v.complaint        as treatment,
  -- MVP: existence of the row = session done and paid. Always 'Complete'.
  'Complete'         as status,
  p.id               as patient_id,
  v.date,
  v.grand_total_in_paise
from visits v
join patients p on p.id = v.patient_id;

-- Usage:
-- SELECT * FROM daily_ledger
-- WHERE date = CURRENT_DATE AND clinic_id = '<clinic-uuid>'
-- ORDER BY time;


-- ============================================================================
-- 16. INDEXES
-- ============================================================================

-- pg_trgm for full-text name search in PatientSearch
-- Enable first: Supabase Dashboard → Database → Extensions → pg_trgm → Enable
create index idx_patients_full_name_trgm on patients
  using gin (full_name gin_trgm_ops);

-- All high-frequency filter patterns
create index idx_patients_owner_id     on patients(owner_id);
create index idx_patients_phone        on patients(phone);

create index idx_visits_clinic_id       on visits(clinic_id);
create index idx_visits_patient_id      on visits(patient_id);
create index idx_visits_date            on visits(date);
create index idx_visits_complaint_id    on visits(complaint_course_id);

create index idx_complaint_courses_clinic_id   on complaint_courses(clinic_id);
create index idx_complaint_courses_patient_id  on complaint_courses(patient_id);

create index idx_packages_clinic_id    on packages(clinic_id);
create index idx_packages_patient_id   on packages(patient_id);

create index idx_invoices_clinic_id    on invoices(clinic_id);
create index idx_invoices_patient_id   on invoices(patient_id);
create index idx_invoices_date         on invoices(date);

create index idx_visit_services_visit_id       on visit_services(visit_id);
create index idx_visit_services_clinic_id      on visit_services(clinic_id);
create index idx_clinic_service_prices_clinic  on clinic_service_prices(clinic_id);
create index idx_timeline_events_patient_id    on timeline_events(patient_id);
create index idx_timeline_events_owner_id      on timeline_events(owner_id);
create index idx_patient_alerts_patient_id     on patient_alerts(patient_id);
create index idx_patient_alerts_owner_id       on patient_alerts(owner_id);
create index idx_patient_vitals_patient_id     on patient_vitals(patient_id);
create index idx_patient_vitals_owner_id       on patient_vitals(owner_id);
create index idx_clinical_notes_patient_id     on clinical_notes(patient_id);
create index idx_clinical_notes_owner_id       on clinical_notes(owner_id);


-- ============================================================================
-- 17. ROW LEVEL SECURITY
-- DECISION: RLS enabled on all tables. V1 policies are permissive
-- ("authenticated users see all") — tightened when auth quest starts.
-- Location-specific tables are scoped by clinic_id.
-- Patient-global tables are scoped by owner_id.
-- patients is scoped by patient_clinic_access (staff) or owner_id (admin).
--   using (clinic_id = (select clinic_id from profiles where id = auth.uid()))
-- ============================================================================

alter table clinics                enable row level security;
alter table profiles               enable row level security;
alter table patients               enable row level security;
alter table patient_clinic_access  enable row level security;
alter table patient_alerts         enable row level security;
alter table patient_vitals         enable row level security;
alter table clinical_notes         enable row level security;
alter table complaint_catalog      enable row level security;
alter table complaint_courses      enable row level security;
alter table services               enable row level security;
alter table visits                 enable row level security;
alter table visit_services         enable row level security;
alter table packages               enable row level security;
alter table invoices               enable row level security;
alter table timeline_events        enable row level security;

-- V1 permissive policies — allow all authenticated
-- REPLACE these when RBAC is implemented:
create policy "v1_allow_all" on clinics               for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on profiles              for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on patients              for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on patient_clinic_access for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on patient_alerts        for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on patient_vitals        for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on clinical_notes        for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on complaint_catalog     for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on complaint_courses     for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on services              for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on visits                for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on visit_services        for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on packages              for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on invoices              for all to authenticated using (true) with check (true);
create policy "v1_allow_all" on timeline_events       for all to authenticated using (true) with check (true);

-- What the REAL two-policy isolation model will look like (for reference):
--
-- STAFF policy (receptionists and clinicians):
-- create policy "staff_patients" on patients
--   for all to authenticated
--   using (
--     id in (
--       select patient_id from patient_clinic_access
--       where clinic_id = (select clinic_id from profiles where id = auth.uid())
--     )
--   );
--
-- ADMIN policy:
-- DECISION (v5): Simplified. patients.owner_id directly references profiles(id)
-- where the profile is the admin. So the check is just: owner_id = auth.uid().
-- The v4 version was broken — admins have clinic_id = NULL by design, so
-- `where id = NULL` matched nothing and admins were locked out of every record.
-- create policy "admin_patients" on patients
--   for all to authenticated
--   using (
--     owner_id = auth.uid()
--     and (select role from profiles where id = auth.uid()) = 'admin'
--   );
--
-- ⚠ patient_clinic_access WRITE POLICY:
-- DECISION (v6): No client role should ever have direct INSERT/UPDATE/DELETE
-- on patient_clinic_access. Branch access is a security boundary — a receptionist
-- must not be able to grant themselves (or a patient) access to another branch
-- by hand-crafting a write.
-- The ONLY code that writes to patient_clinic_access is the
-- process_new_complaint_course() trigger, which runs as SECURITY DEFINER.
-- When tightening RLS, the real policy for patient_clinic_access should be:
--   SELECT: allowed (staff sees access for their clinic, admin sees all)
--   INSERT/UPDATE/DELETE: DENIED to all client roles — trigger only.
--
-- ⚠ KNOWN GAP — CROSS-BUSINESS ADMIN:
-- The admin policy above works perfectly for one chain. If a second unrelated
-- chain lands in the same DB, each admin only sees their own patients
-- (owner_id = auth.uid() scopes to their chain — correct by design).
-- The gap only appears if a single admin needs to manage MULTIPLE chains,
-- which isn't the current requirement.
alter table clinic_service_prices enable row level security;
create policy "v1_allow_all" on clinic_service_prices for all to authenticated using (true) with check (true);
```

---

## Table Summary

| # | Table | PK Type | Notes |
|---|---|---|---|
| 1 | `clinics` | uuid | One row per branch. All settings + `invoice_counter`. |
| 2 | `profiles` | uuid (auth) | `clinic_id` nullable (admins span all branches). |
| 3 | `patients` | uuid | **`clinic_id` removed. `owner_id` = which chain.** MRN unique per chain. |
| 3a | `patient_clinic_access` | uuid | **NEW (v4)** — junction: which branches a patient has attended. |
| 4 | `patient_alerts` | uuid | Patient-global. `owner_id` (chain isolation). |
| 5 | `patient_vitals` | uuid | Patient-global. `owner_id`. |
| 6 | `clinical_notes` | uuid | Patient-global. `owner_id`. |
| 7 | `complaint_catalog` | text *(seed)* | Unchanged. |
| 8 | `complaint_courses` | uuid | Location-specific. `clinic_id`. |
| 9 | `services` | text *(seed)* | Unchanged. |
| 9a | `clinic_service_prices` | uuid | Per-clinic price overrides. |
| 10 | `visits` | uuid | Location-specific. `clinic_id`. `grand_total` GENERATED. |
| 11 | `visit_services` | uuid | Location-specific. `clinic_id` from visit. |
| 12 | `packages` | uuid | Location-specific. `clinic_id`. |
| 13 | `invoices` | uuid | Location-specific. `clinic_id` + `invoice_number` from one merged trigger. |
| 14 | `timeline_events` | uuid | Patient-global. `owner_id`. **Guard trigger added (v4 bugfix).** |
| — | `daily_ledger` (VIEW) | — | Derived from visits + patients. |

**Total: 16 tables + 1 view**

---

## Resolved Decisions

| # | Decision | Resolution |
|---|---|---|
| 1 | Gender format | `'M' \| 'F' \| 'X'` with CHECK. |
| 2 | Money naming | `_in_paise` suffix everywhere. |
| 3 | MVP visit status | Single-touch. Row existing = complete + paid. |
| 4 | Multi-location isolation | `clinics` + `clinic_id` on location-specific tables, `owner_id` on patient-global tables. |
| 5 | `clinic_settings` | Merged into `clinics`. |
| 6 | `complaint_catalog` FK | `complaint_catalog_id` nullable on `complaint_courses`. |
| 7 | Conditional CHECKs | `referral_doctor_info` and `consultation_type` enforced at DB level. |
| 8 | `grand_total_in_paise` | GENERATED ALWAYS AS — cannot drift. |
| 9 | Text PKs | Transactional tables: UUID. Seed tables: text. |
| 10 | Invoice numbering | `invoice_number` + `invoice_counter` + single merged trigger (`process_new_invoice`). |
| 11 | `timeline_events` | `clinician_id` FK replaces string fields. `visit_id`/`note_id` back-links. |
| 12 | `clinic_id` guard triggers | `process_new_complaint_course` auto-provisions + validates. `validate_visit_clinic_id` + `validate_clinic_id_from_access` validate-only on visits/packages. Patient-global tables derive `owner_id`. |
| 13 | `visit_services.clinic_id` | Added. Derived from `visits.clinic_id` via `derive_clinic_id_from_visit()`. |
| 14 | Service pricing per-location | `clinic_service_prices` override table. App resolves via COALESCE. |
| 15 | Admin RLS policy | Simplified to `owner_id = auth.uid()`. V4 version was broken (NULL clinic_id on admins). |
| 16 | Patient cross-branch identity | `patients.clinic_id` removed. `owner_id` = chain. `patient_clinic_access` = per-branch. MRN unique per chain. |
| 17 | Invoice trigger ordering | Merged into `process_new_invoice()` SECURITY DEFINER — validates clinic_id + generates number. |
| 18 | `timeline_events` guard trigger | Was missing (v3 bug). Added. Derives `owner_id`. |
| 19 | No-op guard trigger regression | `derive_clinic_id_from_patient()` was gutted in v4. Fixed with proper validation functions per table. |
| 20 | Auto-provision `patient_clinic_access` | `process_new_complaint_course()` upserts access row on complaint_course insert — the true first clinical touchpoint. |
| 21 | Stale index on removed column | `idx_patients_clinic_id` → `idx_patients_owner_id`. Patient-global indexes updated to `owner_id`. |
| 22 | First-visit deadlock (v5 bug) | `complaint_courses` blocked on access that only `visits` could create; `visits` blocked on complaint_course. Fixed: auto-provision moved to `complaint_courses`. |
| 23 | `patient_clinic_access` write policy | No direct client writes ever. Only `process_new_complaint_course()` (SECURITY DEFINER) writes to it. Documented in RLS section. |
| 24 | Cross-chain tenancy in SECURITY DEFINER | `process_new_complaint_course` is SECURITY DEFINER (bypasses RLS). Added explicit `owner_id` match check — the trigger is its own enforcement boundary. |
| 25 | `owner_id` model limitation | Chain identity = single admin's user id. Schema doesn't constrain `owner_id` to admin profiles. Works today. Revisit if a second admin per chain is ever needed. |
| 26 | INSERT-only guard on `complaint_courses` | `complaint_courses_process_new` only fired on INSERT. Extended to `before insert or update` in v8. |
| 27 | `invoices` UPDATE tenancy gap | `process_new_invoice` is INSERT-only and must stay that way (extending it would re-mint invoice numbers on every UPDATE). Added separate `validate_invoice_clinic_id()` UPDATE-only trigger. Short-circuits when clinic_id/patient_id unchanged. |
| 28 | `complaint_courses` UPDATE optimization | `process_new_complaint_course` now short-circuits on UPDATE when clinic_id/patient_id unchanged. Prevents two SELECTs + upsert on every trivial update (e.g. bumping `total_sessions`). |
| 29 | `owner_id` not validated as admin | `patients.owner_id` and `clinics.owner_id` could point to any profile. A non-admin `owner_id` silently breaks the admin RLS policy with no error at write time. Added `validate_owner_is_admin()` trigger on both tables. |
| 30 | `clinics.owner_id` nullable | Was nullable only to work around DDL ordering. Changed to `not null` — every clinic must have an owner at creation. Deferred FK (`alter table add constraint`) still handles ordering. |
