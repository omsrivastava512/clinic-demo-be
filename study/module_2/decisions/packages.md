# Packages

Packages are required for MVP. This page incorporates the three named package records through **2 November 2026** (S25–S27), alongside the earlier Module 2 decisions. Status labels below reflect the reconciled decisions from the handover (S25), addendum (S26), and discussion patch (S27, PKG-PATCH-2026-11-02-01), as well as direct user clarifications.

## Established baseline

- Coverage is associated with a particular complaint/course. A covered complaint-treatment visit is a real clinical occurrence; bundled routine procedures do not each create a separate therapy charge (P03–P04).
- A package purchase is a billed fact distinct from a positive payment receipt and from treatment redemption. Package purchase and purchase invoice issuance are paired at checkout; positive payment receipts are separate transactions (P21, P30; S27 §7.1).
- **Hard Ceiling on Long Offering:** 26 package-covered complaint-visits is a strict hard maximum. On visit 26/26, the package completes/exhausts automatically and cannot cover a 27th visit (S27 N01, `[OWNER—REPORTED—CONFIRMED]`).
- **One Covered Treatment per Day:** The long offering ordinarily covers at most one eligible complaint-treatment visit per calendar day. A second genuine treatment on the same date is charged at the ordinary complaint-visit rate (S27 N02, `[OWNER—REPORTED—CONFIRMED]`).
- **Review Clock Start:** A package begins its review clock on its **registration/purchase date**, regardless of when payment is collected (S27 N13, `[USER—DECIDED]`).
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
- **Long offering:** Maximum 26 covered visits (`[OWNER—REPORTED—CONFIRMED]`, S27 N01).
  - **Review window:** Fixed **30 consecutive calendar dates**, Sundays included, beginning on registration day.
  - **Soft review:** Day 30 is the review date. No automatic hard time expiry; unused visits can continue under review warning/discretion until visit 26 completes the package or until manually closed (`[USER—DECIDED]`, S27 N03).
  - **Sundays & Calendar opportunities:** A 30-consecutive-day window contains 4 or 5 Sundays, yielding 26 or 25 non-Sunday opportunities. 26 is a hard quantity ceiling, not a commercial guarantee of 26 non-Sunday slots in 30 days (S27 §2.3).
  - Owner confirmation of the 30-day commercial wording versus the original "one calendar month" description remains tracked (`[OPEN—OWNER]`).
- **No holiday ledger:** Avoid an automatic absence/leave/holiday ledger or automatic extension engine (P08, P15; S27 §2.3).
- **No arbitrary grace expiry buffer:** Defer any second hard-expiry window until empirical attendance data justifies it upon owner request (S27 N08).

## Review-day and overdue user experience

- **Two distinct states (`[USER—DECIDED]`, S27 §4):**
  1. **Review Day (Day 10 counted / Day 30 calendar):** Display a small, non-interruptive nudge: *"Review due today · Regular charges may apply tomorrow"* with an information `(i)` icon explaining that remaining credits may continue under clinic discretion. Do not block actions.
  2. **Review Overdue (Next date onward):** When an active, non-exhausted package has remaining credits, display a stronger confirmation warning before covered treatment (offering Continue / Back / Go to Packages).
- **Continuation authority:** Admin or staff may proceed with overdue covered visits under discretion; does not require per-visit admin preapproval (S27 N06).
- **Post-review audit feed:** A query-derived **Needs Attention** list grouped by package surfaces overdue covered redemptions for admin visibility in MVP. Persistent notification tables with a `Dismiss` action are deferred to future enhancements (S27 N07, §6.1).
- **Justification notes (`[USER—CONDITIONAL]`, S27 N18):** If a post-review justification note feature is adopted, it is mandatory for receptionists and optional for admin. Whether MVP includes notes remains an open product choice.

## Coverage, repeated visits, closure, and renewals

- An applicable active package ordinarily covers eligible complaint therapy; intentional general closure belongs in the Packages workflow (P09, P20).
- **Same-day treatments on long offering:** At most one covered visit per calendar day (`[OWNER—REPORTED—CONFIRMED]`). A second genuine treatment on the same date is billed at ordinary complaint rates.
- **Closure authority (`[USER—DECIDED]`, S27 N04 & user confirmation):**
  - **Admin:** May close an active package anytime (before, on, or after review) with a documented reason.
  - **Receptionist (future role):** May close a package **strictly from the day after the review date onward (when it enters overdue status)** with a documented reason. Receptionist closure is forbidden before or on the review date.
  - **Staffing reality:** Admin is currently the sole application user; receptionist permissions govern future staffing only.
- **Mistaken package handling (`[USER—WORKING DIRECTION]`, S27 N21):** Admin closes the mistaken package with reason `"Made by mistake"`; the associated purchase invoice is handled via the existing invoice waiver workflow. Full generic void/cascade module is deferred out of MVP.
- **Multiple packages & renewals (`[USER—DECIDED]`):**
  - A patient may hold independent packages for *different* complaints.
  - **Current working restriction:** Purchasing another package for the *same complaint* while an existing package is Active is blocked (S27 N15).
  - **Queued advance purchase / Renewal:** When a replacement package is purchased, Day 1 of the new package is the **N+1 day** following completion of the previous package. Even if purchased in the afternoon of the same day, the new package's coverage starts the next day (the 1/day long offering cap applies complaint-wide across both packages on the transition date).

## Payment and activation

- **Review clock anchor:** Registration/purchase date begins the review clock, irrespective of when payment settles (S27 N13, `[USER—DECIDED]`).
- **Payment gating (`[OPEN—OWNER]`, S27 N14):** Explicitly reopened for owner consultation. User is evaluating full prepayment, partial payment with a grace pause buffer, or proportional allowance. Neither unconditional zero-payment coverage nor strict full-prepayment gating is locked; status is strictly **OPEN—OWNER**.
- **Finance separation:** Purchase invoice balances remain visible and collectible independently of treatment coverage. No fictitious ₹0 receipts are created. The general receipt and finality rules in [payments and settlement](payments-and-settlement.md) remain in force.

## Pricing and offering configuration

- **Independent tier pricing (`[USER—DECIDED]`, S27 N23):** Routine and rehabilitation are distinct package categories, each with separately configurable package prices for short and long offerings.
- **Future-facing changes:** Catalog price updates apply to future purchases only. Historical issued invoices and purchased terms remain frozen.

## Open decisions and engineering audits

1. **Owner payment policy (O-01):** Owner determination on whether packages require full prepayment, allow partial payment with a time pause, or permit zero-payment coverage.
2. **Long offering commercial wording (O-02):** Formal owner confirmation of the fixed 30-day term vs "one calendar month" description.
3. **Queued purchase schema & state machine (O-06, O-16):** Lifecycle states for Queued vs Active vs Review Due vs Overdue vs Completed vs Closed.
4. **Post-review note requirement (O-08):** Decision whether MVP requires a text justification note when continuing overdue coverage.
5. **Exact DDL and redemption integrity (O-15, O-17, O-18):** Atomic purchase-and-invoice issuance transaction, package-to-visit FK/redemption records, last-allowance race locking, and prevention of duplicate submissions.
6. **Existing package migration (O-20, O-28):** Import strategy and mapping for active patient packages from legacy data.

## Evidence

- [Original handover, 28 October 2026](history/clinic_packages_handover_2026-10-28.md) — S25, baseline P01–P30.
- [Subsequent addendum, 28–29 October 2026](history/clinic_packages_addendum_2026-10-28_to_2026-10-29.md) — S26, A31–A32, initial 26-cap leaning.
- [Post-addendum discussion patch, 2 November 2026](history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md) — S27, N01–N24, O-01–O-28, owner-confirmed 26 cap and 1/day limit, Sunday counting rule, role-gated closure, registration clock, payment gating reopened.
- [Reconciliation register](reconciliation-2026-10-27.md) — Cross-module decisions, archive manifest, and A11 workstream.
