# Packages

Packages are required for MVP. This page incorporates the two named package logs through **29 October 2026** (S25–S26), alongside the earlier Module 2 decisions. **Scope warning:** a later package discussion exists but was explicitly excluded from this incorporation. Compare it before treating these package terms as the latest decisions or implementing them. Status labels below describe the two supplied logs at their cutoff; reported owner statements are not fresh confirmations.

## Established baseline

- Coverage is associated with a particular complaint/course. A covered complaint-treatment visit is a real clinical occurrence; bundled routine procedures do not each create a separate therapy charge (P03–P04).
- A package purchase is a billed fact distinct from a positive payment receipt and from treatment redemption. Whether a purchase and treatment share an invoice, and whether same-checkout purchase can cover treatment, remain open (P21,P30).
- Covered complaint-visits retain real ₹0 invoice lines with verified package coverage. A covered-only invoice is allowed only with at least one real line and backend validation. After issuance, offer only Mark as paid; record no payment and zero collections. This established Q01 decision is not reopened by the source logs (R32–R33,R74).
- Examination/return and laboratory fees are described in the logs as independent patient-level charges, not complaint-visit redemptions. They do not consume complaint-package allowances or become covered solely by a complaint package (P05,P25). General examination/lapse-fee bands and their billing-only meaning remain governed by [examination fees](examination-fees.md), including S23.
- A package-covered visit and a waiver are different facts. Preserve the actual eligible visit/coverage link; neither a zero-valued line nor payment status by itself establishes a redemption.
- Individual standalone sessions are not presold in V1. Package coverage of standalone services is not settled by these logs.
- The user previously reported approximately 60–80% package use. This is a reported estimate, not verified data (P01).

## Offering and review directions through 29 October

- **Ten-visit offering:** ten eligible covered complaint-treatment visits; exhaustion follows actual ten uses rather than ten elapsed days. A review date is intended after ten dates excluding Sundays. Saturday is not automatically excluded. Exact start-day, review-day warning boundary, Sunday purchase and local-time handling remain open (P06–P08).
- **Long offering:** the doctor originally described one calendar month, Sundays included. The later user-selected software direction in S26 uses a fixed 30 consecutive calendar days for review, rather than an anniversary month. Owner approval of that commercial change remains **open** (P02,P14,P28).
- **26-visit ceiling:** proposed/leaning as of S26, not established by these two logs as accepted owner policy. Whether it is a commercial entitlement or safeguard remains open (A31).
- A 30-consecutive-day window contains four or five Sundays, giving 26 or 25 non-Sunday dates. This is calendar arithmetic, not a promise of treatment opportunities (A32).
- The user direction favors one soft-review approach for both offerings, without automatically terminating coverage solely because the review threshold passed. For an overdue active package, the required acknowledgement, any reason, who may proceed and the extent of clinic/owner approval remain open. Review overdue is distinct from exhausted or closed (P07,P10,P16).
- Avoid an automatic absence/leave/holiday ledger or automatic extension engine. Reported clinic-closure concessions still need a workable operational policy; this does not settle whether any extension mechanism is needed (P08,P15).

## Coverage, repeated visits and closure

- The user direction rejects an ordinary arbitrary per-visit package-on/off toggle. An applicable active package ordinarily covers eligible complaint therapy; intentional general closure belongs in the Packages workflow (P09,P20). Exact eligibility and exceptional handling remain open.
- The doctor reportedly allows two genuine treatments for the same complaint on the same day, with a warning rather than a date-based prohibition. Distinguish genuine treatment from duplicate submission. For the short package, two covered visits would consume two allowances under the user's proposed direction; any override remains open (P17–P18).
- The doctor reportedly said the long offering covers one treatment for the complaint on a date and normally bills a second genuine treatment. Whether this daily rule changes if the proposed 26 cap is adopted remains open. Do not silently replace the reported rule with a different commercial policy (P19,A31).
- The latest S26 closure direction describes an explicit admin action with a reason, separate from refund processing. An earlier statement allowed reception closure after review. Admin-only at any time is the **working interpretation**, and the conflicting receptionist permission/timing still needs confirmation (P09,P26).
- Closure does not itself record a refund, waiver, invoice cancellation or payment reversal. Treatment history and the distinction between closure and exhaustion must be preserved. Refund, reopening, erroneous-invoice correction, and unused-value handling remain open/separate (P26).

## Payment and activation through 29 October

- S25–S26 record the user's direction that zero or partial collection should not **automatically** block package use solely because of the balance. The exact event making a package usable, same-checkout ordering and any owner-imposed operational payment rule remain open (P22–P23,P30). This supersedes the older full-payment activation direction **within these two logs' cutoff**; the excluded later discussion must be reconciled before treating it as the latest rule.
- Outstanding purchase-invoice balances remain visible and collectible. They are separate from treatment coverage; no or partial collection is not a waiver. The general receipt, finality and role rules in [payments and settlement](payments-and-settlement.md), including S24, remain in force.
- Previously discussed ₹4,500/₹6,000 prices for historical 30-day examples are tentative and must not be assumed to price the revised offering. Short-offering prices and final catalog/configuration rules remain open (S25 §B2; S26 §10).

## Visibility and proposed workflow

Package/complaint visibility, age and usage, overdue Continue/Back/Packages navigation, different-complaint indications, and admin Needs Attention are user proposals/directions with exact UX still open (P10–P13,P27). A later S26 proposal calls for admin attention when an actual covered visit is recorded after review; event frequency, contents, acknowledgement and delivery are not settled. Do not block an unrelated complaint merely because another complaint has a package.

## Open decisions and engineering audits

1. Owner confirmation of the 30-day commercial definition; proposed 26 cap and its meaning; long-offering same-day coverage rule; and whether post-review continuation requires approval.
2. Review-date calculation boundaries, inclusion of purchase day, Sunday purchases, local dates and first-warning date.
3. Final manual-closure role/timing, reasons and any refund evidence; treatment of erroneous issued invoices and unused value.
4. One versus multiple usable packages for a complaint, advance purchases, renewal ordering and selection between entitlements.
5. Exact usable/activation event, outstanding-payment policy, same-checkout coverage and purchase/invoice boundaries.
6. Specific package-to-visit redemption representation, patient/clinic/complaint integrity, duplicate submissions, last-allowance and same-day concurrency, imported existing packages and migration.
7. Offering configuration and prices, complaint/therapy-tier scope, expiry or exhaustion transitions, closure/reopening, notifications, and any future refund/transfer/extension policy.

These open items are not permission to choose a schema, implement a role, approve a refund engine or infer a rule from an assistant recommendation.

## Illustrative cases

- Ten eligible covered visits complete the short allowance, whether they occurred over ten days or included separately confirmed same-day treatments. The precise same-day exception remains open.
- Under the reported long-offering rule, an additional genuine same-complaint treatment on one day is normally billed separately. Whether a proposed cap changes that is unresolved.
- An overdue review date alone does not silently consume remaining short-package allowances. Staff authority to continue, and the timing and wording of warnings, remain open.
- A verified package-covered complaint visit may yield a ₹0 line. A separate applicable examination fee remains independently chargeable on the invoice.

## Evidence

- [Original handover, 28 October](history/clinic_packages_handover_2026-10-28.md) — P01–P30, user decisions, owner reports, proposals and open questions.
- [Subsequent addendum, 28–29 October](history/clinic_packages_addendum_2026-10-28_to_2026-10-29.md) — changes, A31–A32, supersessions and unresolved conflicts.
- [Reconciliation register](reconciliation-2026-10-27.md) — cross-module decisions and remaining A11 work.

Both source files are retained byte-for-byte. The later package patch is excluded at the user's request; this page is a cutoff-scoped reconciliation, not a verified account of later package decisions.
