# PATIENT_PROFILE.md — Frontend Patient Profile Documentation (Comprehensive Clinical Record)

**Scope:** Patient Profile module (`/patient/:id`). Provides exhaustive end-to-end documentation across:
1. **Responsive Shell:** Desktop two-column master-detail split vs. Tablet/Mobile panel switcher (`Profile` vs `History`).
2. **Passport Panel:** Demographics sidebar (Avatar, Full Name, Age/Gender, Critical Alerts, MRN, Blood Type, Insurer).
3. **Overview Tab:** Vitals Grid (**confirmed placeholder**) + Complaint History Timeline with deep-link navigation.
4. **Visits Tab:** Encounter history table, dual-axis filtering (Visit Type & Complaint Course), fee calculation visualization (Doctor Prescribed vs. Machine-Only), line-item procedure pricing, and theme/data bugs.
5. **Packages Tab:** Treatment package cards (`PackageCard`), attendance tracking with permanent interactive dot grid (`dayLog`), Sunday-exclusion date math, and backend deferral boundaries.
6. **Billings Tab:** Invoice history ledger, payment status badges, and documentation of its premature placeholder status.

**Basis:** 6 screenshots provided (Sept 3 2026), cross-referenced against `src/features/patient/PatientProfile/` (`index.tsx`, `PassportPanel.tsx`, `ClinicalHistoryPanel.tsx`, `tabs/OverviewTab.tsx`, `tabs/VitalsGrid.tsx`, `tabs/ClinicalTimeline.tsx`, `tabs/VisitsTab.tsx`, `tabs/PackagesTab.tsx`, `tabs/BillingsTab.tsx`, `components/PackageCard.tsx`, `components/ServiceTag.tsx`, `hooks/usePatientProfileUrlState.ts`), `src/data/mock_data.ts`, `supabase_migration.md` (v11 canonical), `study/business_logic/packages_logic/DEFERRAL.md`, `README.md` (fee rules), and `docs/FRONTEND_WORKFLOW.md`.
- **Screenshot 1 (Responsive Tablet/Mobile — "Profile" Panel):** Standalone `PassportPanel` rendered when mobile switcher is toggled to `Profile` (`PK`, Priya Kapoor, 38 Yrs · Female, `Penicillin Allergy`, `MRN: MED-001`, `Blood Type: B+`, `Insurer: Star Health Insurance`).
- **Screenshot 2 (Responsive Tablet/Mobile — "History" Panel):** `ClinicalHistoryPanel` rendered when mobile switcher is toggled to `History` with `Overview` active, displaying 4 vitals cards (BP, HR, Temp, SpO2) and the vertical `Complaint History` timeline (`Ankle Sprain`, `Cervical Spondylosis`, `Plantar Fasciitis`).
- **Screenshot 3 (Desktop Mode — Full Split Screen):** Master-detail side-by-side view with `PassportPanel` pinned as a left sidebar (w-72) and `ClinicalHistoryPanel` taking the remaining flex area with the tab bar (`Overview`, `Visits`, `Packages`, `Billings`) and overview content.
- **Screenshot 4 (Visits Tab — Encounter Table & Dropdown Filter):** Priya Kapoor's visits list showing `10 of 18 visits`, open complaint selector dropdown (`All complaints`, `Ankle Sprain · Active` checked), segmented visit type filter (`All` / `Consult` / `Machine Only`), procedure tags with conditional itemized pricing, and surfacing an unintended light-mode UI state.
- **Screenshot 5 (Packages Tab — Stacked View):** 3 package cards stacked vertically on tablet viewport (`Chronic Back Pain — 20 Day Package` [Active, 6 Days Left, 14 of 20 elapsed, 12 attended, 2 missed, Excl. Sundays], `Ankle Rehab — 10 Day Package` [Completed, 10 Days Total], `IFT Therapy — 5 Day Package` [Completed, 5 Days Total]).
- **Screenshot 6 (Packages Tab — Desktop 2-Column Grid):** Responsive 2-column grid (`xl:grid-cols-2`) showing Card 1 (Active) and Card 2 (Completed) side-by-side, with Card 3 (Completed) in the second row.

**Wiring status:** 100% mock data in frontend state (`MOCK_PATIENT_PROFILES`, `MOCK_VISITS_V2`, `MOCK_COMPLAINT_COURSES`, `MOCK_PACKAGES`, `MOCK_INVOICES`). Includes a simulated 20% random network error (`fetchPatientProfile` in `index.tsx`, Ref: ADR-PP-11). Not connected to live Supabase backend. Packages module is catalog-phase mock data, explicitly deferred from backend MVP implementation. Billings tab is an immature placeholder.

**Legend:** `MOCK` = catalog-phase, hardcoded/simulated · `NOT YET WIRED` = no live Supabase call exists · `PLANNED` = named in mockup, not built · `CONFIRMED` = matches code/schema audit · `NEW` = surfaced by this pass · 🔴/🟡/🟢 = structural / safety-net / performance tagging convention.

---

## Section 1: Architectural & Responsive Shell Matrix

| Layout Mode / Tab | Component / View | Screen Breakpoint | Initiated By | Reads (SELECT / Props) | Writes (URL / Local State) | Status |
|---|---|---|---|---|---|---|
| **Desktop Shell** | Master-Detail Container | `lg:flex-row` ($\ge 1024\text{px}$) | Route mount `/patient/:id` | `patients`, `patient_alerts`, `patient_vitals`, `complaint_courses`, `visits`, `invoices`, `packages` | — | `MOCK` — in-memory arrays filtered by `patientId` |
| **Tablet / Mobile** | Panel Switcher Header | `< lg` ($< 1024\text{px}$) | User click | Active panel state (`'profile' \| 'history'`) | Local state: `setActivePanel()` | `CONFIRMED` (`index.tsx` L114–129) |
| **Tablet / Mobile** | Profile Panel View | `< lg` (activePanel = `'profile'`) | Panel toggle | Patient profile demographics & alerts | — (Read-only) | `MOCK` (`PassportPanel.tsx`) |
| **Tablet / Mobile** | History Panel View | `< lg` (activePanel = `'history'`) | Panel toggle | Clinical history, vitals, timeline, tabs | URL state: `?tab=overview` | `MOCK` (`ClinicalHistoryPanel.tsx`) |
| **Overview Tab** | Vitals Grid | All viewports | Tab active (`tab=overview`) | `patient_vitals` (**PLACEHOLDER ONLY**) | — | `MOCK` — 🔴 Domain-incompatible dummy data |
| **Overview Tab** | Complaint Timeline | All viewports | Tab active (`tab=overview`) | `complaint_courses` (sorted active-first) | Navigation click: `goToVisits({ complaintId })` | `MOCK` — transitions to `?tab=visits&complaint=...` |
| **Visits Tab** | Filter Toolbar | All viewports | Tab active (`tab=visits`) | `courses` (for dropdown list), `visits.length` | URL state: `setVisitType()`, `setComplaintId()`, `clearVisitFilters()` | `CONFIRMED` (`usePatientProfileUrlState`) |
| **Visits Tab** | Encounters Data Table | All viewports | Tab active (`tab=visits`) | Filtered `visits` joined with `visit_services` | — (Read-only tabular render) | `MOCK` — client-side `.filter()` over `MOCK_VISITS_V2` |
| **Packages Tab** | Packages Grid | Stacked ($< 1280\text{px}$) / 2-Col ($\ge 1280\text{px}$) | Tab active (`tab=packages`) | `packages` (filtered by `patientId`) | Hover/Click: Popover day inspection | `MOCK` — Catalog-phase, deferred from MVP |
| **Billings Tab** | Invoice History List | All viewports | Tab active (`tab=billings`) | `invoices` (filtered by `patientId`) | — (Read-only list) | `MOCK` — 🟡 Immature placeholder list |

---

## Section 2: Component-by-Component Deep Dive

### 1. Responsive Shell & Layout Switcher (`index.tsx`)
*Screenshots 1, 2 vs. Screenshot 3*

- **Current State:**
  - **Desktop ($\ge 1024\text{px}$):** Renders a side-by-side flex container (`flex lg:flex-row`). The left column (`PassportPanel`, `w-72 shrink-0`) stays fixed while the right column (`ClinicalHistoryPanel`, `flex-1 min-w-0`) handles clinical tabs and scrolls independently.
  - **Responsive Mode ($< 1024\text{px}$):** The desktop sidebar collapses. A mobile top tab bar appears with `← Back` and segmented buttons: `Profile` vs `History`. Toggling renders either `PassportPanel` (Screenshot 1) or `ClinicalHistoryPanel` (Screenshot 2) full-width.
- **Intended Behavior:** Provide responsive ergonomics for receptionists and clinicians using iPads, Android tablets, or laptops in treatment bays.
- **Backend Interaction Points:**
  - Route param `:id` triggers profile query:
    ```sql
    SELECT * FROM patients WHERE id = :id;
    ```
  - Parallel queries populate child panels: `patient_alerts`, `complaint_courses`, `visits`, `packages`, `invoices`.
- **Frontend Assumptions:**
  - **`CONFIRMED` (Ref: ADR-PP-08, ADR-PP-10):** Safe navigation fallback implemented in `handleBack`: checks `window.history.state.idx > 0` before calling `navigate(-1)`, otherwise falls back to root `/`.
  - **`CONFIRMED` (Ref: ADR-PP-11):** Contains an artificial 20% network error simulation inside `fetchPatientProfile` for UI error-state testing.

---

### 2. Passport Panel / Demographics (`PassportPanel.tsx`)
*Screenshot 1 & Screenshot 3 (Left Column)*

- **Current State:**
  - Avatar circle with initials (`PK`) or `photoUrl`.
  - Full legal name (`Priya Kapoor`) with subtitle `38 Yrs · Female`.
  - **Critical Alerts Card:** Red pill badge `Penicillin Allergy` inside a dedicated alert box.
  - **Demographic Rows:**
    - `MRN`: `MED-001` (monospace font)
    - `Blood Type`: `B+`
    - `Insurer`: `Star Health Insurance`
- **Backend Interaction Points:**
  - Demographics map to `patients.full_name`, `patients.date_of_birth` (derived age via `calculateAge`), `patients.gender`, `patients.mrn`.
  - Alerts map to `patient_alerts` (`patient_id = :id AND is_active = true`).
- **Frontend Assumptions & Schema Gaps:**
  - **`NEW` — Missing Schema Column: `Blood Type`:** `PassportPanel.tsx` (L70) renders `patient.bloodType`. In `supabase_migration.md` (v11), the `patients` table has **no `blood_type` or `blood_group` column**.
  - **`NEW` — Missing Schema Column: `Insurer`:** `PassportPanel.tsx` (L71) renders `patient.insurerName`. The `patients` table in `supabase_migration.md` has **no insurance or payer columns** (`insurer_name`, `policy_number`, etc.).
  - **`CONFIRMED` (Ref: ADR-PP-19):** `patient.alerts.map` uses composite natural key `${alert.type}-${alert.label}` to guarantee stable reconciliation.

---

### 3. Clinical History: Overview Tab (`OverviewTab.tsx`)
*Screenshot 2 & Screenshot 3 (Right Column)*

The Overview tab is composed of two primary sub-components: the **Vitals Grid** and the **Complaint History Timeline**.

#### A. The Vitals Grid (`VitalsGrid.tsx`) — 🔴 Domain Placeholder Warning
- **Current State:** 4 metric cards in a responsive grid (`grid-cols-2 sm:grid-cols-3`):
  1. `BLOOD PRESSURE`: `118/76 mmHg` (`↗ Normal` in green)
  2. `HEART RATE`: `78 bpm` (`↗ Normal` in green)
  3. `TEMPERATURE`: `98.4 °F` (`↗ Normal` in green)
  4. `OXYGEN SATURATION`: `98 %` (`↗ Normal` in green)
- **Domain Reality & Explicit Directive:**
  > **IMPORTANT:** As explicitly directed by clinic operations, **these vitals are pure design placeholders and NOT real clinical data.** In an outpatient physiotherapy clinic, routine hospital vital signs (BP, HR, Temp, SpO2) are almost never measured or logged.
- **Backend Schema Interaction:**
  - `supabase_migration.md` lines 409–424 contains a dedicated `patient_vitals` table (`type in ('BP', 'HR', 'TEMP', 'SPO2')`, `value`, `unit`, `trend`).
  - **Finding:** The backend schema includes a table specifically designed for these hospital vitals, while clinical operations confirms they are non-functional placeholders. See Decision 1.

#### B. Complaint History Timeline (`ClinicalTimeline.tsx`)
- **Current State:** Vertical timeline with chronological nodes connected by a subtle gray vertical track:
  - **Active Complaints (Green Dot + Emerald Left-Border Card):**
    - `Ankle Sprain` | Badge: `Active` (green) | `15 Jan 2024 → Ongoing` | `10 sessions so far` | Action: `View visits →`
    - `Cervical Spondylosis` | Badge: `Active` (green) | `15 Jan 2024 → Ongoing` | `9 sessions so far` | Action: `View visits →`
  - **Completed Complaints (Muted Node + Gray Card):**
    - `Plantar Fasciitis` | Badge: `Completed` (gray) | `1 Jun 2023 → 15 Jul 2023 · 12 sessions` | Action: `View visits →`
    - `Lower Back Pain` | Badge: `Completed` (gray) | `10 Jan 2023 → 28 Feb 2023 · 14 sessions` | Action: `View visits →`
- **Interactive Routing / Inter-Tab Navigation:**
  - Clicking any complaint card or its `View visits →` prompt triggers:
    ```ts
    goToVisits({ complaintId: course.id })
    ```
  - This utilizes the centralized `usePatientProfileUrlState` hook to set URL query parameters:
    `?tab=visits&complaint=<complaint_course_id>`.
  - The UI automatically switches to the `Visits` tab pre-filtered to the selected complaint's encounter history.

---

### 4. Clinical History: Visits Tab (`VisitsTab.tsx`)
*Screenshot 4: Visits Tab toolbar, open complaint dropdown, encounter table, procedure tags with itemized fee logic, and light-mode theme inversion.*

#### A. Dual-Axis Filter Toolbar
- **Current State:**
  - **Visit Type Segmented Control:** Fixed pill buttons: `All`, `Consult`, `Machine Only`.
  - **Complaint Selector (ShadCN Dropdown):** Renders `All complaints` plus the list of complaint courses with their status (e.g. `Ankle Sprain · Active`, `Cervical Spondylosis · Active`, `Plantar Fasciitis`, `Lower Back Pain`, `Tennis Elbow (Left)`, `Knee Pain (Right)`).
  - **Counter & Reset:** Shows match ratio (e.g. `10 of 18 visits`), revealing that `Ankle Sprain` is actively filtered (10 visits match). A `Clear filters` button appears whenever filters are active.
- **State Synchronization:**
  - Fully synchronized with URL query params: `?tab=visits&type=MACHINE_ONLY&complaint=CC-p01-01`.
- **Architectural Question (Client vs. Server Filtering):**
  - Today, filtering is performed client-side over the `visits` prop (`VisitsTab.tsx` L29–34). Evaluated for server pagination when visit volume exceeds 100 encounters. See Decision 5.

#### B. Fee Rules & Procedure Line-Item Display Logic
1. **Doctor Prescribed / Consultation Encounters (`visitType: 'CONSULTATION'`):**
   - Doctor's consultation/therapy fee covers standard machines (`IFT`, `UST`, `TENS`) $\rightarrow$ rendered as neutral gray pills **without individual prices** (`isCharged = false`).
   - Add-on specialized procedures (e.g. `Dry Needling ₹700`) are classified as `PREMIUM` with `isCharged = true` and display their fee explicitly in amber.
   - Grand Total = Consultation Fee (₹200) + Special Procedure (₹700) = **₹900** (Row 3, 2024-02-05).
2. **Machine-Only Encounters (`visitType: 'MACHINE_ONLY'`):**
   - No consultation fee (`consultation_fee = 0`). Every machine is billed at standalone clinic pricing and displays its individual price tag (`IFT ₹50`, `UST ₹70` $\rightarrow$ Total: **₹120**).

#### C. Observed Bugs in Screenshot 4
1. **🔴 Light-Mode Inversion Bug (Theme Leakage):** Page rendered in stark white theme (`bg-white`) while the dropdown was dark (`bg-zinc-900`). Theme toggle allowed light mode when spec mandates dark-mode strictly. See Decision 6.
2. **🟡 Complaint Course Dropdown Scope:** Dropdown renders all 6 historical records (including 2021–2022 courses `Tennis Elbow` and `Knee Pain`), while Overview timeline only displayed the top 4.
3. **🟡 Missing Visit Type Badge in Row 1:** The first table row (2024-02-11, `Ankle Sprain`, Total ₹450) has an empty/unrendered visit type cell.

---

### 5. Clinical History: Packages Tab (`PackagesTab.tsx` & `PackageCard.tsx`)
*Screenshots 5 & 6: Tablet stacked view vs. Desktop 2-column grid (`xl:grid-cols-2`), card anatomy, permanent dot grid, Sunday exclusion badges, and popovers.*

#### A. Card Anatomy & Visual Hierarchy (Ref: Variant 4 / ADR-PP-09)
Each package is rendered as a distinct ShadCN card featuring an accent status border on the left:
- **Status Left Border:** Blue for `Active` (`border-l-blue-500`), Muted Zinc for `Completed` (`border-l-zinc-300 dark:border-l-zinc-600`), Red for `Expired` (`border-l-rose-500`).
- **Header:**
  - `packageName`: e.g. `Chronic Back Pain — 20 Day Package`, `Ankle Rehab — 10 Day Package`, `IFT Therapy — 5 Day Package`.
  - `linkedComplaintName`: e.g. `Chronic Back Pain`, `Ankle Rehab`.
  - `excludeSundays` pill badge: When `true`, displays an amber pill badge `Excl. Sundays` (`bg-amber-500/10 text-amber-400`).
  - Status Badge: Top-right badge (`Active` in solid blue, `Completed` in secondary muted gray).
- **Primary Metric Row:**
  - Big bold countdown:
    - If `Active`: `6 Days Left` (prominent 3xl font).
    - If `Completed`: `10 Days Total` / `5 Days Total` (muted 3xl font).
  - Explicit Date Range: Compact side-by-side display with vertical divider:
    - `START`: `1 Jul 2026` / `1 Nov 2023` / `15 Jul 2023`.
    - `EXPIRES`: `20 Jul 2026` / `10 Nov 2023` / `19 Jul 2023`.
  - `AMOUNT PAID`: Right-aligned currency display (e.g. `₹8,000`, `₹4,500`, `₹2,200`).

#### B. Permanent Dot Grid & Attendance Tracking
- **Elapsed Counter Header:** Shows `14 OF 20 DAYS ELAPSED` (left) alongside colored breakdowns: `12 Attended` (blue) and `2 Missed` (red).
- **Dot Grid Matrix:** A horizontal wrap of individual micro-dots representing each scheduled session day:
  - `attended`: Filled solid blue (`bg-blue-500`).
  - `missed`: Filled solid red/rose (`bg-rose-500`).
  - `pending / upcoming`: Muted zinc dot (`bg-zinc-200 dark:bg-zinc-800`).
- **Accessible Screen Reader Summary:** Includes visually hidden text (`sr-only`):
  `"Attendance summary: 12 attended, 2 missed out of 20 total days."`
- **Interactive Day Inspection (Popover):**
  - Hovering or tapping any individual dot opens a compact ShadCN `Popover`:
    - Computes the true calendar date via `getDayDate(purchaseDate, idx, excludeSundays)`.
    - If `excludeSundays = true`, Sundays are automatically skipped during the date walk.
    - Popover displays: `2 Jul 2026 | Attended` or `14 Jul 2026 | Missed`.

#### C. Backend Schema Alignment & MVP Deferral Boundary
- **Schema Mapping (`supabase_migration.md` L737–761):**
  - Maps to `packages` table: `package_name`, `linked_complaint_id`, `linked_complaint_name`, `purchase_date`, `expiry_date`, `duration_days`, `exclude_sundays`, `attended_days`, `missed_days`, `amount_paid_in_paise`, `status`, `day_log jsonb`.
- **MVP Deferral Boundary (Ref: `study/business_logic/packages_logic/DEFERRAL.md`):**
  > **CONFIRMED:** The frontend implementation of the Packages Tab is a high-fidelity catalog-phase component using static mock data (`MOCK_PACKAGES`). As confirmed by client directives, **the entire Packages subsystem is formally deferred from the MVP backend wiring.** No live Supabase table, automatic session depletion triggers, or payment deductions will be wired in Phase 1.

---

### 6. Clinical History: Billings Tab (`BillingsTab.tsx`)
*Screenshot / Code Analysis: Minimalist invoice history ledger, premature placeholder status.*

#### A. Component State & Visual Representation
- **Current State:** A simple bordered card (`bg-white dark:bg-zinc-900/50`) containing a vertical list of invoice records:
  - Section Header: `INVOICE HISTORY` in 10px uppercase tracking.
  - Rows:
    - Left side: Formatted invoice amount in monospace bold (e.g. `₹2,500`, `₹1,800`, `₹500`) with invoice date below (`2024-02-11`, `2024-01-28`, `2024-01-15`).
    - Right side: Standard `StatusBadge` showing `Paid` (emerald), `Pending` (amber), or `Overdue` (rose).
- **Empty State:** High-fidelity empty container with `Receipt` icon and message: *"No Billing Records. There are no invoices or billing records for this patient yet."*

#### B. Immature / Placeholder Status & Gaps
As explicitly noted in operations feedback, this tab is **premature and not yet mature**:
1. **Missing `invoice_number`:** Does not render the human-readable identifier (e.g. `INV-2026-0001` or `INV-001`). Staff cannot cross-reference a paper receipt with this screen.
2. **Missing Payment Mode:** Does not disclose whether payment was collected via `Cash`, `UPI`, or `Card`.
3. **No Line-Item or Visit Association:** Does not indicate which visit, package, or complaint this invoice was billed for. Clicking a row does not expand to show itemized procedures or taxes.
4. **No Actionable Financial Controls:** No button to download/print a PDF receipt, issue a refund, or collect payment on `Pending`/`Overdue` balances.

---

## Section 3: Data Flow & State Map

| Interaction / Transition | Source Component | State Mechanism | DB Entity / Expression | Status |
|---|---|---|---|---|
| Load Patient Record | `PatientProfilePage` | URL `:id` via `useEffect` | `patients WHERE id = :id` | `NOT YET WIRED` (Mock Promise with 20% err) |
| Toggle Mobile Panel | `index.tsx` (Mobile switcher) | React Local State: `activePanel` | Client-only layout state | `CONFIRMED` |
| Switch Clinical Tab | `ClinicalHistoryPanel` | URL Param: `?tab=overview\|visits\|packages\|billings` | Client-only URL synchronization | `CONFIRMED` (`usePatientProfileUrlState`) |
| Render Vitals Cards | `VitalsGrid.tsx` | Props from `profile.vitals` | `patient_vitals` (**DUMMY PLACEHOLDER**) | `MOCK` — Marked for removal/redesign |
| Click Complaint Card | `ClinicalTimeline.tsx` | `goToVisits({ complaintId })` | Transitions URL to `?tab=visits&complaint=:id` | `CONFIRMED` |
| Filter by Visit Type | `VisitsTab.tsx` | Segmented button click | URL Param: `?type=CONSULTATION\|MACHINE_ONLY` | `CONFIRMED` |
| Filter by Complaint | `VisitsTab.tsx` | Select dropdown change | URL Param: `?complaint=CC-p01-01` | `CONFIRMED` |
| Inspect Package Dot | `PackageCard.tsx` | Hover/Tap on micro-dot | Computes date via `getDayDate()` | `MOCK` — Client-side Popover |
| Render Invoices List | `BillingsTab.tsx` | Props from `invoices` | `invoices WHERE patient_id = :id` | `MOCK` — Immature placeholder list |

---

## Section 4: Unresolved Design Decisions

### Decision 1 — How to handle the Vitals Placeholder vs. Physiotherapy Practice?
**Priority: HIGH** · **Tag: 🔴 Domain / Schema Alignment**
- Hospital vitals (`BP`, `HR`, `TEMP`, `SPO2`) are generic placeholders never collected in outpatient physiotherapy. The `patient_vitals` table in `supabase_migration.md` is dead schema for MVP. Either remove `VitalsGrid` or repurpose it into functional physio metrics (*VAS Pain Score 1–10*, *Active ROM degrees*).

### Decision 2 — Missing Schema Columns: Blood Type and Insurer
**Priority: MEDIUM** · **Tag: 🟡 Schema Reconciliation**
- `PassportPanel.tsx` renders `Blood Type` (`B+`) and `Insurer` (`Star Health Insurance`). Neither column exists on `patients` in `supabase_migration.md`. Retain as mock display fields until private insurance scope is decided.

### Decision 3 — Status Enum Mismatch: `'Resolved'` vs. `'Completed'`
**Priority: LOW** · **Tag: 🟡 Schema Constraint**
- UI renders `Completed` for closed complaint courses; PostgreSQL schema constraint requires `status IN ('Active', 'Resolved', 'Dormant')`. Standardize display adapter: map DB `'Resolved'` $\leftrightarrow$ UI `'Completed'`.

### Decision 4 — Session Counter Storage: Stored Column vs. Dynamic Count
**Priority: MEDIUM** · **Tag: 🟢 Performance & Integrity**
- Complaint cards display `X sessions so far`. For MVP (< 50 visits/patient), compute dynamically via `COUNT(*) FROM visits WHERE complaint_course_id = :id` to prevent counter-drift bugs.

### Decision 5 — Client-Side vs. Backend Filtering for Visits Tab
**Priority: MEDIUM** · **Tag: 🟡 Architecture & Scalability**
- Filtering across visit types and complaints is currently performed in memory. Retain client filtering for Phase 1 / MVP; migrate to server pagination only if individual patient visit volume exceeds 100 rows.

### Decision 6 — Enforce Dark Mode Strictly Across the Application
**Priority: HIGH** · **Tag: 🟡 UI & Design System Integrity**
- App leaked into light theme in Screenshot 4. Defect remediation: remove header theme toggle and hardcode `class="dark"` on root `<html>` element.

### Decision 7 — Packages Module Backend Deferral & Auto-Depletion
**Priority: DEFERRED FOR MVP** · **Tag: 🔴 Structural / Business Logic**
- Packages is 100% catalog-phase UI. When eventually wired in Phase 2, an architectural decision is required on whether visit logging automatically decrements `packages.attended_days` and appends to `day_log jsonb`, or requires explicit receptionist confirmation.

### Decision 8 — Billings Tab Maturation & Auditability
**Priority: MEDIUM for Phase 2** · **Tag: 🟡 UI & Financial Controls**
- Current `BillingsTab.tsx` is an immature placeholder omitting `invoice_number`, payment method, itemized procedure breakdown, and PDF download triggers. Prior to production billing go-live, this component must be rebuilt to reflect canonical invoice schema attributes.

---

## Section 5: Frontend ↔ Backend Divergences

### A. Confirmed in Code / Prior Audits
1. **Currency Paise Storage:** All frontend tabs (`VisitsTab`, `PackageCard`, `BillingsTab`) display integer Rupees (`₹8,000`, `₹120`), whereas `supabase_migration.md` stores `amount_paid_in_paise` and `amount_in_paise` as integer paise ($8,000 \times 100 = 800,000$).
2. **Missing Schema Columns:** `blood_type` and `insurer_name` rendered in `PassportPanel.tsx` do not exist in `patients` table.
3. **Status Terminology Drift:** UI uses `Completed` for finished complaints; PostgreSQL schema constraint requires `Resolved`.
4. **Artificial Error Simulation:** `fetchPatientProfile` in `index.tsx` includes a `Math.random() < 0.2` simulated network failure (ADR-PP-11).

### B. Newly Surfaced by this Pass
1. **Vitals Mismatch with Physiotherapy Domain:** `patient_vitals` table in backend schema conflicts with clinic workflow where vital signs are non-functional placeholders.
2. **Packages Backend Deferral:** Full `packages` schema table exists in migration v11, but the feature is deferred from MVP wiring.
3. **Billings Tab Missing Canonical Identifiers:** `BillingsTab.tsx` completely omits `invoice_number` (`INV-2026-XXXX`) and payment method.
4. **Day-Log Popover Date Alignment:** `PackageCard.tsx` contains custom client-side date-stepping logic (`getDayDate`) that skips Sundays when `excludeSundays = true`. In production, this must match backend date math to prevent popover date drift.

---

## Closing Summary

The Patient Profile module is now completely documented across all four clinical tabs:
- **Overview Tab:** High-level summary connecting the demographics passport, a placeholder vitals grid, and an interactive complaint history timeline.
- **Visits Tab:** Granular encounter ledger with dual-axis filtering, visually confirming the clinic's core fee logic (consultation bundling vs. standalone machine tariffs).
- **Packages Tab:** High-fidelity attendance tracking cards featuring permanent dot grids and Sunday-skipping date logic, operating as a self-contained catalog-phase component deferred from MVP.
- **Billings Tab:** Immature placeholder list of invoices marked for future expansion to include canonical invoice numbers, payment methods, and PDF receipt downloads.

---

## Sources
- `src/features/patient/PatientProfile/index.tsx` — layout orchestration and simulated fetch
- `src/features/patient/PatientProfile/PassportPanel.tsx` — demographic sidebar, alerts, and insurance layout
- `src/features/patient/PatientProfile/tabs/OverviewTab.tsx` & `VitalsGrid.tsx` — overview tab structure and vitals placeholder
- `src/features/patient/PatientProfile/tabs/ClinicalTimeline.tsx` — complaint history timeline, status badges, and visit navigation
- `src/features/patient/PatientProfile/tabs/VisitsTab.tsx` & `components/ServiceTag.tsx` — visits table, filter toolbar, and pricing engine
- `src/features/patient/PatientProfile/tabs/PackagesTab.tsx` & `components/PackageCard.tsx` — package cards and permanent dot grid
- `src/features/patient/PatientProfile/tabs/BillingsTab.tsx` — invoice history placeholder list
- `src/features/patient/PatientProfile/hooks/usePatientProfileUrlState.ts` — typed URL parameter management
- `supabase_migration.md` (v11) — canonical schemas for `patients`, `patient_vitals`, `visits`, `packages`, and `invoices`
- `study/business_logic/packages_logic/DEFERRAL.md` — confirmation of packages deferral from MVP
- Screenshots 1–6: Patient Profile (Responsive Tablet `Profile`, Responsive Tablet `History/Overview`, Desktop Master-Detail, Visits Tab, Packages Tab Stacked, and Packages Tab 2-Column Grid)
