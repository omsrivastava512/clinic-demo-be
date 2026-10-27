# Packages

Current package decisions as of 27 October 2026. Packages are required for MVP. Covered-only zero-total invoices are explicitly allowed; remaining package design is parked in its dedicated discussion.

## Settled MVP position

- Offering definition, purchase/entitlement acquisition, and redeemed treatment are different facts. A purchase is separately billed. Coverage activation after full payment is the current direction; waiver and partial-payment implications remain open in package design.
- Each package-covered complaint-visit has its own zero-valued invoice line. An entirely covered checkout still requires an invoice. Coverage must be proved through the eventual entitlement/redemption model, not inferred from zero price.
- Routine and rehabilitation offerings with 10-day and 30-day options were discussed. ₹4,500 routine / ₹6,000 rehab for 30 days remain tentative; 10-day prices are unknown. Calendar-day versus attended-day meaning is not confirmed. Do not prorate missing prices.
- Individual standalone sessions are not presold in V1; prepaid packages remain distinct.
- The user previously reported roughly 60–80% package use and reconfirmed that packages must ship. That is an owner report, not verified usage data.

## Covered-only invoice exception — Q01 resolved

A zero-total invoice is permitted when all its real invoice lines are package-covered. Retain at least one line and validate coverage on the backend (R32–R33,R74).

After issuance, show only Mark as paid, with no Pay Later or payment-method collection choice. Settle PAID using the existing actor/time metadata, create no payment row, and report zero collections. It is not a waiver. An applicable uncovered therapy or automatic fee that produces a positive total uses ordinary settlement. Package coverage/redemption and activation rules remain in the dedicated audit.

## Parked questions for the dedicated discussion

- Offering/price representation, purchase source, entitlement/redemption relationships, imported existing entitlements, and proof of coverage.
- Activation after full/partial payment or waiver; buying a package and covering treatment within the same checkout.
- Day-window versus attended-session rules; complaint, therapy tier, branch scope; expiry, exhaustion, multiple complaints, and concurrent redemption.
- Combined package-purchase/treatment UI and transaction contract; separate forms do not automatically require separate invoices.
- Refund, transfer, and validity-extension lean-scope assent was not individually specified; retain that uncertainty rather than inventing lifecycle policies.
- No generic package void/correction mechanism is approved. Retrospective correction features remain outside MVP.

## Scenarios

Knee therapy ₹200 plus covered shoulder therapy ₹0 produces two visit lines and a positive total, subject to verified shoulder coverage. An entirely covered invoice may total zero. Purchasing a package during checkout does not itself establish immediate coverage; activation/redemption details remain parked.

## Evidence

Current rules and source references: [reconciliation register](reconciliation-2026-10-27.md#decision-register). Original versions are in [history/](history/); their wording does not override the current rules above.

Related: [checkout](checkout-and-invoice-lines.md), [payments and settlement](payments-and-settlement.md).

Current authority: [user instructions and consistency corrections](history/clinic_module2_current_decision_authority_2026-10-27.md). Q01–Q05 and Q07 are settled; Q06 remains open. Earlier answers are preserved in [S17](history/clinic_module2_reconciliation_resolutions_2026-10-27.md).

Latest answer: [clinical preview and fee visibility](history/clinic_module2_clinical_preview_fee_visibility_2026-10-27.md).

Catalog authority: [admin-only prices and availability](history/clinic_module2_admin_catalog_authority_2026-10-27.md). Q07 is closed.
