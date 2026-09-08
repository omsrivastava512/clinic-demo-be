# OPD Workflow Screens — Comprehensive Audit Report

**Scope:** The 5 podium showcase screens in [`vistitworkflow.tsx`](file:///c:/Github/clinic-demo/src/pages/vistitworkflow.tsx), evaluated against clinical workflow intent, Supabase v11 schema contracts, and [`frontend-design-principles.md`](file:///c:/Github/clinic-demo-be/.agents/workflows/frontend-design-principles.md).

**Methodology:** 6 parallel research agents audited all component source files, backend specifications ([`FRONTEND_WORKFLOW.md`](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md), [`supabase_migration.md`](file:///c:/Github/clinic-demo-be/docs/supabase_migration.md), [`schema_cross_reference.md`](file:///c:/Github/clinic-demo-be/docs/schema_cross_reference.md)), business logic decision logs ([`specialty-decision-log.md`](file:///c:/Github/clinic-demo-be/study/business_logic/speciality_and_invoice/specialty-decision-log.md), [`DECISIONS.md`](file:///c:/Github/clinic-demo-be/study/module_1/DECISIONS.md), [`GAPS.md`](file:///c:/Github/clinic-demo-be/study/module_1/GAPS.md), [`LOGS.md`](file:///c:/Github/clinic-demo-be/study/module_2/LOGS.md)), and the [`frontend-backend-wiriing-proposal.md`](file:///c:/Github/clinic-demo-be/plans/frontend-backend-wiriing-proposal.md).

**Baseline Note:** All 5 screens are currently isolated, non-interconnected presentation components running on 100% mock data with dummy `alert()` handlers. No Supabase integration exists.

> **[COMPACTED PROMPT]**
> <details>
> <summary><b>Click to view the Compacted Audit Prompt</b></summary>
> Deploy a multi-agent architectural and design audit across the 5 isolated podium showcase screens in `src/pages/vistitworkflow.tsx` (`DailyLedger`, `NewPatientIntake`, `ComplaintSelector`, `ProcedureLogger`, `InvoiceBuilder`), cross-referencing backend specifications (`FRONTEND_WORKFLOW.md`, `supabase_migration.md`, `schema_cross_reference.md`, `specialty-decision-log.md`, `module_1/decisions`, `module_1/gaps`) and evaluating them against `frontend-design-principles.md`. First, regarding **Inter-Screen & Step-by-Step Workflow Readiness**—trace the end-to-end clinical reception journey (`Daily Ledger` search/intake → `Complaint Selector` → `Procedure Logger` → `Invoice & Payment`), identifying exact prop contracts, missing callbacks, shared state machines, and data-pipeline gaps required to transition from isolated presentation mocks into a connected multi-step flow. Second, regarding **Backend & Data Contract Alignment**—audit each component against Supabase v11 schemas, triggers, and business invariants (pricing calculations, consultation models, RLS visibility windows, MRN generation, Pay Later lifecycle, and multi-visit foreign keys), isolating critical backend blockers and contract drift. Third, regarding **UX, Visual Hierarchy & Accessibility Auditing**—evaluate individual screens and global composition against design principles (typography scale, spatial grid alignment, WCAG contrast compliance, token consistency, keyboard accessibility/ARIA patterns, and data persistence/failure recovery), delivering a prioritized matrix of technical blockers, design violations, and critical bug fixes.
>
> </details>

---

## Summary Comparison Matrix

| Screen | Clinical Alignment | Backend Blockers | Design Principle Violations | Critical Bugs | Key Strengths |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **01 DailyLedger** | 🟡 Partially aligned; status badges have no DB home | 🔴 3 blockers (status lifecycle, cross-branch RLS, MRN search broken) | 🔴 Heavy: monospace abuse, ALL-CAPS labels, missing ARIA combobox | 🔴 In-render `.sort()` mutates array; broken `data-selected` CSS; missing React keys | Strong keyboard nav, dark mode, responsive mobile |
| **02 NewPatientIntake** | 🟡 Functional form; clinical scope blurring (notes at desk) | 🔴 3 blockers (age vs DOB, MRN generation, notes write sequencing) | 🔴 Heavy: ALL-CAPS labels, monospace addresses, `alert()` validation | 🔴 Empty age bypasses validation; cursor jumping on capitalize; `tabIndex={-1}` locks out notes | Strong type-safe reducer, discriminated union referral types |
| **03 ComplaintSelector** | 🔴 Free-text entry is non-functional | 🔴 3 blockers (catalog ID dropped, write-timing trigger trap, multi-visit FK) | 🟡 Moderate: middot meta, arrow buttons, cramped popover | 🔴 Event bubbling on delete; hardcoded MRN; render-phase state setting | Excellent ARIA combobox, keyboard navigation, portal popover |
| **04 ProcedureLogger** | 🔴 Consultation fee model inverted (per-complaint vs once-per-session) | 🔴 3 blockers (consultation model, legacy catalog IDs, client UUID reconciliation) | 🟡 Moderate: staggered entrance animations, ALL-CAPS, inert search | 🟡 Ineffective `useCallback`; hardcoded patient name; typos in filenames | Cleanest hook extraction, composite key architecture, derived billing state |
| **05 InvoiceBuilder** | 🔴 No Pay Later, no receipt, payment mode dropped | 🔴 3 blockers (pre-minted invoice number, payment mode not threaded, multi-visit FK) | 🟡 Moderate: ALL-CAPS labels, isolated Hindi text, monospace prices | 🔴 Invalid `<div>` inside `<table>`; conflicting disabled hover CSS | Clean decomposition, Indian locale UPI default, stable composite keys |

---

## Integration Readiness Assessment

### The Workflow as Designed

```
Ledger Search ─┬─ (existing patient) ──► Complaint Selector ──► Procedure Logger ──► Invoice & Payment
               └─ (new patient) ────► Patient Intake ──┘
```

The receptionist opens the **Daily Ledger** — a live OPD register. They search for a patient by name or phone. If found, they select them and jump to the **Complaint Selector**. If not found, they register via **New Patient Intake**, which returns them to the Complaint Selector with the new patient attached. The doctor checks off today's complaints, then in the **Procedure Logger** toggles which modalities were administered per complaint while a live receipt updates. Finally, the **Invoice Builder** shows the itemized total with payment mode selection and confirmation.

---

### Inter-Screen Integration Readiness: ~15%

The 5 screens are **completely isolated**. They share no state, no callbacks, and no awareness of each other.

**What exists (the 15%):**
- Each component is self-contained and renders correctly with its own mock data
- Each component has well-structured internal state management (hooks, reducers, derived state)
- The conceptual data pipeline is sound — complaints echo into procedure headers, procedures echo into invoice line items

**What's missing:**

| Connection | Current State | What's Needed |
| :--- | :--- | :--- |
| **Ledger → Intake or Complaints** | `alert("Selected Existing")` / `alert("Add New")` | `DailyLedger` accepts **zero props**. Needs `onSelectPatient(patient)` and `onRegisterNew(query)` callbacks. |
| **Intake → Complaints** | `onSubmit` receives `FormData`, parent does nothing | Must return created `Patient` object with real `id`, then pass it into `ComplaintSelector`. |
| **Complaints → Procedures** | `onConfirm` emits `string[]` (just selected IDs) | Must emit full `MedicalComplaint[]` objects — `ProcedureLogger` needs `title`, `id`, and complaint metadata, not bare strings. |
| **Procedures → Invoice** | `onComplete` returns `InvoiceItem[]`, parent calls `alert()` | Must pipe the items array directly into `InvoiceBuilder` as props. |
| **Invoice → Ledger** | `onClose` calls `alert("Visit Closed")` | Must persist payment, then refresh/return to the Daily Ledger. |
| **Stepper / Orchestrator** | Does not exist | A parent state machine tracking which step is active and what data has been collected across steps. Currently [`vistitworkflow.tsx`](file:///c:/Github/clinic-demo/src/pages/vistitworkflow.tsx) renders all 5 simultaneously as a vertical catalog. |

**Effort assessment:** Medium. The internal component logic is solid — clean hooks, good state isolation, clear (where they exist) prop interfaces. The work is building the orchestrator and adjusting ~5 prop contracts. No major internal rewrites needed for this alone.

---

### Backend Integration Readiness: ~25% structurally ready, ~0% wired

No Supabase calls exist anywhere. But the real concern isn't "just add fetch calls" — several components have **structural assumptions that conflict with the database schema**, requiring architectural changes before a single query can fire.

#### Tier 1 — Could wire with minimal changes (swap mock data for queries)

| Component | What's close | Remaining adapter work |
| :--- | :--- | :--- |
| **Daily Ledger (reads)** | Search UI and table rendering work. Just swap `MOCK_PATIENTS.filter()` with a Supabase RPC call. | Add debounce to search. Fix [`cleanSearchInput()`](file:///c:/Github/clinic-demo/src/lib/inputValidation.tsx) breaking MRN queries (strips hyphens/digits). Swap `MOCK_LEDGER_ENTRIES` with `daily_ledger` view query. |
| **Complaint Selector (reads)** | Active complaint checklist and catalog search render correctly. Region chips match `complaint_catalog.region` exactly. | Swap mock arrays with Supabase queries on `complaint_courses` and `complaint_catalog`. |

#### Tier 2 — Needs data model changes before any query can work

| Component | What conflicts | Schema reality | Severity |
| :--- | :--- | :--- | :--- |
| **New Patient Intake** | Captures `age` as a 2-digit number string | DB column is `patients.date_of_birth date`. The form literally cannot produce a valid INSERT payload. | 🔴 Blocker |
| **New Patient Intake** | No MRN field or generator anywhere | `patients.mrn` is `NOT NULL`. Insert throws constraint violation. | 🔴 Blocker |
| **New Patient Intake** | Clinical notes bundled in flat `FormData` | `clinical_notes.patient_id NOT NULL FK`. Must INSERT patients first, get back `id`, then INSERT notes as a separate operation. | 🔴 Blocker |
| **Complaint Selector** | Drops `complaint_catalog_id` on selection | Only `title` string is preserved via `add(item.title)`. DB wants the FK for categorization and trigger logic. | 🔴 Blocker |
| **Complaint Selector** | No DB write happens at selection time | `complaint_courses` INSERT must fire **before** `visits` INSERT (trigger chain: `process_new_complaint_course()` provisions `patient_clinic_access`, which `validate_visit_clinic_id()` then checks). If deferred to local-only state, Step 3/4 writes will throw a raw Postgres trigger exception. | 🔴 Blocker |

#### Tier 3 — Fundamental architecture conflicts (the hardest fixes)

| Component | What's inverted | Schema reality | Severity |
| :--- | :--- | :--- | :--- |
| **Procedure Logger** | "Consultation" is a ₹500 per-complaint checkbox | `visits.consultation_fee_in_paise` is a dedicated column charged **once per session** (₹350). Consultation is not a `visit_services` row. This is the **exact opposite** of how the component models it. | 🔴 Architectural |
| **Procedure Logger** | Uses legacy `PHYSIO_PROCEDURES` catalog (`PROC_01`–`PROC_08`) | Real table is `services` (`SVC-01`–`SVC-13`). Foreign keys will fail on insert. | 🔴 Blocker |
| **Procedure Logger** | Bills every procedure unconditionally | `visit_services.is_charged` determines whether a standard modality is actually billed (`false` = bundled into flat therapy fee) or itemized separately (only for `MACHINE_ONLY` visits or `PREMIUM` carve-outs). `charged_amount_in_paise` must also be set. The component ignores both columns. | 🔴 Blocker |
| **Procedure Logger** | No clinic price override awareness | `clinic_service_prices` table allows per-clinic price customization. Component uses hardcoded procedure costs. | 🟡 Missing feature |
| **Invoice Builder** | Expects pre-minted `invoiceNumber` as a prop | DB trigger `process_new_invoice()` mints the number atomically on INSERT. Displaying it pre-submit burns the counter and the format doesn't even match (`INV-2026-0306-088` vs trigger's `INV-YYYY-XXXX`). | 🔴 Architectural |
| **Invoice Builder** | `paymentMode` state exists but is never emitted | "Confirm Payment" calls raw `onClose`. Selected payment method is silently discarded. Backend requires `confirm_invoice_payment(invoice_id, payment_mode)`. | 🔴 Blocker |
| **Invoice Builder** | Items span multiple complaints | `invoices.visit_id` is a singular FK. Multi-complaint checkout needs the `invoice_line_items` junction table (specified in specialty-decision-log but not yet built). | 🟡 Schema gap |
| **Daily Ledger** | Displays `Waiting` / `In Therapy` / `Paid` status badges | Schema has **no encounter lifecycle**. `daily_ledger` view hardcodes `'Complete' as status`. In-progress patients have no row anywhere in the transactional schema (Decision 3 unresolved). | 🟡 Open Decision |

---

### Systemic Backend Invariants (Must Be Respected by All Screens)

These are non-negotiable database-level rules that will cause hard failures if violated:

1. **Write sequencing is trigger-enforced:**
   ```
   patients INSERT → complaint_courses INSERT (provisions patient_clinic_access)
                    → visits INSERT (validates access exists via trigger)
                    → invoices INSERT (mints number via trigger)
   ```
   Any attempt to skip or reorder this chain causes a Postgres exception. The frontend orchestrator must respect this exact sequence.

2. **Currency is always integer paise:** All `*_in_paise` columns store ₹1 = 100. No floats, no rupee strings. The frontend must convert display values (₹350) to storage values (35000) before any write.

3. **Invoice numbers are server-minted:** The client must **never** pre-generate invoice numbers. The DB trigger atomically increments `clinics.invoice_counter` and returns the minted number. InvoiceBuilder should display "Auto-generated on confirmation" until the POST response arrives.

4. **Consultation fee is once per patient lifetime (₹350):** Not per complaint, not per session. Maps to `visits.consultation_fee_in_paise`, not a `visit_services` row. This is confirmed in Decision 6 and the specialty-decision-log.

5. **Multi-complaint sessions produce multiple `visits` rows:** Each selected complaint creates a separate `visits` record. The invoice must link to all of them via `invoice_line_items`, not a singular `invoices.visit_id` FK.

---

### Recommended Integration Sequence

```
Phase 1: Inter-Screen Integration (mock data, no backend)
  └─ Build stepper/orchestrator state machine
  └─ Wire prop contracts between screens
  └─ Fix onConfirm output types (string[] → MedicalComplaint[])
  └─ Pipe ProcedureLogger output into InvoiceBuilder items

Phase 2: Fix Structural Conflicts (still no backend, but schema-compatible)
  └─ Replace age field with DOB picker + derived age display
  └─ Add MRN generation mechanism
  └─ Extract consultation fee from procedure grid → session-level toggle
  └─ Swap PHYSIO_PROCEDURES with services table catalog (SVC-01–SVC-13)
  └─ Implement is_charged / charged_amount_in_paise logic
  └─ Remove pre-generated invoice number; show placeholder until minted
  └─ Thread paymentMode through FooterActions to submission handler
  └─ Preserve complaint_catalog_id in handleCatalogSelect

Phase 3: Supabase Wiring (actual integration)
  └─ Wire DailyLedger to daily_ledger view + patient search RPC
  └─ Wire NewPatientIntake to patients INSERT (returning id for notes)
  └─ Wire ComplaintSelector to complaint_courses + complaint_catalog queries
  └─ Wire ProcedureLogger to checkout_and_create_invoice() coordinated RPC
  └─ Wire InvoiceBuilder to confirm_invoice_payment() mutation
  └─ Add RLS-aware cross-branch search (Decision 4)
  └─ Resolve status badge backing (Decision 3)
```

> [!IMPORTANT]
> **Phase 1 and Phase 2 can be done independently and in parallel.** Phase 3 depends on both being complete. Phase 1 is the easier lift (~2-3 days); Phase 2 is the harder one because the ProcedureLogger's billing model needs to be rebuilt from scratch to match the schema's fee structure.

---

## Screen 1: Daily Ledger (`DailyLedger`)

> [Component files](file:///c:/Github/clinic-demo/src/features/ledger) • [Workflow spec: Step 0](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L27-L42) • [Decisions 3 & 4](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L166-L183)

### Clinical Intent vs. Current State

The Daily Ledger is the receptionist's home terminal — a live, clinic-scoped register of today's OPD activity that doubles as the search gateway into the workflow. Currently:

- **Dead-end handlers**: Selecting a patient fires `alert()` instead of routing to Step 2. Selecting "Register New" fires `alert()` instead of routing to Step 1.
- **Wrong click target**: Table rows navigate to `/patient/:id` (Patient Profile) instead of continuing the OPD visit stepper.
- **Zero props contract**: `DailyLedger` and `PatientSearch` accept no callbacks (`onSelectPatient`, `onRegisterNewPatient`), making workflow orchestration impossible.

### Backend Schema Divergences

> [!CAUTION]
> **Decision 3 — Status Badges Have No DB Home**: The UI prominently displays `Waiting`, `In Therapy`, and `Paid` badges. The Supabase schema is strictly single-touch: *"Row existing = session complete + paid."* The `daily_ledger` view hardcodes `'Complete' as status`. In-progress patients have **no row anywhere** in the transactional schema.

- **Cross-Branch Search (Decision 4)**: Under real staff RLS, `patients` SELECT is scoped to rows where `patient_clinic_access` exists. A patient registered at Branch A visiting Branch B for the first time will return zero results → receptionist registers a duplicate → violates `unique(owner_id, mrn)`.
- **MRN Search Broken**: [`cleanSearchInput()`](file:///c:/Github/clinic-demo/src/lib/inputValidation.tsx) strips hyphens and digits from alphabetic queries. Typing `"MED-001"` becomes `"MED "`, completely disabling MRN search.

### Design Principles Violations

| Principle | Violation | Location |
| :--- | :--- | :--- |
| AI Default: ALL-CAPS labels | `uppercase tracking-widest` on "OPD COUNT", column headers, dividers | [LedgerHeader.tsx:23](file:///c:/Github/clinic-demo/src/features/ledger/components/LedgerHeader.tsx#L23), [TableHeader.tsx:9](file:///c:/Github/clinic-demo/src/features/ledger/components/TableHeader.tsx#L9) |
| AI Default: Monospace data labels | `font-mono` on OPD count, time column, patient metadata | [LedgerHeader.tsx:22](file:///c:/Github/clinic-demo/src/features/ledger/components/LedgerHeader.tsx#L22), [TableRow.tsx:28](file:///c:/Github/clinic-demo/src/features/ledger/components/TableRow.tsx#L28) |
| AI Default: Middot separator | `<span>•</span>` in patient result metadata | PatientResult.tsx:96 |
| AI Default: Arrow icon on links | `<ArrowUpRightIcon>` tacked onto every search result | PatientResult.tsx:103 |
| React: In-render sort (§3 avoid recomputation) | `MOCK_LEDGER_ENTRIES.sort(...)` mutates array **in-place during render** | [TableBody.tsx:78-83](file:///c:/Github/clinic-demo/src/features/ledger/components/TableBody.tsx#L78-L83) |
| React: Duplicated state (§3 derive don't duplicate) | `filteredPatients` stored in state instead of derived from `input` | [PatientSearch/index.tsx:13](file:///c:/Github/clinic-demo/src/features/ledger/components/PatientSearch/index.tsx#L13) |
| React: No debounce (§3 debounce input-driven filtering) | Synchronous `.filter()` on every keystroke | [PatientSearch/index.tsx:31-35](file:///c:/Github/clinic-demo/src/features/ledger/components/PatientSearch/index.tsx#L31-L35) |
| Accessibility: No ARIA combobox | Search input lacks `role="combobox"`, `aria-expanded`, `aria-controls` | SearchInput.tsx:32-44 |
| Accessibility: Inaccessible table rows | Clickable `<div>` with no `role="button"`, `tabIndex`, or keyboard handler | [TableRow.tsx:16-25](file:///c:/Github/clinic-demo/src/features/ledger/components/TableRow.tsx#L16-L25) |

### Critical Bugs

1. **In-place array mutation** ([TableBody.tsx:78](file:///c:/Github/clinic-demo/src/features/ledger/components/TableBody.tsx#L78)): `.sort()` mutates module-scoped `MOCK_LEDGER_ENTRIES` on every render.
2. **Broken CSS selector** (primitives.tsx:48-52): `group-data-[selected=true]` child selectors never apply because `<button>` lacks `data-selected={isSelected}` attribute. String interpolation evaluates `false` to literal `"false"` in className.
3. **Missing React keys** (PatientResult.tsx:21-27): `<PatientListItem>` rendered inside `.map()` without a `key` prop on the root element.
4. **Timer leak** ([TableBody.tsx:52-59](file:///c:/Github/clinic-demo/src/features/ledger/components/TableBody.tsx#L52-L59)): `snapBackTimer` never cleared on unmount; fires against unmounted DOM node.
5. **Non-standard scrollTop** ([TableBody.tsx:29](file:///c:/Github/clinic-demo/src/features/ledger/components/TableBody.tsx#L29)): Assumes negative `scrollTop` in `flex-col-reverse`, which is non-standard CSSOM and breaks in Safari/Firefox.

### Strengths

- Centralized layout system ([`ledgerRow.styles.tsx`](file:///c:/Github/clinic-demo/src/features/ledger/ledgerRow.styles.tsx)) guarantees column alignment
- Thoughtful mobile responsive adaptation (treatment column moves to subtitle on small screens)
- Comprehensive dark mode coverage
- Keyboard arrow navigation in search with contextual user hints

---

## Screen 2: New Patient Registration (`NewPatientIntake`)

> [Component files](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake) • [Workflow spec: Step 1](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L45-L61) • [Decision 5](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L186-L192)

### Clinical Intent vs. Current State

Patient registration for an active Indian OPD clinic. The form captures demographics, referral source, and optional clinical baseline notes via the `ClinicalNotesBuilder` modal.

- **Broken keyboard shortcut**: Cancel button shows `"Cancel (Esc)"` but no `Escape` keydown listener exists on `NewPatientIntake`.
- **Cursor jumping**: Real-time `capitalizeEachWord()` on every keystroke resets cursor position to end of input when editing mid-name.
- **Clinically unrealistic field bounds**: Name minimum 5 chars rejects legitimate Indian names ("Ali", "Ram", "Uma", "Dev"). Address max 50 chars rejects standard Indian postal addresses (typical: 80-100 chars).
- **Clinical scope blurring**: Attaching diagnostic observations ("fall risk", "surgical history") at the reception desk during demographic intake mixes administrative onboarding with clinical triage.

### Backend Schema Divergences

| Field | Backend (`supabase_migration.md`) | Frontend | Impact |
| :--- | :--- | :--- | :--- |
| **Age vs DOB** | `patients.date_of_birth date` | `age: string` (2-digit number) | 🔴 Cannot write to `date_of_birth`. Age becomes stale annually. |
| **Sex vs Gender** | `patients.gender` | `FormData['sex']` | 🟡 Key name mismatch; values match. Adapter needed. |
| **Referral constraint** | `chk_referral_doctor_info`: must be `NULL` when not `DOCTOR` | Reducer removes `doctorInfo` key, but empty string `""` ≠ `NULL` in PostgreSQL | 🟡 Empty string triggers CHECK failure. Must explicitly send `null`. |
| **Clinical Notes sequencing** | `clinical_notes.patient_id NOT NULL FK` | Bundled in flat `FormData` | 🔴 Notes cannot INSERT simultaneously with patients. Must await returned `patient.id`. |
| **MRN generation** | `patients.mrn text NOT NULL` with no trigger/counter | No MRN field, no generator | 🔴 Insert throws `NOT NULL violation`. |
| **Clinician** | `patients.clinician_id uuid` (nullable) | No selector | 🟢 Accepted scope gap per `GAPS.md`. |

### Design Principles Violations

| Principle | Violation | Location |
| :--- | :--- | :--- |
| AI Default: ALL-CAPS tracked labels | `text-[10px] font-bold uppercase tracking-widest` on every field label | [index.tsx:201](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/index.tsx#L201), DemographicsSection, AddressArea, ReferralSection |
| AI Default: Monospace misuse | `font-mono` on phone input AND patient address textarea | [DemographicsSection.tsx:39](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/components/DemographicsSection.tsx#L39), [primitives.tsx:65,79](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/components/primitives.tsx#L65) |
| Writing: Error messaging | Native `alert(errors.join('\n'))` for validation | [index.tsx:126](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/index.tsx#L126) |
| Writing: Action copy | Trailing ellipsis: `"Create Profile & Proceed..."` | FormFooter.tsx:64 |
| Accessibility: Tab order | `tabIndex={-1}` on Clinical Notes button locks out keyboard users | [FormFooter.tsx:23](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/components/FormFooter.tsx#L23) |
| Accessibility: Missing ARIA | Close buttons lack `aria-label`. Critical toggle lacks `aria-pressed`. | FormHeader.tsx:24, CNBInput.tsx:67 |
| React: Dual state (§2 colocation) | `ClinicalNotesBuilder` copies `initialNotes` to local state; parent updates silently discarded | [index.tsx:175-184](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/index.tsx#L175-L184) (marked `HACK`) |

### Critical Bugs

1. **Empty age bypasses validation** ([index.tsx:18](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/index.tsx#L18)): `+""` evaluates to `0`, passing the `< 0 || > 99` check. Empty age field silently submits as age 0.
2. **Doctor info validated as address** ([index.tsx:21-22](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/index.tsx#L21-L22)): `validateAddress(data.doctorInfo)` applies street address regex to doctor names/specialties.
3. **In-place state mutation** (ClinicalNotesBuilder/index.tsx:17): `Array.prototype.sort()` mutates `notes` state in-place during render.
4. **`maxLength` ignored on number input** ([DemographicsSection.tsx:51](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/components/DemographicsSection.tsx#L51)): HTML5 spec: `maxLength` is ignored on `type="number"`.
5. **Missing `type="button"`** (CNBInput.tsx:67): Critical toggle button can accidentally trigger form submission.

### Strengths

- Strongly-typed discriminated union for referral handling ensures compile-time safety
- Real-time defensive input filtering (regex utilities block invalid chars at keystroke level)
- Reactive conditional UI with auto-focus on referral doctor field
- Clean `useReducer` state machine with typed actions
- Responsive grid layout (`sm:grid-cols-12`) adapts cleanly to mobile

---

## Screen 3: Complaint Selector (`ComplaintSelector`)

> [Component files](file:///c:/Github/clinic-demo/src/features/visit/ComplaintSelector) • [Workflow spec: Step 2](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L64-L80) • [Decision 2](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L156-L163)

### Clinical Intent vs. Current State

Enables the clinician to review the patient's ongoing therapy conditions, check off which complaints are being treated today, and register new conditions from the catalog or free text.

> [!CAUTION]
> **Free-Text Entry Is Non-Functional**: [`handleKeyDown`](file:///c:/Github/clinic-demo/src/features/visit/ComplaintSelector/index.tsx#L139-L168) only intercepts `Enter` when `filteredCatalogItems.length > 0`. When no catalog match exists, `Enter` does nothing. There is no submit button or fallback `add()` call. Clinicians cannot add uncatalogued conditions.

- **Output contract broken**: `onConfirm` emits only `selectedIds: string[]`. Parent cannot reconstruct `MedicalComplaint[]` objects for custom additions to pass to ProcedureLogger.

### Backend Schema Divergences

1. **Catalog ID dropped** ([index.tsx:183-193](file:///c:/Github/clinic-demo/src/features/visit/ComplaintSelector/index.tsx#L183-L193)): `handleCatalogSelect` calls `add(item.title)`, discarding `item.id`. The DB requires `complaint_courses.complaint_catalog_id` for accurate categorization.
2. **Write-timing trigger trap** (Decision 2): `complaint_courses` must INSERT before `visits` to provision `patient_clinic_access` via `process_new_complaint_course()`. If deferred to local state, downstream `visits` INSERT triggers `validate_visit_clinic_id()` exception.
3. **Multi-complaint vs singular FK**: "MULTIPLE ALLOWED" produces multiple `visits` rows, but `invoices.visit_id` is a singular FK.
4. **Chain-wide `last_visit_at`**: PatientHeader displays "Last visit: X" from `patients.last_visit_at`, which is chain-wide, not branch-specific.

### Design Principles Violations

- **AI defaults**: Tracked-out uppercase labels (`SectionLabel`), middot meta separators (PatientHeader), arrow button (`Continue to Procedures →`), monospace MRN display.
- **Cramped popover**: `max-h-28` (~112px, ~2.5 catalog cards) is excessively small for browsing categorized lists.
- **Render-phase state setting** ([index.tsx:62-68](file:///c:/Github/clinic-demo/src/features/visit/ComplaintSelector/index.tsx#L62-L68)): Comparing `filteredCatalogItems` during render and invoking `setPrevFilteredCatalogItems` triggers cascading re-renders.
- **Unmemoized render computation** ([index.tsx:200-205](file:///c:/Github/clinic-demo/src/features/visit/ComplaintSelector/index.tsx#L200-L205)): `catalogExistingIds` creates a new `Set` every render, causing downstream `useMemo` to re-run constantly.

### Critical Bugs

1. **Event bubbling on delete** (ComplaintItem.tsx:43-51): Delete button lacks `e.stopPropagation()`. Clicking delete fires both `remove` AND the row's `onToggle`, causing state race. Button uses `hidden group-hover:block`, making it keyboard/touch inaccessible.
2. **Inaccessible checklist rows** (ComplaintItem.tsx:19-27): `<div onClick>` with no `role="checkbox"`, `aria-checked`, `tabIndex`, or `onKeyDown`.
3. **Hardcoded MRN** (PatientHeader.tsx:15): Hardcodes `"MRN-9921"` instead of using `patient.mrn`.
4. **Missing button types** (FooterActions.tsx:15,21): `<button>` without `type="button"` risks accidental form submission.

### Strengths

- **Exemplary ARIA combobox** in `NewComplaintInput`: `role="combobox"`, `aria-controls`, `aria-activedescendant`, `aria-expanded` using dynamic IDs from `useId()`
- Smooth keyboard orchestration: Arrow keys, Enter selection, `Ctrl+Arrow` region navigation, Escape to close
- Auto-scroll to bottom on new complaint addition with layout paint delay
- Portal-anchored popover escaping overflow/z-index constraints
- O(1) selection performance via `Set<string>` in `useComplaintSelection`

---

## Screen 4: Procedure Logger (`ProcedureLogger`)

> [Component files](file:///c:/Github/clinic-demo/src/features/visit/ProcedureLogger) • [Workflow spec: Step 3](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L83-L98) • [Decision 6](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L195-L201) • [Specialty Decision Log](file:///c:/Github/clinic-demo-be/study/business_logic/speciality_and_invoice/specialty-decision-log.md)

### Clinical Intent vs. Current State

The treatment logging workspace where procedures are assigned per complaint with a live running bill.

> [!CAUTION]
> **Consultation Fee Model Is Inverted**: "Consultation" is modeled as an ordinary ₹500 checkbox procedure scoped per complaint. Two complaints checked = two ₹500 charges = ₹1,000. The confirmed business rule charges the exam fee **once per session** (₹350), shared across all complaints. `visits.consultation_fee_in_paise` is a dedicated snapshot column, entirely separate from `visit_services`.

- **No anatomical filtering**: Every complaint gets the identical 8-procedure grid. "Cervical Traction" displays under "Femur Fracture" with equal prominence.
- **Legacy catalog IDs**: Uses `PHYSIO_PROCEDURES` (`PROC_01`–`PROC_08`) instead of canonical `services` table (`SVC-01`–`SVC-13`). Foreign keys will fail on insert.
- **No `is_charged` logic**: `visit_services` requires `is_charged` and `charged_amount_in_paise`. Standard modalities under `CONSULTATION` visits should be `is_charged = false, charged_amount_in_paise = 0`. The component bills everything unconditionally.
- **No clinic price overrides**: `clinic_service_prices` is never consulted.

### Backend Schema Divergences

| Schema Concept | Backend Contract | Frontend Current State | Severity |
| :--- | :--- | :--- | :--- |
| Consultation fee | `visits.consultation_fee_in_paise` (once per session) | Per-complaint checkbox `PROC_01 = ₹500` | 🔴 Blocker (Decision 6) |
| Service catalog | `services` (`SVC-01`–`SVC-13`) | Legacy `PHYSIO_PROCEDURES` (`PROC_01`–`PROC_08`) | 🔴 FK failure |
| Junction semantics | `visit_services.is_charged`, `charged_amount_in_paise` | Flat `InvoiceItem` with unconditional `cost` | 🔴 Inflates total |
| Client ID reconciliation | `visits.complaint_course_id` (real UUID FK) | Ephemeral `crypto.randomUUID()` from Step 2 | 🔴 Trigger exception |

### Design Principles Violations

- **AI defaults**: ALL-CAPS eyebrows ("Common Procedures", "Current Session Bill"), decorative numbered badges on concurrent complaints (not sequential steps), staggered entrance animations, monospace price labels.
- **Missing accessibility**: `ProcedureCard` button lacks `role="checkbox"`, `aria-checked`, and `focus-visible` rings. Search input lacks `<label>` and `aria-label`.
- **Ineffective memoization** ([useProcedureLogger.tsx:61-62](file:///c:/Github/clinic-demo/src/features/visit/ProcedureLogger/hook/useProcedureLogger.tsx#L61-L62)): `isProcedureSelected` wrapped in `useCallback([items])` — but `items` changes on every click, defeating referential stability. Meanwhile `getItemsForComplaint` is un-memoized (inconsistent).
- **Inert UI element** (primitives.tsx:84-89): Search input is a purely decorative stub — no `value`, no `onChange`, no ref. Also misspells "Laser" as "Lazer".
- **Hardcoded patient** ([index.tsx:44](file:///c:/Github/clinic-demo/src/features/visit/ProcedureLogger/index.tsx#L44)): `patientName="Amit Trivedi"` while Step 5 uses `"Amitabh Bachchan"`.
- **Filename typos**: `RecieptFooter.tsx`, `ReciptListProps`.

### Strengths

- **Cleanest hook extraction in the codebase**: `useProcedureLogger` fully isolates selection records, derived totals, and key generation from JSX
- Composite key architecture (`${ctxId}::${procId}`) enables independent multi-complaint toggling in a flat dictionary
- Derived billing state (`totalCost`, `hasItems`) via `useMemo` prevents state duplication
- Defensive validation boundary (`validateInvoiceItems`) strips orphaned selections on submit
- Clean component hierarchy with crisp single responsibilities

---

## Screen 5: Invoice & Payment (`InvoiceBuilder`)

> [Component files](file:///c:/Github/clinic-demo/src/features/visit/InvoiceBuilder) • [Workflow spec: Step 4](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L101-L118) • [Decision 1](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md#L146-L153)

### Clinical Intent vs. Current State

The financial checkout terminal — itemized billing, payment mode, and confirmation.

- **No Pay Later**: Missing button for `payment_status = 'Pending'`. Receptionist has no recourse if patient cannot pay immediately.
- **Non-functional receipt**: "Print Receipt" fires `alert("Yet to be implemented")`.
- **Payment mode dropped**: `paymentMode` state exists in `InvoiceBuilder` but is **never passed** to `FooterActions` or emitted via any callback. "Confirm Payment" calls raw `onClose`.
- **Pipeline break**: ProcedureLogger's `onComplete` does not pipe `InvoiceItem[]` into InvoiceBuilder. They render independent hardcoded mock data.

### Backend Schema Divergences

> [!WARNING]
> **Pre-Generated Invoice Number Burns Atomic Counter**: [`vistitworkflow.tsx:146`](file:///c:/Github/clinic-demo/src/pages/vistitworkflow.tsx#L146) passes hardcoded `'INV-2026-0306-088'`. In PostgreSQL, `process_new_invoice()` atomically increments `clinics.invoice_counter` on INSERT. Pre-generating burns a number if the user abandons. Additionally, the mock format (3 segments) doesn't match the trigger output (`INV-YYYY-XXXX`, 2 segments).

- **Dead code**: `InvoiceHeader` declares `clinicName?`, `clinicCode?`, `doctorName?` props but **never renders them**.
- **Multi-complaint vs singular FK**: Items span multiple complaints but `invoices.visit_id` is singular.
- **Resolved by specialty-decision-log**: Decision 1 resolved — invoice write fires on "Create Invoice" as `payment_status = 'Pending'`, then "Confirm Payment" executes `UPDATE SET payment_status = 'Paid', payment_mode = :mode`.

### Design Principles Violations

- **AI defaults**: ALL-CAPS labels ("AMOUNT DUE", "PROCEDURE", "COST", "Payment Mode"), monospace price formatting.
- **Isolated bilingual text** (PaymentSelector.tsx:23): `"Payment Mode (भुगतान का प्रकार)"` — the **only Hindi label** in the entire English-only application.
- **Invalid DOM** ([ItemTable.tsx:10-13](file:///c:/Github/clinic-demo/src/features/visit/InvoiceBuilder/components/ItemTable.tsx#L10-L13)): `<div>` rendered as immediate child of `<table>`. Invalid HTML5, triggers hydration warnings.
- **CSS collision** ([FooterActions.tsx:21-22](file:///c:/Github/clinic-demo/src/features/visit/InvoiceBuilder/components/FooterActions.tsx#L21-L22)): Two conflicting `hover:disabled` classes — zinc overridden by emerald, causing disabled buttons to turn green on hover.
- **Missing accessibility**: Payment buttons lack `role="radio"`, `role="radiogroup"`, `aria-checked`, and focus-visible rings.
- **Formatting inconsistency**: `InvoiceHeader` uses `rupee.format(total)` (proper `₹1,700`) while `primitives.tsx` uses raw `₹{item.cost}` (missing thousands grouping).
- **Writing**: Empty state says `"No items here."` — passive and dead-ended. Should invite action: *"No procedures logged. Return to Procedure Logger to add treatments."*

### Strengths

- Clean modular decomposition (Header, ItemTable, PaymentSelector, FooterActions)
- Indian market awareness: UPI pre-selected as default payment mode
- Centralized `Intl.NumberFormat('en-IN')` currency formatting
- Stable composite keys (`${complaintId}-${procedureId}`) in item rendering
- Consistent dark mode coverage

---

## Cross-Cutting Findings

### A. Systemic Design Principle Violations (Present in ALL 5 Screens)

1. **Tracked-out ALL-CAPS labels** (`text-[10px] font-bold uppercase tracking-widest`): Found in every single screen. This is explicitly listed in `frontend-design-principles.md` Section 1 as an "AI-generated default to actively avoid."
2. **Monospace for non-code data**: `font-mono` applied to patient metadata, OPD counts, time columns, price labels, and even residential addresses. The principles state this should only be used for actual code or genuinely tabular numeric data.
3. **Native `alert()` for user feedback**: Used for validation errors (NewPatientIntake), selection confirmation (PatientSearch), billing (ProcedureLogger), and print stubs (InvoiceBuilder).

### B. Systemic Backend Blockers (Must Resolve Before Wiring)

1. **Write sequencing invariant**: `patients` → `complaint_courses` (provisions access) → `visits` (validates access) → `invoices` (mints number). This strict ordering is enforced by database triggers and cannot be bypassed.
2. **Session entity absence**: The schema has no first-class session/encounter table grouping multiple complaints treated on the same day. Multi-complaint checkout requires either `invoice_line_items` junction table (Option B) or a coordinated RPC.
3. **Consultation fee decoupling**: Must be extracted from the procedure grid and modeled as a session-level calculation mapped to `visits.consultation_fee_in_paise`.
4. **Client UUID reconciliation**: Ephemeral `crypto.randomUUID()` IDs from Step 2 must be reconciled with real `complaint_courses.id` values before Step 3/4 can persist.

### C. Accessibility Scorecard

| Screen | Keyboard Navigation | ARIA Semantics | Focus States | Screen Reader Support |
| :--- | :--- | :--- | :--- | :--- |
| DailyLedger | 🟡 Search only | ❌ Missing combobox ARIA | ❌ Table rows unreachable | ❌ No labels on input |
| NewPatientIntake | 🟡 Partial (`tabIndex={-1}` blocks notes) | ❌ Missing `aria-label` on buttons | 🟡 Inputs have rings, buttons don't | ❌ Escape handler missing |
| ComplaintSelector | ✅ Excellent | ✅ Full combobox ARIA via `useId()` | 🟡 Delete button keyboard-inaccessible | ✅ `aria-activedescendant` |
| ProcedureLogger | ❌ No checkbox semantics | ❌ Missing `role="checkbox"`, `aria-checked` | ❌ No focus-visible rings | ❌ Search input unlabeled |
| InvoiceBuilder | ❌ No radio semantics | ❌ Missing `role="radiogroup"`, `aria-checked` | ❌ No focus-visible rings | ❌ Payment selection invisible |

---

## Prioritized Action Plan

### Phase 1: Critical Bugs & React Hygiene (Fix in isolation now)

| Priority | Screen | Fix | Files |
| :--- | :--- | :--- | :--- |
| 🔴 P0 | Ledger | Copy array before sorting: `[...entries].sort(...)` or `useMemo` | [TableBody.tsx:78](file:///c:/Github/clinic-demo/src/features/ledger/components/TableBody.tsx#L78) |
| 🔴 P0 | Ledger | Add `data-selected={isSelected}` to `ListItemButton`; fix `"false"` string interpolation | primitives.tsx:46-52 |
| 🔴 P0 | Intake | Fix empty age validation: `data.age.trim() === '' \|\| isNaN(+data.age)` | [index.tsx:18](file:///c:/Github/clinic-demo/src/features/patient/NewPatientIntake/index.tsx#L18) |
| 🔴 P0 | Invoice | Fix invalid `<div>` inside `<table>`: wrap in `<tbody><tr><td colSpan={2}>` | [ItemTable.tsx:10-13](file:///c:/Github/clinic-demo/src/features/visit/InvoiceBuilder/components/ItemTable.tsx#L10-L13) |
| 🔴 P0 | Invoice | Remove conflicting `hover:disabled` CSS classes | [FooterActions.tsx:21-22](file:///c:/Github/clinic-demo/src/features/visit/InvoiceBuilder/components/FooterActions.tsx#L21-L22) |
| 🔴 P0 | Complaint | Add `e.stopPropagation()` to delete button; make keyboard accessible | ComplaintItem.tsx:43-51 |
| 🔴 P0 | Complaint | Fix free-text entry: handle `Enter` when `filteredCatalogItems.length === 0` | [index.tsx:139-168](file:///c:/Github/clinic-demo/src/features/visit/ComplaintSelector/index.tsx#L139-L168) |
| 🟡 P1 | All 5 | Remove ALL-CAPS `uppercase tracking-widest` labels; use sentence/title case | All label components |
| 🟡 P1 | All 5 | Remove `font-mono` from non-code contexts (addresses, metadata, prices) | All primitive files |
| 🟡 P1 | All 5 | Replace `alert()` with inline validation messages or toast notifications | All handler files |

### Phase 2: Backend Integration Blockers (Must resolve before Supabase wiring)

| Priority | Screen | Fix | Decision Ref |
| :--- | :--- | :--- | :--- |
| 🔴 P0 | Intake | Transition `age` to `date_of_birth` (DOB picker with derived age) | — |
| 🔴 P0 | Intake | Implement MRN auto-generation (client or trigger) | Decision 5 |
| 🔴 P0 | Intake | Separate clinical_notes write to fire AFTER patients INSERT returns `id` | — |
| 🔴 P0 | Complaint | Preserve `complaint_catalog_id` in `handleCatalogSelect` | Decision 2 |
| 🔴 P0 | Complaint | Resolve write timing: immediate INSERT vs. deferred coordinated RPC | Decision 2 |
| 🔴 P0 | Procedure | Remove "Consultation" from procedure grid; model as session-level toggle | Decision 6 |
| 🔴 P0 | Procedure | Align catalog IDs with `services` table (`SVC-01`–`SVC-13`) | — |
| 🔴 P0 | Procedure | Implement `is_charged` / `charged_amount_in_paise` logic per `visit_services` | — |
| 🔴 P0 | Invoice | Remove pre-generated invoice number; display "Auto-generated on confirmation" | Decision 1 |
| 🔴 P0 | Invoice | Thread `paymentMode` through `FooterActions` to submission handler | — |
| 🟡 P1 | Ledger | Resolve status badge backing (new table/column or ephemeral-only) | Decision 3 |
| 🟡 P1 | Ledger | Create elevated search RPC for cross-branch patient discovery under RLS | Decision 4 |
| 🟡 P1 | Invoice | Add "Pay Later" button (`payment_status = 'Pending'`) | — |

---

## Sources

| Document | Location | Role in Audit |
| :--- | :--- | :--- |
| Frontend Workflow Spec | [`clinic-demo-be/docs/FRONTEND_WORKFLOW.md`](file:///c:/Github/clinic-demo-be/docs/FRONTEND_WORKFLOW.md) | 5-step flow, open Decisions 1–6, divergence catalog |
| Supabase Migration v11 | [`clinic-demo-be/docs/supabase_migration.md`](file:///c:/Github/clinic-demo-be/docs/supabase_migration.md) | Canonical schema, triggers, CHECK constraints, RLS |
| Schema Cross-Reference | [`clinic-demo-be/docs/schema_cross_reference.md`](file:///c:/Github/clinic-demo-be/docs/schema_cross_reference.md) | TS interface ↔ DB column alignment |
| Schema Iterations | [`clinic-demo-be/docs/backend-schema-iterations.md`](file:///c:/Github/clinic-demo-be/docs/backend-schema-iterations.md) | Architectural evolution v1–v11, deadlock fixes |
| Specialty Decision Log | [`clinic-demo-be/study/business_logic/speciality_and_invoice/specialty-decision-log.md`](file:///c:/Github/clinic-demo-be/study/business_logic/speciality_and_invoice/specialty-decision-log.md) | Session architecture, coordinated writes, fee rules |
| Specialty UX Mockups | [`clinic-demo-be/study/business_logic/speciality_and_invoice/specialty-treatment-ux-mockups.md`](file:///c:/Github/clinic-demo-be/study/business_logic/speciality_and_invoice/specialty-treatment-ux-mockups.md) | Approach D modal for specialty treatments |
| Module 1 Decisions | [`clinic-demo-be/study/module_1/DECISIONS.md`](file:///c:/Github/clinic-demo-be/study/module_1/DECISIONS.md) | Clinician attribution, invoice/session options |
| Module 1 Gaps | [`clinic-demo-be/study/module_1/GAPS.md`](file:///c:/Github/clinic-demo-be/study/module_1/GAPS.md) | Cross-chain validation, missing `updated_at`, session absence |
| Module 2 Logs | [`clinic-demo-be/study/module_2/LOGS.md`](file:///c:/Github/clinic-demo-be/study/module_2/LOGS.md) | Staff write isolation, denormalized snapshots |
| Wiring Proposal | [`clinic-demo-be/plans/frontend-backend-wiriing-proposal.md`](file:///c:/Github/clinic-demo-be/plans/frontend-backend-wiriing-proposal.md) | 4-axis integration architecture |
| Design Principles | [`clinic-demo-be/.agents/workflows/frontend-design-principles.md`](file:///c:/Github/clinic-demo-be/.agents/workflows/frontend-design-principles.md) | Visual restraint, React engineering, accessibility |
| Component Source | [`clinic-demo/src/features/`](file:///c:/Github/clinic-demo/src/features) | All component files audited line-by-line |
