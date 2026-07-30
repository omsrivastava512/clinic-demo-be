# Schema Cross-Reference: Evidence Trail

> **Purpose**: Before writing the SQL, this document maps every proposed table and column to the exact TypeScript source, mock data export, and architectural decision from the codebase that justifies it. No invented columns. No hallucinated features.

---

## Evidence Sources Used

| Source | Path | What It Tells Us |
|---|---|---|
| **TypeScript Types** | [types/index.tsx](file:///C:/Github/clinic-demo/src/types/index.tsx) | All entity shapes: `Patient`, `Visit`, `Service`, `ComplaintCourse`, `PackageRecord`, `InvoiceRecord`, `VitalSign`, `PatientAlert`, `TimelineEvent`, `LedgerEntry`, `MedicalComplaint`, `Procedure`, `InvoiceItem`, `VisitService` |
| **Feature Types** | [patient/types/index.tsx](file:///C:/Github/clinic-demo/src/features/patient/types/index.tsx) | `FormData` (intake form shape), `ClinicalNote` |
| **Mock Data** | [mock_data.tsx](file:///C:/Github/clinic-demo/src/data/mock_data.tsx) | 22 patients, 4 profiles, ~50 visits (V2), 16 complaint courses, 6 packages, 8 invoices, 13 services, 8 procedures (legacy), 22 ledger entries |
| **Complaint Catalog** | [complaints_catalog.ts](file:///C:/Github/clinic-demo/src/data/complaints_catalog.ts) | 38 complaint entries, 7 regions, catalog + region chip system |
| **Existing SQL Sketch** | [mock_data.tsx L853-877](file:///C:/Github/clinic-demo/src/data/mock_data.tsx#L853-L877) | A partial `patients` table SQL draft + `toCamel()` mapper |
| **Backend Migration Report** | [backend-migration-report.md](file:///C:/Github/clinic-demo/.agents/learnings/backend-migration-report.md) | Full API consumer map, endpoint catalogue, migration phases |
| **Brainstorming: Foundational Contract** | [clinic_more_issues.md](file:///C:/Github/clinic-demo/.agents/context/brainstorming_issues/clinic_more_issues.md) | AR1 (clinician_id), AR2 (audit trail), AR7 (paise), RBAC, multi-therapist toggle, i18n, feature flags |
| **Brainstorming: Verdict** | [verdict.md](file:///C:/Github/clinic-demo/.agents/context/brainstorming_issues/verdict.md) | P0 list: money as paise, clinician_id, completion landings, EOD reconciliation |
| **Brainstorming: Clinic Context** | [comment_1.md](file:///C:/Github/clinic-demo/.agents/context/brainstorming_issues/comment_1.md) | `useClinic()`, `usePermissions()`, `clinicConfig` JSONB shape |
| **Issue #32 body** | [open_issues_utf8.json](file:///C:/Github/clinic-demo/.agents/context/open_issues_utf8.json) | Detailed complaint schema design with search, aliases, layman_terms, region, onset_type, pg_trgm |

---

## Table-by-Table Evidence

### 1. `patients`

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (uuid PK) | `Patient.id` (L12) | `"uuid-p01"` etc. | Already UUIDs in mock |
| `mrn` (text unique) | `Patient.mrn` (L13) | `"MED-001"` | Already in SQL sketch (L855) |
| `full_name` (text) | `Patient.fullName` (L14) | `"Priya Kapoor"` | Already in SQL sketch (L856) |
| `phone` (text) | `Patient.phone` (L17) | `"9000000001"` | Already in SQL sketch (L857) |
| `address` (text) | `Patient.address` (L18) | `"Flat 12A, Lokhandwala, Mumbai"` | Already in SQL sketch (L858) |
| `date_of_birth` (date) | `Patient.dateOfBirth` (L15) | `"1988-03-12"` | Already in SQL sketch (L859) |
| `gender` (text) | `Patient.gender` (L19) | `"female"` / `"male"` | SQL sketch says "enum" (L860). FormData uses `'M' | 'F' | 'X'`. **DISCREPANCY**: mock uses `"female"`/`"male"`, FormData uses `"M"`/`"F"`/`"X"`. Using text for now, NOT an enum, to avoid breaking either. |
| `last_visit_at` (timestamptz) | `Patient.lastVisitAt` (L16) | `"2024-02-11T09:45:00Z"` | Already in SQL sketch (L861) |
| `is_active` (boolean) | `Patient.isActive` (L21) | `true` in all mocks | Already in SQL sketch (L863) |
| `referral_mode` (text) | `Patient.referralMode` (L25-26) | `"WALKIN"`, `"GOOGLE"`, `"DOCTOR"` | SQL sketch says "enum" (L864). Using text with CHECK constraint. |
| `referral_doctor_info` (text, nullable) | `Patient.referralDoctorInfo` (L26) | `"Dr. Rao – Orthopedics"` | Already in SQL sketch (L865). Nullable because only set when referral_mode = 'DOCTOR' |
| `blood_type` (text, nullable) | `PatientProfile.bloodType` (L96) | `"B+"`, `"O+"`, `"A-"` | Not in SQL sketch. Comes from PatientProfile extension type. |
| `insurer_name` (text, nullable) | `PatientProfile.insurerName` (L97) | `"Star Health Insurance"` | Not in SQL sketch. Comes from PatientProfile extension type. |
| `photo_url` (text, nullable) | `PatientProfile.photoUrl` (L101) | Optional, unused in mock | Declared in TS type but not populated in mock. Including because the type explicitly defines it. |
| `clinician_id` (uuid FK, nullable) | **NOT in TS types** | **NOT in mock data** | **From AR1 in verdict.md and clinic_more_issues.md**: "every clinical entity gets `clinician_id` from day one." Default to single seeded clinician. |
| `created_by` (uuid, nullable) | **NOT in TS types** | **NOT in mock data** | **From AR2 in clinic_more_issues.md**: "every write records `actor_user_id`" |
| `created_at` (timestamptz) | `Patient.createdAt` (L22) | `"2024-06-01T10:00:00Z"` | Already in SQL sketch (L867) |
| `updated_at` (timestamptz) | `Patient.updatedAt` (L23) | `"2024-06-01T10:00:00Z"` | Already in SQL sketch (L868) |

**NOT including** `notes` as a column — `Patient.notes` is `ClinicalNote` (L20), which is an array of `{category, observation, isCritical}`. That's a separate table, not a text column. The SQL sketch's `notes text` (L862) is wrong for the actual type shape.

---

### 2. `patient_alerts`

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (uuid PK) | Generated | N/A | No ID in TS type — but Supabase needs a PK |
| `patient_id` (uuid FK) | Implicit — `PatientProfile.alerts` is an array on the patient | Lives inside `MOCK_PATIENT_PROFILES[n].alerts` | |
| `type` (text) | `PatientAlert.type` (L72) | `'ALLERGY'`, `'FALL_RISK'` | Enum: `'ALLERGY' | 'FALL_RISK' | 'DNR' | 'OTHER'` |
| `label` (text) | `PatientAlert.label` (L73) | `"Penicillin Allergy"`, `"Post-Op Fall Risk"` | |
| `created_at` (timestamptz) | Not in TS type | N/A | Standard audit column |

---

### 3. `patient_vitals`

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (uuid PK) | Generated | N/A | |
| `patient_id` (uuid FK) | Implicit — `PatientProfile.vitals` is an array on the patient | Lives inside `MOCK_PATIENT_PROFILES[n].vitals` | |
| `type` (text) | `VitalSign.type` (L77) | `'BP'`, `'HR'`, `'TEMP'`, `'SPO2'` | |
| `value` (text) | `VitalSign.value` (L78) | `"118/76"`, `"78"`, `"98.4"`, `"98"` | Kept as text because BP is `"118/76"` — not a number |
| `unit` (text) | `VitalSign.unit` (L79) | `"mmHg"`, `"bpm"`, `"°F"`, `"%"` | |
| `recorded_at` (timestamptz) | `VitalSign.recordedAt` (L80) | `"2024-02-11T09:30:00Z"` | |
| `trend` (text) | `VitalSign.trend` (L81) | `'NORMAL'`, `'HIGH'`, `'LOW'` | |

---

### 4. `clinical_notes`

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (uuid PK) | Generated | N/A | |
| `patient_id` (uuid FK) | Implicit — `Patient.notes` is `ClinicalNote` on the patient type | `MOCK_PATIENT_PROFILES` don't have notes populated, but the FormData.clinicalNotes is `ClinicalNote[]` | |
| `category` (text) | `ClinicalNote.category` (L19 in patient/types) | N/A — not in mock | e.g. fall risk category, surgical history category |
| `observation` (text) | `ClinicalNote.observation` (L20 in patient/types) | N/A — not in mock | |
| `is_critical` (boolean) | `ClinicalNote.isCritical` (L21 in patient/types) | N/A — not in mock | |
| `created_at` (timestamptz) | Standard | N/A | |

---

### 5. `complaint_catalog` (the seed/reference table)

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (text PK) | `MedicalComplaint.id` (L31) | `"CAT_S01"`, `"CAT_SH01"` etc. | Using text PK because the IDs are already semantic strings |
| `title` (text) | `MedicalComplaint.title` (L32) | `"Cervical Spondylosis"` | |
| `region` (text) | `MedicalComplaint.region` (L38) | `"Spine"`, `"Shoulder"`, `"Knee"`, `"Hip"`, `"Elbow"`, `"Ankle"`, `"Neuro"` | From `CATALOG_REGIONS` constant |
| `is_active` (boolean) | `MedicalComplaint.isActive` (L34) | `true` for all | |
| `created_at` (timestamptz) | Standard | N/A | |

**NOT including** `doctor`, `type` ('EXISTING'|'NEW') — those are runtime/UI concerns set when a complaint is attached to a patient, not properties of the catalog entry.

**NOT including** the Issue #32 extended schema (aliases, layman_terms, body_parts, onset_type, search_text tsvector) — that design was discussed in the issue body but **never implemented** in any type or mock. It's a good future schema but I will NOT pretend it exists in the codebase. It can be added as a migration when the search feature is built.

---

### 6. `complaint_courses` (patient-specific treatment arcs)

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (text PK) | `ComplaintCourse.id` (L162) | `"CC-p01-01"`, `"CC-p02-01"` etc. | Semantic text IDs in mock |
| `patient_id` (uuid FK) | `ComplaintCourse.patientId` (L163) | `"uuid-p01"` | |
| `complaint_name` (text) | `ComplaintCourse.complaintName` (L164) | `"Ankle Sprain"`, `"Frozen Shoulder"` | |
| `start_date` (date) | `ComplaintCourse.startDate` (L165) | `"2024-01-15"` | |
| `last_date` (date) | `ComplaintCourse.lastDate` (L166) | `"2024-02-11"` | |
| `total_sessions` (integer) | `ComplaintCourse.totalSessions` (L167) | `10`, `9`, `12` etc. | |
| `status` (text) | `ComplaintCourse.status` (L168) | `'Active'`, `'Completed'` | |
| `clinician_id` (uuid FK, nullable) | **NOT in TS type** | **NOT in mock** | **AR1**: clinical entity gets clinician_id |
| `created_at` (timestamptz) | Standard | N/A | |
| `updated_at` (timestamptz) | Standard | N/A | |

---

### 7. `services` (the service/procedure catalogue)

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (text PK) | `Service.id` (L110) | `"SVC-01"` through `"SVC-13"` | |
| `name` (text) | `Service.name` (L111) | `"SWD"`, `"IFT"`, `"Cupping"` etc. | |
| `standalone_price` (integer) | `Service.standalonePrice` (L112) | `80`, `70`, `500`, `600`, `700` | **AR7**: Store as integer paise. Mock values are in rupees. In DB: `80` → `8000` paise. |
| `category` (text) | `Service.category` (L113) | `'STANDARD'`, `'PREMIUM'` | |
| `created_at` (timestamptz) | Standard | N/A | |

**About legacy `Procedure` type**: The `Procedure` interface (L41-46) with `id`, `name`, `code`, `cost` is used by `PHYSIO_PROCEDURES` (L383-392) and `ComplaintSection.tsx`. The backend-migration-report says `GET /procedures` replaces this. However, the V2 billing model uses `Service`, not `Procedure`. The migration report confirms: "Only migrate `MOCK_VISITS_V2` — the legacy model can be dropped." **Decision: Use `services` table (V2 model). The legacy `Procedure` type and `PHYSIO_PROCEDURES` mock will be deprecated on the frontend.**

---

### 8. `visits` (one per complaint per session)

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (text PK) | `Visit.id` (L131) | `"VP01-01"`, `"VP02-07"` etc. | |
| `patient_id` (uuid FK) | `Visit.patientId` (L132) | `"uuid-p01"` | |
| `complaint_course_id` (text FK) | `Visit.complaintId` (L135) | `"CC-p01-01"` | TS comment says "FK → ComplaintCourse.id" |
| `date` (date) | `Visit.date` (L133) | `"2024-01-15"` | |
| `complaint` (text) | `Visit.complaint` (L134) | `"Ankle Sprain"` | TS comment: "display name". Denormalized for display. |
| `visit_type` (text) | `Visit.visitType` (L136) | `'CONSULTATION'`, `'MACHINE_ONLY'` | |
| `consultation_type` (text, nullable) | `Visit.consultationType` (L138) | `'FIRST'`, `'SUBSEQUENT'`, `undefined` | Only for CONSULTATION visits |
| `consultation_fee` (integer) | `Visit.consultationFee` (L141) | `300`, `200`, `0` | **AR7**: paise. `300` → `30000`. TS comment: "300 (FIRST) | 200 (SUBSEQUENT) | 0 (MACHINE_ONLY)" |
| `services_total` (integer) | `Visit.servicesTotal` (L143) | Computed in mock helper fn `_mv()` | **AR7**: paise. TS comment: "sum of charged VisitServices" |
| `grand_total` (integer) | `Visit.grandTotal` (L144) | Computed in mock helper fn `_mv()` | **AR7**: paise. TS comment: "consultationFee + servicesTotal" |
| `clinician_id` (uuid FK, nullable) | **NOT in TS type** | **NOT in mock** | **AR1**: clinical entity |
| `created_by` (uuid, nullable) | **NOT in TS type** | **NOT in mock** | **AR2**: audit trail |
| `created_at` (timestamptz) | Standard | N/A | |

**TS DEBT comments on this type**: Lines 139-140 say "DEBT: These billing fields should be derived at read time, not stored. Use calculateConsultationFee() from patientUtils instead." and Lines 123-124 on VisitService say similar. **For the DB schema, I'm storing these as columns anyway** because the mock data stores them, the profile page reads them, and computing them on every read adds unnecessary DB load. The DEBT comments are about the TS mock data helpers, not about the DB design.

---

### 9. `visit_services` (junction table: which services used in a visit)

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (text PK) | `VisitService.id` (L118) | `"VP01-01-s1"`, generated by `_vs()` helper | |
| `visit_id` (text FK) | `VisitService.visitId` (L119) | Set by `_vs()` helper | |
| `service_id` (text FK) | `VisitService.serviceId` (L120) | Mapped from `Service.id` | |
| `service_name` (text) | `VisitService.serviceName` (L121) | Denormalized. TS comment: "denormalised for display" | |
| `service_category` (text) | `VisitService.serviceCategory` (L122) | `'STANDARD'`, `'PREMIUM'` | |
| `is_charged` (boolean) | `VisitService.isCharged` (L125) | Derived in `_vs()`: `vt === 'MACHINE_ONLY' || svc.category === 'PREMIUM'` | |
| `charged_amount` (integer) | `VisitService.chargedAmount` (L126) | `0` if not charged, else `standalonePrice` | **AR7**: paise |

---

### 10. `packages` (time-based therapy packages)

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (text PK) | `PackageRecord.id` (L178) | `"PKG-001"` through `"PKG-006"` | |
| `patient_id` (uuid FK) | `PackageRecord.patientId` (L179) | `"uuid-p01"` | |
| `linked_complaint_id` (text FK) | `PackageRecord.linkedComplaintId` (L180) | `"CC-p01-01"` | FK → complaint_courses.id |
| `linked_complaint_name` (text) | `PackageRecord.linkedComplaintName` (L181) | `"Chronic Back Pain"` | Denormalized |
| `package_name` (text) | `PackageRecord.packageName` (L182) | `"Chronic Back Pain — 20 Day Package"` | |
| `purchase_date` (date) | `PackageRecord.purchaseDate` (L183) | `"2026-07-01"` | |
| `duration_days` (integer) | `PackageRecord.durationDays` (L184) | `20`, `10`, `30` etc. | |
| `expiry_date` (date) | `PackageRecord.expiryDate` (L185) | `"2026-07-21"` | |
| `exclude_sundays` (boolean) | `PackageRecord.excludeSundays` (L186) | `true`, `false` | |
| `attended_days` (integer) | `PackageRecord.attendedDays` (L187) | `12`, `10`, `7` etc. | Denormalized aggregate |
| `missed_days` (integer) | `PackageRecord.missedDays` (L188) | `2`, `0`, `1` etc. | Denormalized aggregate |
| `amount_paid` (integer) | `PackageRecord.amountPaid` (L189) | `8000`, `4500`, `12000` etc. | **AR7**: paise. `8000` → `800000` |
| `status` (text) | `PackageRecord.status` (L190) | `'Active'`, `'Completed'`, `'Expired'` | |
| `day_log` (jsonb) | `PackageRecord.dayLog` (L192) | `['attended','missed','upcoming',...]` | TS comment L171-173: "Max 30 elements, stores per-day attendance status" |
| `created_at` (timestamptz) | Standard | N/A | |
| `updated_at` (timestamptz) | Standard | N/A | |

---

### 11. `invoices`

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (text PK) | `InvoiceRecord.id` (L196) | `"INV-001"` through `"INV-008"` | |
| `patient_id` (uuid FK) | `InvoiceRecord.patientId` (L197) | `"uuid-p01"` | |
| `amount` (integer) | `InvoiceRecord.amount` (L198) | `2500`, `1800`, `500` etc. | **AR7**: paise |
| `date` (date) | `InvoiceRecord.date` (L199) | `"2024-02-11"` | |
| `payment_status` (text) | `InvoiceRecord.paymentStatus` (L200) | `'Paid'`, `'Pending'`, `'Overdue'` | |
| `payment_mode` (text, nullable) | From InvoiceBuilder UI — `paymentMode` state | Migration report L204: "`paymentMode` state is already captured" | |
| `visit_id` (text FK, nullable) | **NOT in TS type** | **NOT in mock** | Logical: an invoice is generated from a visit. Migration report L305: "POST /visits + POST /invoices". Adding FK to link them. |
| `created_by` (uuid, nullable) | **NOT in TS type** | **NOT in mock** | **AR2**: audit trail |
| `created_at` (timestamptz) | Standard | N/A | |

---

### 12. `timeline_events` (patient clinical timeline)

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (text PK) | `TimelineEvent.id` (L85) | `"TL-p01-01"`, `"TL-p02-03"` etc. | |
| `patient_id` (uuid FK) | Implicit — `PatientProfile.timeline` is array on patient | Lives inside `MOCK_PATIENT_PROFILES[n].timeline` | |
| `title` (text) | `TimelineEvent.title` (L86) | `"Ankle Sprain"`, `"Frozen Shoulder (Adhesive Capsulitis)"` | |
| `description` (text) | `TimelineEvent.description` (L87) | `"Grade II lateral ankle sprain..."` | |
| `timestamp` (timestamptz) | `TimelineEvent.timestamp` (L88) | `"2024-01-15T09:00:00Z"` | |
| `doctor_name` (text) | `TimelineEvent.doctorName` (L89) | `"Dr. R. Sharma"` | |
| `doctor_initials` (text) | `TimelineEvent.doctorInitials` (L90) | `"RS"`, `"AG"` | |
| `category` (text) | `TimelineEvent.category` (L91) | `'PHYSIO'` | Enum: `'PHYSIO' | 'CONSULT' | 'LAB' | 'MEDICATION' | 'SURGERY' | 'NOTE'`. Only `'PHYSIO'` used in mock. |

---

### 13. `clinic_settings` (single-row config)

| Column | TS Source | Mock Data Source | Notes |
|---|---|---|---|
| `id` (uuid PK) | N/A | N/A | Single row |
| `clinic_name` (text) | From Issue #70 body (open_issues_utf8.json): "Clinic name" | N/A | |
| `clinic_address` (text) | From Issue #70 body | N/A | |
| `clinic_phone` (text) | From Issue #70 body | N/A | |
| `default_currency` (text) | From comment_1.md clinicConfig: `"defaultCurrency": "INR"` | N/A | |
| `default_timezone` (text) | From comment_1.md clinicConfig: `"defaultTimezone": "Asia/Kolkata"` | N/A | |
| `consultation_fee_first` (integer) | From Visit type L141: "300 (FIRST)" | `300` in `_mv()` helper | **AR7**: paise → `30000` |
| `consultation_fee_subsequent` (integer) | From Visit type L141: "200 (SUBSEQUENT)" | `200` in `_mv()` helper | **AR7**: paise → `20000` |
| `exclude_sundays_default` (boolean) | From PackageRecord.excludeSundays (L186) | Varies per package | Default for new packages |
| `payment_methods_accepted` (text[]) | From Issue #70 body: "Default payment methods accepted (Cash, UPI, Card)" | N/A | |
| `multi_therapist` (boolean) | From comment_1.md: `"multiTherapist": false` | N/A | Feature flag |
| `color_scheme` (text) | From user request: "color blindness" use case | N/A | `'default'`, `'high-contrast'`, `'deuteranopia'` |
| `config` (jsonb) | From comment_1.md clinicConfig shape | N/A | Additional feature flags: inventoryEnabled, appointmentsEnabled, etc. |
| `updated_at` (timestamptz) | Standard | N/A | |

---

## What I Am NOT Including (and Why)

| Omitted | Reason |
|---|---|
| `ledger_entries` table | `LedgerEntry` is a **view**, not a table. The migration report says `GET /ledger/entries?date=today` — this is a JOIN query over `visits` + `patients` filtered by today's date. No separate table needed. |
| `users` / `profiles` table | Supabase Auth handles this automatically via `auth.users`. We only need a `profiles` table to store `role` and `display_name` — Supabase has a standard pattern for this. |
| Extended complaint catalog (aliases, layman_terms, body_parts, onset_type, search_text) | From Issue #32 body — discussed but **never implemented** in any type, mock, or component. Add as a future migration. |
| `products` / `inventory` / `purchase_items` | **Explicitly excluded** in clinic_more_issues.md §2: "do not introduce any inventory structures in v1" |
| `appointments` / `scheduling` tables | Covered by Epic #15 — future feature, no current types or mocks exist for it |
| i18n infrastructure | From clinic_more_issues.md §4 — this is a frontend concern (react-i18next), not a DB schema concern |
| `InvoiceItem` as a table | `InvoiceItem` (L48-54) is a transient TS type used in the ProcedureLogger → InvoiceBuilder flow. The actual stored data is in `visit_services` (what was used) + `invoices` (the total). Invoice line items are reconstructible from `visit_services` via `visit_id`. No separate `invoice_items` table needed. |
| `MedicalComplaint` with `type: 'EXISTING' | 'NEW'` and `doctor` fields | These are **runtime/UI state** — when a complaint is attached to a patient in the ComplaintSelector, the UI tracks whether it's an existing or new complaint and which doctor ordered it. This state lives in `complaint_courses`, not in the catalog. |
