# DASHBOARD.md — Frontend Dashboard Documentation (Current State vs. Target Operations Mockup)

**Scope:** Current `/dashboard` (KPI row + Patient Profiles card grid navigating to `/patient/:id`) vs. Target Operations Dashboard Mockup (6-card operational command center: Revenue Trend, Key Metrics, Clinic Health, Action Items, Referral Sources, Recent Patients). This covers dashboard analytics, operational KPIs, patient directory entry points, and clinic-level aggregation rules. It does **not** cover the deep clinical record inside `/patient/:id` (documented in patient profile specs) or the 5-step visit stepper (documented in `FRONTEND_WORKFLOW.md`).

**Basis:** 2 screenshots provided (Sept 3 2026), cross-referenced against `supabase_migration.md` (v11, canonical), `src/pages/dashboard.tsx`, `src/pages/PatientCard.tsx`, `src/data/mock_data.ts`, `FRONTEND_WORKFLOW.md`, `schema_reconciliation_audit.md`, and `README.md`.
- **Screenshot 1 (Current Live Implementation):** `/dashboard` with 4 KPI summary cards (`Total Patients: 4`, `Total Visits: 50`, `Revenue Collected: ₹11,300`, `Pending / Overdue: 2`) and a 4-card "PATIENT PROFILES" grid (`Priya Kapoor`, `Vikram Singh`, `Anjali Devi`, `Aarav Mehta`).
- **Screenshot 2 (Target Operations Mockup — Unfinished / Non-functional):** Complete redesign into a 6-widget operational overview featuring a 30-day Revenue Trend histogram (`₹1,27,786`, `+12%`), Key Metrics, Clinic Health (`No-show Rate: 4.2%`, `New vs Returning: 1:4`, `Active Treatment Plans: 28`), Action Items (`Pending Payment: Anjali Devi`), Referral Sources distribution, and a compact Recent Patients feed with a "View all" trigger.

**Wiring status:** 100% mock data in the current implementation (`MOCK_PATIENT_PROFILES`, `MOCK_VISITS_V2`, `MOCK_INVOICES`). The Target Operations Mockup is catalog-phase / static design only — not established, non-functional, and incomplete in data threading.

**Legend:** `MOCK` = catalog-phase, hardcoded/simulated · `NOT YET WIRED` = no live Supabase call exists · `PLANNED` = named in mockup, not built · `CONFIRMED` = matches code/schema audit · `NEW` = surfaced by this pass · 🔴/🟡/🟢 = structural / safety-net / performance tagging convention.

---

## Section 1: Dashboard Overview Matrix

| Mode / Screen | Widget / Section | Initiated By | Reads (SELECT / Aggregations) | Writes (INSERT / UPDATE) | Status |
|---|---|---|---|---|---|
| **Current Live** | Top KPI Row (4 cards) | Page Mount | `patients` (count), `visits` (count), `invoices` (sum of `amount_in_paise` where `Paid`, count where `Pending`/`Overdue`) | — (Read-only) | `MOCK` — client-side `.reduce()` / `.filter()` over in-memory arrays |
| **Current Live** | Patient Profiles Grid | Page Mount | `patients` (profile attributes), `visits` (per-patient count), `invoices` (per-patient unpaid count), `patient_alerts` (active badges) | — (Read-only) | `MOCK` — derived via O(1) hash maps in `dashboard.tsx` |
| **Current Live** | Patient Card Click | User click | — | Local navigation: `navigate('/patient/:id')` | `CONFIRMED` — pure client routing |
| **Target Mockup** | Revenue Trend (30d) | Page Mount | `invoices` (grouped by `date` / `created_at::date` for 30d; prior 30d window for % delta) | — (Read-only) | `PLANNED` / `MOCK` — hardcoded ₹1,27,786 + chart |
| **Target Mockup** | Key Metrics Panel | Page Mount | `invoices` (sum of paid, count of pending), `patients` (count registered in last 30d) | — (Read-only) | `PLANNED` / `MOCK` — ₹11,300, 3 new, 2 pending |
| **Target Mockup** | Clinic Health Panel | Page Mount | `appointments` (**NON-EXISTENT TABLE** for no-show), `visits` (new vs returning ratio), `complaint_courses` (count where `Active`) | — (Read-only) | `PLANNED` / `MOCK` — contains 🔴 ghost metric |
| **Target Mockup** | Action Items Panel | Page Mount | `invoices` (filter `payment_status in ('Pending', 'Overdue')`, join `patients`) | Action resolution (e.g. collect payment) | `PLANNED` / `MOCK` — 1 item rendered out of 2 pending |
| **Target Mockup** | Referral Sources | Page Mount | `patients.referral_mode` (grouped count) | — (Read-only) | `PLANNED` / `MOCK` — distribution bars |
| **Target Mockup** | Recent Patients Widget | Page Mount | `patients` + `visits` (top 3 by `last_visit_at` descending) | Navigation: "View all" link | `PLANNED` / `MOCK` — 3 rows shown |

---

## Section 2: Component-by-Component Deep Dive

### Part A: Current Live Implementation (`/dashboard`)
*Screenshot 1: "Patient Profiles", 4 KPI cards, 4 patient cards with alert badges, visit counters, MRNs, and unpaid badges.*

#### 1. Header & Navigation
- **Current State:** Global top navbar with tabs: `Visit Workflow`, `Dashboard` (active, underlined), `Reports`, `Roadmap`. Page title is prominently displayed as **"Patient Profiles"** with subtitle *"Click any patient card to open their full clinical record."*
- **Intended Behavior:** Serve as the central jumping-off point to individual patient records.
- **Frontend Assumptions:**
  - The tab is labeled `Dashboard`, but the page header declares itself `Patient Profiles`. The page functions as an expanded directory rather than an operational dashboard.

#### 2. Top Summary KPI Row
- **Current State:** 4 metric cards:
  1. `Total Patients`: **4** (`Users` icon)
  2. `Total Visits`: **50** (`Activity` icon)
  3. `Revenue Collected`: **₹11,300** (`CreditCard` icon)
  4. `Pending / Overdue`: **2** (`AlertTriangle` icon)
- **Backend Interaction Points:**
  - `Total Patients`: `COUNT(*) FROM patients WHERE owner_id = :owner_id` (or clinic-filtered via `patient_clinic_access`).
  - `Total Visits`: `COUNT(*) FROM visits WHERE clinic_id = :clinic_id`.
  - `Revenue Collected`: `SUM(amount_in_paise) / 100 FROM invoices WHERE clinic_id = :clinic_id AND payment_status = 'Paid'`.
  - `Pending / Overdue`: `COUNT(*) FROM invoices WHERE clinic_id = :clinic_id AND payment_status IN ('Pending', 'Overdue')`.
- **Frontend Assumptions:**
  - **`CONFIRMED`** — Currency values in `MOCK_INVOICES` are stored as decimal Rupees (`amount: 550`), whereas `supabase_migration.md` strictly mandates integer paise (`invoices.amount_in_paise`). The frontend formatting `₹${totalRevenue.toLocaleString('en-IN')}` directly formats raw mock numbers without dividing by 100.
  - **`CONFIRMED`** — Computed via `useMemo` inside `dashboard.tsx` (L33–49) over full mock arrays. In production, fetching all visits and invoices client-side to count length is unscalable ($O(N)$ network payload); needs server-side RPC or aggregate views.

#### 3. Patient Profiles Card Grid
- **Current State:** 4 cards rendered via `PatientCard.tsx` in a responsive grid (`sm:grid-cols-2 lg:grid-cols-3`):
  - **Priya Kapoor (MED-001):** 38 Yrs · Female · B+ · Badge: `Penicillin Allergy` (red outline) · Footer: `18 visits` · `MED-001`.
  - **Vikram Singh (MED-002):** 41 Yrs · Male · O+ · Badge: `Post-Op Fall Risk` (amber outline) · Footer: `19 visits` · `MED-002` · `1 unpaid` (amber text).
  - **Anjali Devi (MED-003):** 35 Yrs · Female · A- · Badges: `NSAIDs Allergy` (red outline), `Diabetic — Monitor Glucose` (neutral outline) · Footer: `13 visits` · `MED-003` · `1 unpaid` (amber text).
  - **Aarav Mehta (MED-004):** 31 Yrs · Male · O+ · No badges · Footer: `0 visits` · `MED-004`.
- **Backend Interaction Points:**
  - Demographics map to `patients.full_name`, `patients.date_of_birth` (derived age via `calculateAge`), `patients.gender`, `patients.mrn`.
  - Blood type (`B+`, `O+`, `A-`): `patients` table in `supabase_migration.md` (v11) **does NOT have a `blood_group` or `blood_type` column**! See Decision 6.
  - Alerts map to `patient_alerts` (`type IN ('ALLERGY', 'FALL_RISK', 'DNR', 'OTHER')`, `label`, `is_active = true`).
  - Visit counts: Derived from `visits` grouped by `patient_id`.
  - Unpaid invoices: Derived from `invoices` where `payment_status IN ('Pending', 'Overdue')` grouped by `patient_id`.
- **Frontend Assumptions:**
  - **`CONFIRMED` (Ref: ADR-PP-20, ADR-PP-22):** `dashboard.tsx` builds hashmap lookups (`visitsCountLookup`, `unpaidInvoicesCountLookup`) to feed primitive props into `PatientCard` wrapped in `React.memo`.
  - Clicking any card executes `navigate('/patient/${patient.id}')`.

---

### Part B: Target Operations Mockup (`/dashboard` Redesign)
*Screenshot 2: 6 operational widgets: Revenue Trend, Key Metrics, Clinic Health, Action Items, Referral Sources, Recent Patients.*

#### 1. Revenue Trend (Widget 1)
- **Current Mockup State:**
  - Header: `Revenue Trend` | `Last 30 Days`.
  - Value: `₹1,27,786` with delta indicator `+12% vs prior period` (green badge).
  - Visual: 30-day vertical histogram / bar chart (indigo bars showing day-to-day revenue variance).
- **Backend Interaction Points:**
  - Requires daily time-series aggregation:
    ```sql
    SELECT date, SUM(amount_in_paise) as daily_revenue
    FROM invoices
    WHERE clinic_id = :clinic_id
      AND payment_status = 'Paid'
      AND date >= CURRENT_DATE - INTERVAL '30 days'
    GROUP BY date ORDER BY date ASC;
    ```
  - Prior period delta calculation requires querying the window between `CURRENT_DATE - INTERVAL '60 days'` and `CURRENT_DATE - INTERVAL '30 days'`.
- **Frontend Assumptions:**
  - **`NEW` — Mock data contradiction:** Screenshot 1's lifetime revenue is ₹11,300. In Screenshot 2, the 30-day trend alone is ₹1,27,786, while the neighboring Key Metrics card reports `Total Revenue: ₹11,300`. This confirms the mockup is an unintegrated static wireframe with arbitrary dummy values.
  - Requires a charting dependency (e.g. Recharts, Visx, or pure SVG bars).

#### 2. Key Metrics (Widget 2)
- **Current Mockup State:**
  - `Total Revenue`: **₹11,300** (white, bold)
  - `New Patients`: **3** (green text)
  - `Pending Invoices`: **2** (amber text)
- **Backend Interaction Points:**
  - `Total Revenue`: Same as Current Live KPI (`SUM(amount_in_paise) WHERE payment_status = 'Paid'`).
  - `New Patients`: `COUNT(*) FROM patients WHERE created_at >= CURRENT_DATE - INTERVAL '30 days'` (or first visit in 30d).
  - `Pending Invoices`: Matches Current Live KPI (`COUNT(*) WHERE payment_status IN ('Pending', 'Overdue')`).
- **Frontend Assumptions:**
  - **`NEW` — Dropped KPI:** The live screen's `Total Visits: 50` has been removed from this panel, replaced by `New Patients: 3`. Total visit volume is now only indirectly implied.

#### 3. Clinic Health (Widget 3)
- **Current Mockup State:**
  - `No-show Rate`: **4.2%** with badge `Healthy` (green pill)
  - `New vs Returning`: **1 : 4** (`Past 30d` tag, right-aligned)
  - `Active Treatment Plans`: **28**
- **Backend Interaction Points:**
  - `Active Treatment Plans`: Maps cleanly to `SELECT COUNT(*) FROM complaint_courses WHERE status = 'Active' AND clinic_id = :clinic_id`.
  - `New vs Returning`: Computable from `visits` by evaluating if `visit_count == 1` vs `visit_count > 1` within the 30-day window.
  - `No-show Rate`: **NO SCHEMA BACKING.** The v11 schema contains no appointment scheduling, booking calendar, or session attendance tracking. See Decision 1 (🔴 Blocker).
- **Frontend Assumptions:**
  - **`NEW` — Ghost Metric:** "No-show Rate: 4.2%" assumes an appointment scheduling system exists to calculate `cancelled_or_no_show / total_scheduled`. `supabase_migration.md` operates strictly on an encounter-logging model where visits only exist once completed.

#### 4. Action Items (Widget 4)
- **Current Mockup State:**
  - Header: Warning icon + `Action Items` + notification badge `1` (red circle).
  - Item Row: Credit card icon (red) + `Pending Payment` + `Anjali Devi • ₹1200`.
- **Backend Interaction Points:**
  - `SELECT i.id, i.amount_in_paise, p.full_name FROM invoices i JOIN patients p ON i.patient_id = p.id WHERE i.clinic_id = :clinic_id AND i.payment_status IN ('Pending', 'Overdue') ORDER BY i.created_at ASC`.
- **Frontend Assumptions:**
  - **`NEW` — Selective / Truncated Rendering:** The Key Metrics card clearly reports `Pending Invoices: 2`, and Screenshot 1 proves both Vikram Singh (`MED-002`) and Anjali Devi (`MED-003`) have unpaid invoices. However, Action Items only displays **1** item (Anjali Devi • ₹1200). Vikram Singh's pending payment is completely missing.
  - The UI does not explain the filter criteria: Is it capped at 1 item? Is it filtered by amount? Or did the designer simply hardcode a single mock row?

#### 5. Referral Sources (Widget 5)
- **Current Mockup State:**
  - Horizontal stacked/progress bars with counts:
    - `Walk-in`: **12** (blue/purple bar)
    - `Google`: **8** (teal bar)
    - `Doctor Referral`: **4** (orange bar)
    - `Word of Mouth`: **2** (blue bar)
  - Total represented: 26 patients.
- **Backend Interaction Points:**
  - Maps to `patients.referral_mode`.
- **Frontend Assumptions:**
  - **`NEW` — Enum Mismatch:** The UI displays `Word of Mouth`, but `supabase_migration.md` L248 defines the CHECK constraint as:
    `check (referral_mode in ('WALKIN', 'GOOGLE', 'DOCTOR', 'FRIEND_FAMILY', 'CAMP', 'OTHER'))`.
    The UI label "Word of Mouth" maps to the database value `'FRIEND_FAMILY'`.
  - **`NEW` — Tenancy Scope Divergence:** `patients` is an `owner_id`-scoped table with no `clinic_id`. If a multi-branch clinic receptionist views this widget, does it calculate referral sources across the entire chain or filter by patients who had visits at *this* specific branch?

#### 6. Recent Patients Feed (Widget 6)
- **Current Mockup State:**
  - Header: `Recent Patients` (`Users` icon) + top-right interactive link: `View all`.
  - Rows (compact layout with avatar, full name, MRN):
    1. `PK` | **Priya Kapoor** | `MED-001`
    2. `VS` | **Vikram Singh** | `MED-002`
    3. `AD` | **Anjali Devi** | `MED-003`
- **Backend Interaction Points:**
  - `SELECT p.id, p.full_name, p.mrn, p.last_visit_at FROM patients p JOIN patient_clinic_access pca ON p.id = pca.patient_id WHERE pca.clinic_id = :clinic_id ORDER BY p.last_visit_at DESC LIMIT 3;`
- **Frontend Assumptions:**
  - **`NEW` — IA Demotion of Patient Profiles:** The current live `/dashboard` is 100% dedicated to Patient Profiles. In this mockup, the full directory is replaced by a 3-row teaser.
  - Clicking `View all` must navigate to a dedicated `/patients` route or swap the dashboard view.

---

## Section 3: Data Flow & Aggregation Map

| Moment / Widget | Source DB Table(s) | Aggregation / Query Type | Supported by Index? | Tenant Scope | Wiring Status |
|---|---|---|---|---|---|
| Top KPI: Total Patients | `patients` + `patient_clinic_access` | `COUNT(*)` | Yes (`idx_pca_clinic_id`) | Clinic-scoped via access table | `NOT YET WIRED` |
| Top KPI: Total Visits | `visits` | `COUNT(*)` | Yes (`idx_visits_clinic_id`) | Clinic-scoped | `NOT YET WIRED` |
| Top KPI: Total Revenue | `invoices` | `SUM(amount_in_paise) WHERE payment_status = 'Paid'` | Partial (`idx_invoices_clinic_id`) — needs composite index | Clinic-scoped | `NOT YET WIRED` |
| Top KPI: Pending Invoices | `invoices` | `COUNT(*) WHERE payment_status IN ('Pending', 'Overdue')` | Partial — needs composite index | Clinic-scoped | `NOT YET WIRED` |
| Patient Profiles Grid | `patients`, `patient_alerts`, `visits`, `invoices` | Multi-table join or composite RPC | Yes (`idx_patients_owner_id`, `idx_patient_alerts_patient_id`) | Clinic-scoped | `NOT YET WIRED` |
| Revenue Trend (30d) | `invoices` | `GROUP BY date, SUM(amount_in_paise)` | Yes (`idx_invoices_date`, `idx_invoices_clinic_id`) | Clinic-scoped | `PLANNED` |
| Trend Delta % | `invoices` | 2-window comparison (`0-30d` vs `31-60d`) | Yes | Clinic-scoped | `PLANNED` |
| Clinic Health: No-show Rate | **NONE** (`appointments` missing) | **CANNOT EXECUTE** | **NO** | N/A | **IMPOSSIBLE UNDER V11 SCHEMA** |
| Clinic Health: New vs Returning | `visits` + `patients` | Partitioned window count on visits | Yes | Clinic-scoped | `PLANNED` |
| Clinic Health: Active Plans | `complaint_courses` | `COUNT(*) WHERE status = 'Active'` | Yes (`idx_complaint_courses_status`) | Clinic-scoped | `PLANNED` |
| Action Items List | `invoices` JOIN `patients` | `SELECT ... WHERE payment_status != 'Paid' LIMIT N` | Yes | Clinic-scoped | `PLANNED` |
| Referral Sources Breakdown | `patients` JOIN `patient_clinic_access` | `GROUP BY referral_mode, COUNT(*)` | No index on `referral_mode` | Chain vs Clinic ambiguous | `PLANNED` |
| Recent Patients Feed | `patients` + `patient_clinic_access` | `ORDER BY last_visit_at DESC LIMIT 3` | Yes (`idx_patients_last_visit_at`) | Clinic-scoped | `PLANNED` |

---

## Section 4: Unresolved Design Decisions

### Decision 1 — How is "No-show Rate" supported when appointments do not exist?
**Priority: BLOCKER for Clinic Health Widget**  
**Tag: 🔴 Structural**

- **Question:** The Clinic Health widget prominently displays `No-show Rate: 4.2% (Healthy)`. How can this be computed when neither the frontend nor backend possesses an appointment scheduling model?
- **What the frontend currently implies:** The mockup presents No-show Rate as an executive KPI alongside active treatment plans, assuming an underlying booking calendar tracks scheduled vs. attended visits.
- **What the backend needs to support it:** As documented in `schema_cross_reference.md` (L258) and `supabase_migration.md`, appointment scheduling is deferred to "Epic #15". The v11 schema is strictly encounter-based: rows in `visits` are created after or during the session. There is no `scheduled_at`, `status = 'No-Show'`, or cancellation log anywhere in PostgreSQL.
- **Leaning / Options:**
  - **Option A (Drop the Metric):** Remove "No-show Rate" from the MVP dashboard redesign and replace it with a metric directly derivable from existing schema (e.g. `Average Session Fee`, `Visits Completed Today`, or `Retention Rate`).
  - **Option B (Fast-track Epic #15 / Appointments Migration):** Author a new `v12` migration adding an `appointments` table (`clinic_id`, `patient_id`, `scheduled_start`, `scheduled_end`, `status check in ('Booked', 'Completed', 'Cancelled', 'No-Show')`). This is a major structural expansion.
  - **Recommendation:** **Option A for MVP.** Do not block dashboard delivery on an entire scheduling subsystem. Display real operational throughput metrics instead.

---

### Decision 2 — Information Architecture: What happens to the Patient Profiles Directory?
**Priority: HIGH**  
**Tag: 🟡 Information Architecture**

- **Question:** The current live `/dashboard` is titled "Patient Profiles" and renders the complete patient card roster. The target mockup replaces this roster with executive widgets and reduces patient access to a 3-item "Recent Patients" list with a "View all" link. Where does the full patient directory live?
- **What the frontend currently implies:** The target mockup assumes `/dashboard` is an operational command center and that Patient Profiles will be moved to its own dedicated route (e.g. `/patients`).
- **What the backend needs to know:** None; this is a client routing and page layout decision.
- **Leaning:** Formalize a dedicated `/patients` route that houses the rich `PatientCard` grid (with search, allergy filters, and pagination), leaving `/dashboard` as the operational dashboard depicted in Mockup 2. The `View all` link on the "Recent Patients" widget will route directly to `/patients`.

---

### Decision 3 — Action Items Triage: What qualifies as an "Action Item"?
**Priority: MEDIUM**  
**Tag: 🟡 Operational Logic**

- **Question:** In the target mockup, the Action Items widget displays a badge count of `1` and shows only `Pending Payment: Anjali Devi • ₹1200`. Why is Vikram Singh's pending invoice omitted, and what other events generate Action Items?
- **What the frontend currently implies:** Action Items appears to be a prioritized queue rather than a raw dump of all pending invoices.
- **What the backend needs to know:**
  - Is Action Items strictly an invoice collection queue, or does it include clinical alerts (e.g., patient flagged `FALL_RISK`, treatment plan expiring, package balance exhausted)?
  - What determines ranking? Age of debt, amount, or explicit follow-up date?
- **Leaning:** If Action Items is an alert hub, define an RPC or view `clinic_action_items` that unions:
  1. Overdue/Pending invoices older than $X$ days.
  2. Patients with `status = 'Active'` treatment plans who haven't visited in $> 14$ days (drop-off risk).
  3. Exhausted package balances.

---

### Decision 4 — Dashboard Aggregation Strategy: Client-side vs. SQL Views / RPCs
**Priority: BLOCKER before production wiring**  
**Tag: 🟢 Performance / Scalability**

- **Question:** The current live `dashboard.tsx` fetches all raw mock records into memory and computes counts/sums via JavaScript `.reduce()` and `.filter()`. How should the 30-day Revenue Trend, Referral breakdowns, and Health metrics be computed in production?
- **What the frontend currently implies:** The frontend is written assuming small mock arrays where client-side mapping is instantaneous.
- **What the backend needs to know:** On a live clinic database with 5,000+ visits and invoices, downloading all rows to compute a 30-day chart causes massive bandwidth consumption and slow renders.
- **Leaning:** Implement a dedicated PostgreSQL RPC or materialized view `get_clinic_dashboard_metrics(p_clinic_id uuid, p_start_date date, p_end_date date)` that returns:
  - KPI counts (`total_revenue`, `new_patients`, `pending_invoices`, `active_plans`).
  - Pre-aggregated 30-day daily revenue array for the chart.
  - Referral source distribution counts.
  This reduces 6 separate table queries into a single atomic, indexed roundtrip (< 30ms).

---

### Decision 5 — Tenant Scoping of Referral Sources & Patient Counts
**Priority: HIGH**  
**Tag: 🔴 Structural / Security**

- **Question:** `patients` is scoped to `owner_id` (chain-level tenant), while `visits` and `invoices` are scoped to `clinic_id` (branch-level). In the target mockup's "Referral Sources" and "New Patients" widgets, should the metrics reflect chain-wide numbers or only patients associated with the viewing receptionist's branch?
- **What the frontend currently implies:** The dashboard is viewed within a clinic branch context, implying all numbers reflect local branch operations.
- **What the backend needs to know:** `patients` rows do not store a `clinic_id`. To scope referral counts to Branch B, the query must join through `patient_clinic_access`:
  ```sql
  SELECT p.referral_mode, COUNT(*)
  FROM patients p
  JOIN patient_clinic_access pca ON p.id = pca.patient_id
  WHERE pca.clinic_id = :clinic_id
  GROUP BY p.referral_mode;
  ```
  If a patient visited Branch A first and then Branch B, they exist in `patient_clinic_access` for both. This means chain totals and branch sums may differ.
- **Leaning:** Enforce strict branch scoping via `patient_clinic_access` for all dashboard widgets to maintain multi-branch data isolation.

---

### Decision 6 — Blood Type / Demographics Absence in Database Schema
**Priority: LOW**  
**Tag: 🟡 Data Integrity**

- **Question:** Current live `PatientCard.tsx` renders blood types (e.g. `B+`, `O+`, `A-` in `Priya Kapoor`, `Vikram Singh`, `Anjali Devi`). `patients` in `supabase_migration.md` has no `blood_group` column.
- **What the frontend currently implies:** Blood type is treated as a core demographic attribute.
- **What the backend needs to know:** If blood type is a clinical requirement, `patients` requires an `ALTER TABLE patients ADD COLUMN blood_group text check (blood_group in ('A+', 'A-', 'B+', 'B-', 'AB+', 'AB-', 'O+', 'O-'));`. Alternatively, it must be stored inside `clinical_notes` or `patient_vitals`.
- **Leaning:** Add `blood_group` to `patients` via migration or cleanly remove it from the patient card subtitle.

---

## Section 5: Frontend ↔ Backend Divergences

### A. Confirmed in Code / Existing Audits
1. **Rupees vs. Paise Currency Storage:** `dashboard.tsx` (L38) sums raw numbers (`₹11,300`), whereas `invoices.amount_in_paise` is stored as an integer in paise ($11,300 \times 100 = 1,130,000$).
2. **Client-side Array Aggregation:** `dashboard.tsx` computes all dashboard KPIs via client-side `.filter()` and `.reduce()` across all entities. Unviable for live Supabase connection.
3. **Blood Group Missing in Schema:** `PatientCard.tsx` displays `bloodType`, but `patients` schema has no matching column.
4. **Referral Mode Enum Drift:** Mockup displays `"Word of Mouth"`, while `patients.referral_mode` constraint enforces `'FRIEND_FAMILY'`.

### B. Newly Surfaced by Target Dashboard Mockup (Screenshot 2)
1. **No-show Rate has No DB Backing:** `No-show Rate: 4.2%` is a ghost metric requiring an appointment scheduling table that does not exist in the v11 schema (🔴 Blocker).
2. **Mock Data Discrepancy:** 30-day Revenue Trend shows `₹1,27,786`, but the neighboring Key Metrics card and live dashboard report `Total Revenue: ₹11,300`.
3. **Action Items Incomplete Filter:** Action Items shows badge `1` and lists only Anjali Devi (`₹1200`), omitting Vikram Singh despite both having unpaid invoices in `MOCK_INVOICES`.
4. **Information Architecture Reorganization:** The live `/dashboard` is purely a patient roster. The target mockup converts `/dashboard` to an operational overview, necessitating the creation of a separate `/patients` route for the full roster.
5. **Cross-Branch Scoping for Referral Sources:** `patients.referral_mode` is stored on a chain-level table; calculating branch-specific referral attribution requires joining through `patient_clinic_access`.

---

## Closing Summary

The current `/dashboard` is effectively a Patient Directory with summary KPI headers, while the target mockup evolves the view into an executive Operations Command Center. 

To bridge the gap between the two:
1. **Scope Resolution:** Recognize that the target mockup is an unfinished concept wireframe. The "No-show Rate" metric must either be removed for MVP or explicitly backed by a new appointments migration (Decision 1).
2. **Information Architecture Split:** Extract the current `PatientCard` grid from `/dashboard` into a dedicated `/patients` directory route, allowing the operational dashboard to link directly to it via "View all" (Decision 2).
3. **Backend Aggregation RPC:** Replace all client-side array aggregations with a single, clinic-scoped PostgreSQL RPC (`get_clinic_dashboard_metrics`) to deliver performant 30-day revenue trends, key counts, and referral distributions in a single roundtrip.

---

## Sources
- `supabase_migration.md` (v11 canonical) — table schemas for `patients`, `visits`, `invoices`, `complaint_courses`, `patient_clinic_access`, `patient_alerts`
- `src/pages/dashboard.tsx` & `src/pages/PatientCard.tsx` — live implementation code, hooks, and memoization rules
- `src/data/mock_data.ts` — mock data entities (`MOCK_PATIENT_PROFILES`, `MOCK_VISITS_V2`, `MOCK_INVOICES`)
- `docs/FRONTEND_WORKFLOW.md` — workflow pattern, trigger sequence rules, and currency standards
- `docs/schema_cross_reference.md` — confirmation of Epic #15 (appointments table deferral)
- Screenshot 1: Current Live `/dashboard` (Sept 3 2026)
- Screenshot 2: Target Operations Dashboard Mockup (Sept 3 2026)
