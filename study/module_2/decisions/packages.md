# Packages

Current discussion view incorporating the supplied October 25 scope continuation. Packages are in V1; their precise model is still open.

## Current position

- **Settled behavior:** offering definition, purchase/entitlement acquisition and redeemed treatment are different facts. A purchase is a separately billed event; coverage activates after full payment under a rule whose waiver implications still need clarification.
- **Settled:** each package-covered complaint-visit appears as its own ₹0 invoice line; a checkout of only covered visits still generates an invoice. Coverage must be proved, not inferred from a zero amount.
- Known offerings discussed: routine and rehab, each with 10-day and 30-day options. ₹4,500 routine / ₹6,000 rehab for 30 days are tentative; 10-day prices are unknown. Calendar days versus attended days is unconfirmed. Do not prorate missing prices.
- No advance sale of individual standalone sessions is supported in V1; this does not exclude prepaid packages.
- **Reconfirmed October 25:** packages must ship in MVP; the user reported roughly 60–80% of patients use them. Package details were moved to a dedicated discussion, not decided by the payment continuation.

## Open questions

- **O08:** offering and price representation, purchase source, entitlement and redemption relationships, activation after partial payment/waiver, and same-checkout purchase plus coverage.
- Complaint/tier/branch scope, multiple complaints, expiration, exhaustion, concurrent redemption and imported existing entitlements.
- How a covered visit line references/proves its specific package: visit context, redemption occurrence or another constrained relationship remains unselected.
- Combined therapy plus package-purchase checkout is a proposed/user-leaning flow, not complete approval of its UX or transaction contract. Cancellation/refund and other lifecycle details must not be inferred.
- Refund of unused package value, transfer and validity-extension deferral received brief lean-scope assent, not individually specified rules. Confirm these in the dedicated package discussion. No generic package-void mechanism is approved; retroactive correction features are outside MVP.

## Scenarios and changes in position

An invoice can contain knee therapy ₹200 and package-covered shoulder therapy ₹0. That settles invoice completeness, not whether the shoulder visit is eligible. Buying a package at the same checkout does not automatically activate it or authorize covering that checkout.

- [October 22 earlier log](history/clinic-billing-decision-log-2026-10-22.md), §4, deferred packages.
- [Parent register](history/clinic_billing_decision_log_2026-10-23.md), §2.3, supersedes that deferral with mandatory V1 packages; §§3 and 8 retain source-integrity and acceptance obligations.
- [Successor addendum](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md), scenarios S5–S6 and §§7, 11.3, 12, settles covered-visit invoice representation while leaving O08 open. Omitting covered visits was considered and rejected for the chosen whole-checkout invoice meaning.
- [October 25 scope continuation](history/clinic_module2_scope_decision_addendum_2026-10-25.md), §6 A6 and §7: reaffirms mandatory scope and reported prevalence; day-window/session-count, coverage, pricing and model remain dedicated design work. It does not erase earlier established distinctions or finalize those open rules.

Related: [checkout](checkout-and-invoice-lines.md), [payment versus full-payment activation](payments-and-settlement.md).
