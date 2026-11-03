# Packages

Packages are required for MVP. This page incorporates the five named package records through the **3 November 2026 pre-questionnaire session** (S25–S29), alongside the earlier Module 2 decisions. Status labels below reflect the reconciled decisions from the handover (S25), addendum (S26), discussion patch (S27, PKG-PATCH-2026-11-02-01), the voice session decision log (S28), and the pre-questionnaire decision log (S29), as well as direct user clarifications.

## Established baseline

- Coverage is associated with a particular complaint/course. A covered complaint-treatment visit is a real clinical occurrence; bundled routine procedures do not each create a separate therapy charge (P03–P04).
- A package purchase is a billed fact distinct from a positive payment receipt and from treatment redemption. Package purchase and purchase invoice issuance are paired at checkout; positive payment receipts are separate transactions (P21, P30; S27 §7.1; S28 D04).
- **Snapshotting of Purchased Terms (`[USER—DECIDED]`, S28 D01):** A patient purchasing a package snapshots the covered quantity (e.g., 26 visits) and price. Subsequent catalog changes to prices or quantities never alter existing purchased entitlements or billed invoices.
- **Clinical Visit Immutability (`[USER—DECIDED]`, S28 D02):** Clinical visits are recorded by reception after therapy finishes and the patient reaches the counter. Recorded visits are immutable in MVP (no visit correction/deletion feature).
- **Genuine Visit Redemption (`[USER—WORKING DIRECTION]`, S28 D03):** Package consumption is strictly traceable to actual committed complaint-visits, not an arbitrary manual attendance counter.
- **Hard Ceiling on Long Offering:** 26 package-covered complaint-visits is a strict hard maximum. On visit 26/26, the package completes/exhausts automatically and cannot cover a 27th visit (S27 N01, S28 D01, `[OWNER—REPORTED—CONFIRMED]`).
- **One Covered Treatment per Day:** The long offering ordinarily covers at most one eligible complaint-treatment visit per calendar day. A second genuine treatment on the same date is charged at the ordinary complaint-visit rate (S27 N02, S28 D01, `[OWNER—REPORTED—CONFIRMED]`).
- **Review Clock Start:** A package begins its review clock on its **registration/purchase date**, regardless of when payment is collected (S27 N13, S28 D04, `[USER—DECIDED]`).
- Covered complaint-visits retain real ₹0 invoice lines with verified package coverage. A covered-only invoice is allowed only with at least one real line and backend validation. After issuance, offer only Mark as paid; record no payment and zero collections. This established Q01 decision is not reopened by the source logs (R32–R33, R74).
- Examination/return and laboratory fees are independent patient-level charges, not complaint-visit redemptions. They do not consume complaint-package allowances or become covered solely by a complaint package (P05, P25). General examination/lapse-fee bands and their billing-only meaning remain governed by [examination fees](examination-fees.md), including S23.
- A package-covered visit and a waiver are different facts. Preserve the actual eligible visit/coverage link; neither a zero-valued line nor payment status by itself establishes a redemption.
- Individual standalone sessions are not presold in V1. Package coverage of standalone services is not settled by these logs.
- The user previously reported approximately 60–80% package use. This is a reported estimate, not verified data (P01).

## Offering and review directions through 2 November

- **Ten-visit offering:** Ten eligible covered complaint-treatment visits; exhaustion follows actual ten uses rather than elapsed days.
  - **Review counting convention (`[USER—DECIDED]`, S27 N09–N11):** Registration date counts as **Day 1, even if Sunday**. Beginning the following date, **skip Sundays** when counting toward the ten review dates (Monday–Saturday count normally).
  - The **tenth counted date itself is the review date** (gentle reminder with info icon).
  - Starting the **following calendar date**, the package enters **Review Overdue** status.
  - Real treatments on **any Sunday** remain eligible and consume an allowance if the clinic operates; Sundays are skipped only for review-calendar arithmetic, not clinical eligibility.
- **Long offering:** Maximum 26 covered visits (`[OWNER—REPORTED—CONFIRMED]`, S27 N01, S28 D01).
  - **Review window:** Fixed **30 consecutive calendar dates**, Sundays included, beginning on registration day.
  - **Soft review:** Day 30 is the review date. No automatic hard time expiry; unused visits can continue under review warning/discretion until visit 26 completes the package or until manually closed (`[USER—DECIDED]`, S27 N03, S28 D12).
  - **Sundays & Calendar opportunities:** A 30-consecutive-day window contains 4 or 5 Sundays, yielding 26 or 25 non-Sunday opportunities. 26 is a hard quantity ceiling, not a commercial guarantee of 26 non-Sunday slots in 30 days (S27 §2.3).
  - **Fixed 30-day review duration settled (`[USER—DECIDED]`, S29 D05):** The long offering uses fixed 30 consecutive calendar dates including Sundays. Settled as the operative software rule; retiring the earlier open commercial wording comparison (`OPEN—OWNER`).
- **No holiday ledger:** Avoid an automatic absence/leave/holiday ledger or automatic extension engine (P08, P15; S27 §2.3).
- **No arbitrary grace expiry buffer:** Defer any second hard-expiry window until empirical attendance data justifies it upon owner request (S27 N08, S28 D12).

## Review-day and overdue user experience

- **Two distinct states (`[USER—DECIDED]`, S27 §4):**
  1. **Review Day (Day 10 counted / Day 30 calendar):** Display a small, non-interruptive nudge: *"Review due today · Regular charges may apply tomorrow"* with an information `(i)` icon explaining that remaining credits may continue under clinic discretion. Do not block actions.
  2. **Review Overdue (Next date onward):** When an active, non-exhausted package has remaining credits, display a stronger confirmation warning before covered treatment (offering Continue / Back / Go to Packages).
- **Continuation authority:** Admin or staff may proceed with overdue covered visits under discretion; does not require per-visit admin preapproval (S27 N06).
- **Post-review audit feed:** A query-derived **Needs Attention** list grouped by package surfaces overdue covered redemptions for admin visibility in MVP. Persistent notification tables with a `Dismiss` action are deferred to future enhancements (S27 N07, §6.1).
- **Post-review justification notes in MVP (`[USER—DECIDED]`, S29 D06):** Adopted for MVP. Mandatory for future receptionists; optional for admin. Explains why remaining benefits are consumed after review (e.g. Diwali clinic closure for 2–3 days); does not alter review dates or allowances. In the UI, reusable suggestions are surfaced via browser-native autocomplete or local storage, while the authoritative submitted justification is persisted server-side with the specific covered visit. Retires earlier `USER—CONDITIONAL` status.

## Lifecycle: Completed vs. Closed

- **Completed (`[USER—DECIDED / RECONFIRMED]`, S28 D07):** Automatic when all allowances are used (10/10 for short, 26/26 for long). No reason is asked or recorded.
- **Closed (`[USER—DECIDED / RECONFIRMED]`, S28 D07, D08):** Deliberate manual early termination prior to exhaustion. A closure reason is **mandatory** for admins and the future receptionist.

### Final Agreed Closure Reason Presets (`[USER—DECIDED]`, S28 D08, D09)

The UI exposes selectable textual presets plus a custom "Other" option (not a hardcoded DB enum):
1. **Duplicate Package Purchased**
2. **Wrong Package Registered**
3. **Patient Stopped Attending**
4. **Patient Requested Termination**
5. **Patient Deceased**
6. **Doctor Advised Stopping Treatment**
7. **Other — written reason required** (requires custom explanatory text)

*Explicitly rejected candidate labels:* "treatment completed early" (that is Completed), "clinic discontinued service", vague "clinic-initiated / administrative correction", and "medical contraindication". "Doctor Advised Stopping Treatment" refers to stopping the entire course, not altering a single modality.

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
- **Post-closure factual summary (`[USER—DECIDED DIRECTION]`, S28 D06):**
  - Immediately post-closure, display a summary: purchased price, total collected, remaining invoice balance, elapsed time/days, used visits, and relevant dates.
  - Provide direct navigation links to the purchase invoice (for collection or admin final waiver) and to the refund workflow (when built).
- **Mistaken package handling (`[USER—WORKING DIRECTION]`, S27 N21, S28 O06):** Admin closes the mistaken package (using "Duplicate Package Purchased" or "Wrong Package Registered"); the associated purchase invoice is handled via the existing invoice waiver workflow. Full generic void/cascade module is deferred out of MVP.
- **Intentional Forfeiture on Renewal (`[USER—DECIDED]`, S29 D01):** Closing an active package early to permit replacement intentionally forfeits remaining allowances (e.g. 18/26 used forfeits 8). No automatic transfer to replacement. Closing does not waive or erase debt on the purchase invoice; old debt remains independently collectible.
- **Active Package Blocks Complaint Closure (`[USER—DECIDED]`, S29 D07):** Staff cannot mark a complaint/course Closed while it has an associated Active package (including Review Overdue active packages). General complaint auto-closure remains parked.

## Multiple packages, renewals, and strict purchase blocking

- A patient may hold independent packages for *different* complaints.
- **Rejection of Queuing Model (`[USER—DECIDED]`, S28 D13, D14):** Queued advance sales and delayed auto-activation are **formally rejected**.
- **Strict MVP Purchase Blocking (`[USER—DECIDED]`, S28 D13, S29 D01):**
  - Purchasing another package for the *same complaint* is **strictly blocked** while an existing package is Active.
  - A replacement package can only be purchased after the existing same-complaint package has **Completed** (10/10 or 26/26 used) or been **Closed** with intentional forfeiture of unused visits. Merely crossing the review date does *not* unblock purchases if visits remain unused.
- **Replacement Package Start Date (Approach A — Visit-Covered Anchor) (`[USER—DECIDED]`, S28 D14, S29 D02):**
  - When a replacement package is registered on Date $T$, the system checks whether any package has covered a visit for this complaint today:
    - **If YES:** Coverage and the review clock start on **Day T+1 (tomorrow)**. This ensures that a patient who already received their 1-per-day covered visit today on an exhausted package cannot receive an unbilled second covered treatment from the new package on that same date (S29 D02).
    - **If NO:** Coverage starts **Today (Day T)** immediately. For example, if an overdue package was closed after a multi-day absence without treatment today, or a mistaken package was registered and closed without treatment, the replacement package is immediately active and covers today's visit.
- **No Complex Picker:** Because overlapping eligible entitlements for the same complaint cannot exist, checkout requires no entitlement selection picker (S28 D14).

## Payment and activation

- **Review clock anchor:** Registration/purchase date begins the review clock, irrespective of when payment settles (S27 N13, S28 D04, `[USER—DECIDED]`).
- **Payment-independent activation (`[USER—DECIDED]`, S29 D04):** Registration activates the package immediately regardless of payment collected. ₹0 received, partially paid, or fully paid all activate coverage. Unpaid balance does not automatically pause, expire, or close coverage; stopping coverage requires a deliberate authorized manual package closure. Purchase invoice receivable remains financially independent. Retires the earlier `OPEN—OWNER` payment-gating marker.
- **Recording Timing (`[USER—DECIDED]`, S29 D03):** There is no historical physical treatment timestamp in the data model. Coverage eligibility is validated at the moment reception records the visit in software (e.g. 10:00 AM physical treatment, 11:00 AM package registration, 11:05 AM visit logged → software sees active package at 11:05 AM and covers the visit). Retires the earlier treatment-timestamp concern.
- **Finance separation:** Purchase invoice balances remain visible and collectible independently of treatment coverage. No fictitious ₹0 receipts are created. The general receipt and finality rules in [payments and settlement](payments-and-settlement.md) remain in force.

## Pricing and offering configuration

- **Independent tier pricing (`[USER—DECIDED]`, S27 N23):** Routine and rehabilitation are distinct package categories, each with separately configurable package prices for short and long offerings.
- **Future-facing changes:** Catalog price updates apply to future purchases only. Historical issued invoices and purchased terms remain frozen (S28 D01).

## Open decisions and engineering audits

1. **Reopening and corrections after closure (S28 O05):** Policy on whether Closed packages can ever be reopened; generic void/cascade workflows deferred out of MVP.
2. **Exact DDL and redemption integrity (O-15, O-17, O-18; S28 O09):** Atomic purchase-and-invoice issuance transaction, package-to-visit FK/redemption records, last-allowance race locking, and prevention of duplicate submissions.
3. **Existing package migration (O-20, O-28; S28 O08):** Import strategy and mapping for active patient packages from legacy data.

## Evidence

- [Original handover, 28 October 2026](history/clinic_packages_handover_2026-10-28.md) — S25, baseline P01–P30.
- [Subsequent addendum, 28–29 October 2026](history/clinic_packages_addendum_2026-10-28_to_2026-10-29.md) — S26, A31–A32, initial 26-cap leaning.
- [Post-addendum discussion patch, 2 November 2026](history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md) — S27, N01–N24, O-01–O-28, owner-confirmed 26 cap and 1/day limit, Sunday counting rule, role-gated closure, registration clock, payment gating reopened.
- [Voice session decision log, 2 November 2026](history/package_decision_log_2026-11-02_voice_session.md) — S28, D01–D14, 7 closure reasons, "Patient Stopped Attending" date rule, post-closure factual summary, rejection of queuing, N+1 renewal start.
- [Pre-questionnaire decision log, 3 November 2026](history/clinic_package_decision_log_2026-11-03_pre_questionnaire.md) — S29, D01–D08, payment-independent activation, fixed 30-day review confirmed, mandatory post-review notes in MVP, active package blocks complaint closure, counter recording sequence.
- [Reconciliation register](reconciliation-2026-10-27.md) — Cross-module decisions, archive manifest, and A11 workstream.
