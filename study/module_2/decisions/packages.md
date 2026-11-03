# Packages

Packages are required for MVP. This page incorporates the seven named package records through the **3 November 2026 Q18–Q40 questionnaire addendum** (S25–S31), alongside the earlier Module 2 decisions. Status labels below reflect the reconciled decisions from the handover (S25), addendum (S26), discussion patch (S27, PKG-PATCH-2026-11-02-01), voice session decision log (S28), pre-questionnaire decision log (S29), reconstructed final decision questionnaire (S30), and the Q18–Q40 questionnaire addendum (S31), as well as direct user clarifications.

## Established baseline

- Coverage is associated with a particular complaint/course. A covered complaint-treatment visit is a real clinical occurrence; bundled routine procedures do not each create a separate therapy charge (P03–P04).
- A package purchase is a billed fact distinct from a positive payment receipt and from treatment redemption. Package purchase and purchase invoice issuance are paired at checkout; positive payment receipts are separate transactions (P21, P30; S27 §7.1; S28 D04).
- **Snapshotting of Purchased Terms (`[USER—DECIDED]`, S28 D01):** A patient purchasing a package snapshots the covered quantity (e.g., 26 visits) and price. Subsequent catalog changes to prices or quantities never alter existing purchased entitlements or billed invoices.
- **Clinical Visit Immutability (`[USER—DECIDED]`, S28 D02):** Clinical visits are recorded by reception after therapy finishes and the patient reaches the counter. Recorded visits are immutable in MVP (no visit correction/deletion feature).
- **Genuine Visit Redemption (`[USER—WORKING DIRECTION]`, S28 D03):** Package consumption is strictly traceable to actual committed complaint-visits, not an arbitrary manual attendance counter.
- **Hard Ceiling on Long Offering:** 26 package-covered complaint-visits is a strict hard maximum. On visit 26/26, the package completes/exhausts automatically and cannot cover a 27th visit (S27 N01, S28 D01, `[OWNER—REPORTED—CONFIRMED]`).
- **One Covered Treatment per Day:** The long offering ordinarily covers at most one eligible complaint-treatment visit per calendar day. A second genuine treatment on the same date is charged at the ordinary complaint-visit rate (S27 N02, S28 D01, `[OWNER—REPORTED—CONFIRMED]`).
- **Review Clock Anchored to Package Start Date (`[USER—DECIDED]`, S30 Q01, Q02):** A package begins its review clock on its **package start date (`start_date`)**, which is distinct from its transaction/purchase date (`purchased_at`). If an existing package already covered a treatment for that complaint today, `start_date` is Day $T+1$ (tomorrow), and the review clock starts on $T+1$. If no same-complaint treatment was covered today, `start_date` is Day $T$ (today), and the review clock starts on $T$. This supersedes the earlier assumption that registration date is automatically Day 1.
- Covered complaint-visits retain real ₹0 invoice lines with verified package coverage. A covered-only invoice is allowed only with at least one real line and backend validation. After issuance, offer only Mark as paid; record no payment and zero collections. This established Q01 decision is not reopened by the source logs (R32–R33, R74).
- Examination/return and laboratory fees are independent patient-level charges, not complaint-visit redemptions. They do not consume complaint-package allowances or become covered solely by a complaint package (P05, P25). General examination/lapse-fee bands and their billing-only meaning remain governed by [examination fees](examination-fees.md), including S23.
- A package-covered visit and a waiver are different facts. Preserve the actual eligible visit/coverage link; neither a zero-valued line nor payment status by itself establishes a redemption.
- Individual standalone sessions are not presold in V1. Package coverage of standalone services is not settled by these logs.
- The user previously reported approximately 60–80% package use. This is a reported estimate, not verified data (P01).

## Offering and review directions through 3 November

- **Ten-visit offering:** Ten eligible covered complaint-treatment visits; exhaustion follows actual ten uses rather than elapsed days.
  - **Review counting convention (`[USER—DECIDED]`, S27 N09–N11, S30 Q01):** Package start date counts as **Day 1, even if Sunday**. Beginning the following date, **skip Sundays** when counting toward the ten review dates (Monday–Saturday count normally).
  - The **tenth counted date itself is the review date** (gentle reminder with info icon).
  - Starting the **following calendar date**, the package enters **Review Overdue** status.
  - Real treatments on **any Sunday** remain eligible and consume an allowance if the clinic operates; Sundays are skipped only for review-calendar arithmetic, not clinical eligibility.
- **Long offering:** Maximum 26 covered visits (`[OWNER—REPORTED—CONFIRMED]`, S27 N01, S28 D01).
  - **Review window:** Fixed **30 consecutive calendar dates**, Sundays included, beginning on package start date (`S30 Q01`).
  - **Soft review:** Day 30 is the review date. No automatic hard time expiry; unused visits can continue under review warning/discretion until visit 26 completes the package or until manually closed (`[USER—DECIDED]`, S27 N03, S28 D12).
  - **Sundays & Calendar opportunities:** A 30-consecutive-day window contains 4 or 5 Sundays, yielding 26 or 25 non-Sunday opportunities. 26 is a hard quantity ceiling, not a commercial guarantee of 26 non-Sunday slots in 30 days (S27 §2.3).
  - **Fixed 30-day review duration settled (`[USER—DECIDED]`, S29 D05, S30 baseline):** The long offering uses fixed 30 consecutive calendar dates including Sundays. Settled as the operative software rule; retiring the earlier open commercial wording comparison (`OPEN—OWNER`).
- **Configurable Daily Visit Cap (`[USER—DECIDED]`, S30 Q03):** Each purchased package has an operational `max_visits_per_day` setting.
  - **Default is 1** for all packages (both 10-visit short offering and 26-visit long offering).
  - Admin has full discretion to set or increase this limit (e.g. to 2 or more) **at purchase or anytime later**, including on 26-visit packages.
  - Checked dynamically at visit redemption against the package's remaining allowances and today's consumed treatments.
  - **Explicit MVP Concession:** Changes to this operational cap are not tracked in a dedicated audit log table in MVP; this blind spot is intentionally accepted as future scope.
- **No holiday ledger:** Avoid an automatic absence/leave/holiday ledger or automatic extension engine (P08, P15; S27 §2.3).
- **No arbitrary grace expiry buffer:** Defer any second hard-expiry window until empirical attendance data justifies it upon owner request (S27 N08, S28 D12).

## Review-day and overdue user experience

- **Two distinct states (`[USER—DECIDED]`, S27 §4):**
  1. **Review Day (Day 10 counted / Day 30 calendar):** Display a small, non-interruptive nudge: *"Review due today · Regular charges may apply tomorrow"* with an information `(i)` icon explaining that remaining credits may continue under clinic discretion. Do not block actions.
  2. **Review Overdue (Next date onward):** When an active, non-exhausted package has remaining credits, display a stronger confirmation warning before covered treatment (offering Continue / Back / Go to Packages).
- **Continuation authority:** Admin or staff may proceed with overdue covered visits under discretion; does not require per-visit admin preapproval (S27 N06).
- **Post-review audit feed & dashboard queries (`[USER—DECIDED]`, S27 N07, S30 Q08, Q09):**
  - The admin dashboard presents **two distinct query-derived lists**: (1) Overdue inactive packages with unused allowances, and (2) Actual post-review covered redemptions (`S30 Q09`).
  - These are **persistent query-derived views** that remain visible while their criteria remain unresolved; they are **not time-based repeating push notifications** or reminders.
  - When an admin personally authorizes a post-review covered visit, the committed visit is retained, but **no Needs Attention alert or notification ping is created for any admin** (`S30 Q08`).
- **Post-review confirmation modal (`[USER—DECIDED]`, S30 Q10):** An explicit confirmation modal is required before committing **every single post-review covered visit**.
- **Post-review justification notes in MVP (`[USER—DECIDED]`, S29 D06, S30 Q06, Q07, Q11):** Adopted for MVP. Mandatory for future receptionists; optional for admin. Explains why remaining benefits are consumed after review (e.g. Diwali clinic closure for 2–3 days); does not alter review dates or allowances.
  - **Frequency (`S30 Q06`):** Mandatory for **each individual post-review covered visit**; blanket package-level or daily justifications are strictly forbidden.
  - **Immutability (`S30 Q07`):** Submitted justification notes are **completely immutable for all users, including admin**. No edit backdoor or correction workflow exists in MVP.
  - **Typing assistance (`S30 Q11`):** In the UI, reusable suggestions are surfaced via browser-native autocomplete/history only for MVP; a central clinic preset library is deferred. The authoritative submitted text is persisted server-side with the specific committed visit.

## Lifecycle: Completed vs. Closed

- **Completed (`[USER—DECIDED / RECONFIRMED]`, S28 D07):** Automatic when all allowances are used (10/10 for short, 26/26 for long). No reason is asked or recorded.
- **Closed (`[USER—DECIDED / RECONFIRMED]`, S28 D07, D08):** Deliberate manual early termination prior to exhaustion. A closure reason is **mandatory** for admins and the future receptionist.
- **Strict Closure Permanence & Accidental Closures (`[USER—DECIDED]`, S30 Q12, Q13):**
  - Deliberate package closure is **permanent and irreversible in MVP**. Packages cannot be reopened on patient request (`S30 Q12`).
  - Accidental/mistaken closures (e.g. staff intending to close a knee package but closing a shoulder package) are **strictly non-reversible in MVP** (`S30 Q13`).
  - Defense against accidental loss is a **multi-step destructive confirmation dialog** displaying: patient name, exact complaint and package being closed, remaining allowances being forfeited, and an explicit warning that closure is permanent and irreversible, requiring affirmative confirmation.

### Final Agreed Closure Reason Presets (`[USER—DECIDED]`, S28 D08, D09, S30 Q15)

The UI exposes selectable textual presets plus a custom editable free-text field (`S30 Q15`):
1. **Duplicate Package Purchased**
2. **Wrong Package Registered**
3. **Patient Stopped Attending**
4. **Patient Requested Termination**
5. **Patient Deceased**
6. **Doctor Advised Stopping Treatment**
7. **Other — written reason required** (requires custom explanatory text)

*Explicitly rejected candidate labels:* "treatment completed early" (that is Completed), "clinic discontinued service", vague "clinic-initiated / administrative correction", "Nonpayment" (nonpayment does not trigger closure; S30 Q14), and "medical contraindication". "Doctor Advised Stopping Treatment" refers to stopping the entire course, not altering a single modality.

### Special Validation on "Patient Stopped Attending" (`[USER—DECIDED]`, S28 D10, D11)
- **Eligibility:** May be selected **only after the package review date has passed (when Review Overdue begins)**, never on or before the review date.
- **Enforcement:** Disabled/hidden in the UI prior to overdue, and **enforced on the backend**. Even an admin cannot select this reason before overdue. No minimum days since last visit is required.

## Closure authority, finance separation, and post-closure summary

- **Closure authority (`[USER—DECIDED]`, S27 N04, S28 D11):**
  - **Admin:** May close an active package anytime (before, on, or after review) with a valid reason (subject to the D10 overdue restriction on "Patient Stopped Attending").
  - **Receptionist (future role):** May close a package **strictly from the day after the review date onward (when it enters overdue status)** with a reason. Receptionist closure is forbidden before or on the review date.
  - **Staffing reality:** Admin is currently the sole application user; receptionist permissions govern future staffing only.
- **Closure never silently alters finances (`[USER—DECIDED]`, S28 D05):**
  - Manual closure stops future coverage, but **does not cancel the purchase invoice, erase unpaid balances, create a refund, or waive money**.
  - The purchase invoice retains its existing financial state until an authorized payment or final waiver is executed. The package module never computes an automatic prorated refund.
- **Nonpayment Never a Closure Trigger (`[USER—DECIDED]`, S30 Q14):**
  - Nonpayment alone does not auto-close or ordinarily close a package; zero-paid packages remain active until deliberate manual closure.
  - Valid early closure patterns are: (1) patient requested termination, (2) patient stopped attending (waited until after review date), or (3) mistaken registration.
  - Subsequent payment settles the old invoice receivable only and **never reactivates** a closed package; resuming treatment requires purchasing a new package.
- **Post-closure factual summary (`[USER—DECIDED DIRECTION]`, S28 D06):**
  - Immediately post-closure, display a summary: purchased price, total collected, remaining invoice balance, elapsed time/days, used visits, and relevant dates.
  - Provide direct navigation links to the purchase invoice (for collection or admin final waiver) and to the refund workflow (when built).
- **Closed Package History Visibility (`[USER—DECIDED]`, S30 Q16):** Completed and Closed packages remain visible in patient history under a dedicated "Completed/Closed" section showing original allowance, usage, closure reason, and associated invoice link.
- **Mistaken Package Handling & Workarounds (`[USER—DECIDED]`, S31 Q20, Q21, Q22, Q23):**
  - **Unpaid single-line invoice (Q20):** Admin closes the mistaken package (with reason "Duplicate Package Purchased" or "Wrong Package Registered"); if the invoice contains only that package and ₹0 paid, admin executes a terminal whole-invoice waiver. The closed package is never reinstated.
  - **Unpaid mixed invoice (Q21):** If an invoice contains an erroneous package (e.g. ₹4,500) and legitimate charges (e.g. ₹350 exam) with ₹0 paid, selective line voiding is not supported in MVP. Admin closes the erroneous package with reason, terminally waives the entire original invoice (₹4,850), and creates a clean replacement invoice for the legitimate charges (₹350 exam). If the exam was already paid, admin closes the package and terminally waives the remaining package balance (₹4,500).
  - **Partially paid mistaken package (Q22):** In mixed cases with partial collection (e.g. ₹4,850 invoice with ₹1,500 paid = ₹350 exam + ₹1,150 package): (1) close erroneous package with reason, (2) record a ₹1,150 refund against the original invoice in the planned refund module with a mandatory non-empty reason, (3) terminally waive residual balance on old invoice, and (4) if replacement package is purchased, issue a fresh invoice for the correct package and enter ₹1,150 as a new payment (Cash/UPI).
  - **Fully paid mistaken package (Q23):** If the entire package charge (₹4,500) was collected: close package with reason, record full ₹4,500 refund in the planned refund module with a mandatory reason, and if replacement package is purchased, issue new invoice and record new ₹4,500 payment.
- **Planned Refund Module & Tender Boundaries (`[USER—DECIDED]`, S31 Q22):**
  - The refund module is in scope and being built. Every recorded refund requires a **mandatory non-empty reason** explaining why.
  - Payment methods are strictly **Cash and UPI only**. Patient credit wallets, inter-invoice payment transfers, and dummy payment methods are explicitly rejected.
  - The owner explicitly accepts the gross-cash reporting caveat (recording refund + new payment balances net cash in the drawer, but reports will show gross cash out and in); extra verification of momentary physical cash handovers or additional audit subsystems are rejected.
- **Financial Waiver Does Not Revoke Entitlement (`[USER—DECIDED]`, S31 Q25):** Full invoice waiver of a valid package purchase settles the financial receivable only; the package remains active with all 10 or 26 covered visits usable.
- **Closure Presets Exclude "Refunded" (`[USER—DECIDED]`, S31 Q26):** "Refunded" is not an approved package closure preset; refunds are distinct transactions handled via the refund module with their own mandatory reasons.
- **No Retroactive Coverage (`[USER—DECIDED]`, S31):** Past billed visits cannot be retroactively converted to package-covered visits after correcting a complaint or package registration mistake.
- **Internal Clinic Display (`[USER—DECIDED]`, S31):** The software is an internal clinic record tool (no patient portal). The invoice view displays original billed amount and current amount due; payments and waivers are handled in their own dedicated records.
- **Single Combined Invoice for Package Checkouts (`[USER—DECIDED]`, S31 Q19):** At counter checkout, if a patient purchases a package alongside other services (e.g. consultation/examination fee, standalone services), the checkout issues **one combined invoice** with distinct invoice lines for each item (Option B). Creating a dedicated invoice solely to isolate package accounting is explicitly rejected.
- **Intentional Forfeiture on Renewal (`[USER—DECIDED]`, S29 D01, S30 Q04):** Closing an active package early to permit replacement intentionally forfeits remaining allowances (e.g. 18/26 used forfeits 8). No automatic transfer to replacement. Closing does not waive or erase debt on the purchase invoice; old debt remains independently collectible.
- **Active Package Blocks Complaint Closure (`[USER—DECIDED]`, S29 D07):** Staff cannot mark a complaint/course Closed while it has an associated Active package (including Review Overdue active packages). General complaint auto-closure remains parked.

## Multiple packages, renewals, and strict purchase blocking

- A patient may hold independent packages for *different* complaints.
- **Rejection of Queuing Model (`[USER—DECIDED]`, S28 D13, D14):** Queued advance sales and delayed auto-activation are **formally rejected**.
- **Strict MVP Purchase Blocking (`[USER—DECIDED]`, S28 D13, S29 D01):**
  - Purchasing another package for the *same complaint* is **strictly blocked** while an existing package is Active.
  - A replacement package can only be purchased after the existing same-complaint package has **Completed** (10/10 or 26/26 used) or been **Closed** with intentional forfeiture of unused visits. Merely crossing the review date does *not* unblock purchases if visits remain unused.
- **Renewal with Prior Unpaid Invoices ("Assist, Don't Block") (`[USER—DECIDED]`, S30 Q05):**
  - A patient may purchase a replacement package even if an earlier completed or closed package has an unpaid balance on its purchase invoice.
  - The software **assists rather than blocks**: it displays a prominent unpaid balance warning with direct navigation to invoice details, leaving the credit decision to staff discretion.
  - There is no automatic purchase block, no compulsory admin approval override, and no delinquent-renewal cross-report in MVP (deferred as scope creep).
- **Replacement Package Start Date (Approach A — Visit-Covered Anchor) (`[USER—DECIDED]`, S28 D14, S29 D02, S30 Q01, Q02):**
  - The system distinguishes **package purchase/registration date (`purchased_at`)** from **package start date (`start_date`)**.
  - When a replacement package is registered on Date $T$, the system checks whether any package has covered a visit for this complaint today:
    - **If YES:** Coverage and the review clock start on **Day T+1 (tomorrow)**. This ensures that a patient who already received their 1-per-day covered visit today on an exhausted package cannot receive an unbilled second covered treatment from the new package on that same date (`S29 D02`, `S30 Q01`).
    - **If NO:** Coverage and the review clock start **Today (Day T)** immediately (`S30 Q02`). For example, if an overdue package was closed after a multi-day absence without treatment today, or a mistaken package was registered and closed without treatment, the replacement package is immediately active and covers today's visit.
- **No Complex Picker:** Because overlapping eligible entitlements for the same complaint cannot exist, checkout requires no entitlement selection picker (S28 D14).

## Payment and activation

- **Review clock anchor (`[USER—DECIDED]`, S30 Q01, Q02):** The review clock begins on the **package start date (`start_date`)**, which is $T$ or $T+1$ based on prior same-day coverage, irrespective of payment collection date.
- **Payment-independent activation (`[USER—DECIDED]`, S29 D04):** Registration activates the package immediately regardless of payment collected. ₹0 received, partially paid, or fully paid all activate coverage. Unpaid balance does not automatically pause, expire, or close coverage; stopping coverage requires a deliberate authorized manual package closure. Purchase invoice receivable remains financially independent. Retires the earlier `OPEN—OWNER` payment-gating marker.
- **Recording Timing (`[USER—DECIDED]`, S29 D03):** There is no historical physical treatment timestamp in the data model. Coverage eligibility is validated at the moment reception records the visit in software (e.g. 10:00 AM physical treatment, 11:00 AM package registration, 11:05 AM visit logged → software sees active package at 11:05 AM and covers the visit). Retires the earlier treatment-timestamp concern.
- **Finance separation:** Purchase invoice balances remain visible and collectible independently of treatment coverage. No fictitious ₹0 receipts are created. The general receipt and finality rules in [payments and settlement](payments-and-settlement.md) remain in force.

## Pricing and offering configuration

- **Independent tier pricing (`[USER—DECIDED]`, S27 N23):** Routine and rehabilitation are distinct package categories, each with separately configurable package prices for short and long offerings.
- **Clinic-Configurable Prices (`[USER—DECIDED]`, S31 Q27):** Catalog prices for all four offerings (Routine 10, Routine 26, Rehabilitation 10, Rehabilitation 26) are configured by each clinic during setup, not fixed in the specification. Illustrative numbers like ₹4,500 in discussions are examples only.
- **Admin-Only Catalog Modifications (`[USER—DECIDED]`, S31 Q28):** Only admin may modify package prices and catalog-level commercial settings; receptionists cannot. The application recognizes strictly two roles: **admin and receptionist**; the hypothetical third "owner" role is explicitly discarded.
- **No Price Overrides at Checkout (`[USER—DECIDED]`, S31 Q29):** Issued package invoice lines always reflect the configured catalog price. Checkout-time price overrides or selling-price modifications are forbidden. Exceptional discounts are handled via standard invoicing: bill full catalog price (e.g. ₹4,500), collect agreed amount (e.g. ₹4,000), and admin terminally waives the residual balance (e.g. ₹500).
- **Complimentary Packages Invoiced at Catalog Price (`[USER—DECIDED]`, S31 Q30):** Free or gifted packages are registered and invoiced at their ordinary catalog price (e.g. ₹4,500); admin subsequently waives the full invoice balance. There is no zero-selling-price package sale in MVP.
- **Offering Management Deferred (`[USER—DECIDED]`, S31 Q31):** Self-service UI to disable existing package offerings, author new offerings, or configure arbitrary package rules is deferred post-MVP. Prices remain modifiable by admin.
- **Unconfigured Price Blocks Sale (`[USER—DECIDED]`, S31):** Attempting to register or sell a package whose catalog price has not been configured in clinic setup is strictly blocked by backend/UI validation.
- **Future-facing changes:** Catalog price updates apply to future purchases only. Historical issued invoices and purchased terms remain frozen (S28 D01).

## Open decisions and engineering audits

1. **Reopening and corrections after closure (S28 O05, S30 Q12, Q13):** Deliberate and accidental closures are strictly non-reversible in MVP (`S30 Q12`, `Q13`). Generic void/cascade workflows remain deferred out of MVP.
2. **Exact DDL and redemption integrity (O-15, O-17, O-18; S28 O09):** Atomic purchase-and-invoice issuance transaction, `start_date` column distinct from `purchased_at`, package-to-visit FK/redemption records, last-allowance race locking, and prevention of duplicate submissions.
3. **Existing package migration (O-20, O-28; S28 O08, S30 Q32–Q36, S31 Q32–Q36):** Import strategy and mapping for active patient packages from legacy data remain conditional on live data availability (determined by Q32).

### Dropped Scenarios from this Decision Round
- **Clinical Reclassification during Active Package (S30 Q17):** Dropped as an invalid clinical scenario. Moving patients mid-course between Routine and Rehabilitation will not occur; tentative refund/waiver discussion during exploratory brainstorming is explicitly not approved policy.
- **Complaint Reopening Independent of Packages (S31 Q18):** Dropped from package scope. Closed or Completed packages never reopen under any subsequent clinical complaint action; active packages strictly prevent complaint closure. Future complaint reopening rules belong to complaint-module design.
- **Cross-Complaint Package Coverage Impossibility (S31 Q24):** Rejected as operationally implausible. Packages are strictly bound to one complaint; a visit recorded for knee Complaint A cannot pull allowances from a shoulder Complaint B package. The knee visit bills at the normal rate, while the shoulder package remains untouched.

### Follow-Up Decision Reviews & Accepted MVP Behaviors (DR-01, DR-02)
- **DR-01 — Future Generic Invoice and Line Voiding (Status: DEFERRED FOR POST-MVP REVIEW):** The owner requested recording a personal post-MVP review item regarding adding a generic mechanism to void/reverse entire invoices or single invoice lines across packages, visits, examinations, and standalone services. The owner evaluated this against proliferating special-case exceptions and accepted the MVP workaround (whole-invoice waiver + replacement invoicing). This is a document-level future review marker, not approval to build an application audit-log table or void engine in MVP.
- **DR-02 — Whole-Balance Terminal Waiver (Status: ACCEPTED MVP BEHAVIOR):** A waiver applies to the entire remaining collectible invoice balance and terminally settles the invoice. It does not support arbitrary partial-value or single-line waivers in MVP.

### Remaining Questionnaire Items (Q32–Q40 Baseline)
- **Q32 — Active Packages Outside System (`UNANSWERED — FIRST NEXT QUESTION`):** On deployment day, are there active packages on paper/Excel to import? Options: A (real active packages), B (none, launch clean), C (discardable prototype/test data only).
- **Q33–Q36 — Historical Migration (`UNANSWERED — CONDITIONAL ON Q32`):** Import method (Q33), incomplete paper records (Q34), legacy commercial terms (Q35), historical debt import (Q36).
- **Q37–Q40 — Checkout & Edge Cases (`UNANSWERED`):** Standalone services coverage (Q37), different-complaint UI warning (Q38), unpaid packages in Needs Attention (Q39), erroneous committed visit correction (Q40).

## Evidence

- [Original handover, 28 October 2026](history/clinic_packages_handover_2026-10-28.md) — S25, baseline P01–P30.
- [Subsequent addendum, 28–29 October 2026](history/clinic_packages_addendum_2026-10-28_to_2026-10-29.md) — S26, A31–A32, initial 26-cap leaning.
- [Post-addendum discussion patch, 2 November 2026](history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md) — S27, N01–N24, O-01–O-28, owner-confirmed 26 cap and 1/day limit, Sunday counting rule, role-gated closure, registration clock, payment gating reopened.
- [Voice session decision log, 2 November 2026](history/package_decision_log_2026-11-02_voice_session.md) — S28, D01–D14, 7 closure reasons, "Patient Stopped Attending" date rule, post-closure factual summary, rejection of queuing, N+1 renewal start.
- [Pre-questionnaire decision log, 3 November 2026](history/clinic_package_decision_log_2026-11-03_pre_questionnaire.md) — S29, D01–D08, payment-independent activation, fixed 30-day review confirmed, mandatory post-review notes in MVP, active package blocks complaint closure, counter recording sequence.
- [Reconstructed final decision questionnaire, 3 November 2026](history/package_module_questionnaire_reconstructed_2026-11-03.md) — S30, Q01–Q16 settled (package start date anchor, configurable daily cap default 1, closure irreversibility for clerical errors, renewal debt assist-not-block, justification immutability, 2 persistent query lists, nonpayment independence), Q17 dropped, Q18–Q40 unanswered baseline.
- [Q18–Q40 questionnaire addendum, 3 November 2026](history/package_module_Q18-Q40_decision_addendum_2026-11-03.md) — S31, Q18–Q31 settled (single combined invoice, mistaken package refund/waiver workflows, Cash/UPI only, entitlement survival on waiver, admin-only pricing, no price override/free sale, no disable UI, unconfigured price block, internal display); DR-01/DR-02 logged; Q32–Q40 active baseline.
- [Reconciliation register](reconciliation-2026-10-27.md) — Cross-module decisions, archive manifest, and A11 workstream.
