# Payments and settlement

Current payment decisions as of 29 October 2026. Settled rules below are authoritative for the proposed MVP; exact schema, authorization and API work remain audited. Evidence and current issue dispositions are in the [reconciliation register](reconciliation-2026-10-27.md).

## Delivery phase and authority

**Locked / S24, S37 §9 and §22:** the MVP operating phase has one application role, admin. A separate reception role and its corresponding authorization/RLS are planned for post-MVP (reception is the first planned post-MVP addition; S37 §9 and §22 superseding earlier references to a later MVP phase). Reception permissions below describe that future role; this document does not verify deployed roles.

## State meanings and authority

- **Locked:** PAID means the full billed balance is discharged through actual receipts, or through the explicitly permitted zero-total settlements (package-covered-only visits, or calculated-zero package upgrades where current long price $\le$ original invoiced charge; S17 U01, S37 §25). WAIVED means admin forgave the entire remaining collectible amount, including after partial receipts. Both are terminal with zero collectible outstanding and no subsequent receipts, waiver, or state mutation (R36,R74; S37 §25).
- **Locked:** four invoice states are UNPAID (issued with no receipt), PARTIALLY_PAID (positive receipts with a remaining collectible balance), PAID and WAIVED. This explicitly approved naming set replaces the earlier PENDING/DUE labels (R37,R75,R81; S21).
- **Locked:** The current delivery phase uses admin only. When the planned reception role is introduced, it may finish initial collection after a browser interruption, checking the stored outcome before another write. For positive invoices: Pay Later/no receipt leaves UNPAID, a partial positive receipt leaves PARTIALLY_PAID, and full collection closes PAID. Pay Later ends that initial interaction; later receipts remain allowed under R39. The permitted zero-total invoices (covered-only visits and calculated-zero upgrades) retain only Mark as paid, with no payment row (R38, R74; S24; S37 §25).
- **Locked:** Admin now, and the planned reception role when introduced, may record repeated positive receipts against UNPAID or PARTIALLY_PAID invoices with collectible balance, including after Pay Later and on later days. There is no initial-checkout-only, invoice-age or arbitrary repayment-count limit. Partial collection leaves PARTIALLY_PAID; full balance collection closes PAID. Reception need not hand those receipts to admin. Wallets and automatic cross-invoice allocation remain excluded. (R39; S24).
- **Locked:** only admin/owner may waive. The planned reception role must have neither waiver controls nor backend waiver permission. Permission to record later receipts does not permit changing issued lines, saved payments or clinical history (R40; S24).
- **Refund Authorization — Admin Only (`[USER—DECIDED]`, S33 §4, T73):** Only admin has authorization to execute refunds in the planned refund module. The planned reception role must have neither refund controls nor backend refund permission. T73 explicitly settles that reception cannot issue refunds under any circumstances.

## Payment records and transaction boundary

- Invoice issuance commits and the invoice is returned before settlement. Payment rows and the required valid settlement transition save together in a separate transaction (R24).
- One positive actual tender component is one payment row; each row belongs to exactly one invoice, and an invoice may have zero, one, or many payments (R44–R45).
- Use BIGINT integer paise. Payments must be strictly positive; zero collection and waived money create no payment rows (R48). The permitted zero-total invoices (covered-only visits and calculated-zero package upgrades) have a separate Mark as paid closure after issuance, with settled_at/settled_by and no receipt or waived amount; ordinary no-receipt Pay Later does not apply to these cases (S17 U01, S37 §25).
- Only Cash and UPI are accepted backend enum values; exact SQL casing is not selected here. Split is a UI input mode. Operator enters both amounts explicitly; partial split may leave a balance and must not imply full payment (R42–R43).
- Split components and the state update are atomic: failure of any part rolls back the entire settlement. No positive tender component is silently dropped (R44).
- Do not collect more than current collectible outstanding. Backend enforcement under concurrency is required; exact function/locking/authorization implementation remains audit work (R41).
- Payment recorded_by is mandatory and derived from authenticated backend identity rather than a React-supplied recorder ID. The user baseline identifies profiles(id) UUID authentication; actual FK/default/security integration remains audited (R47).
- Keep both created_at and received_at as server-controlled TIMESTAMPTZ NOT NULL with server defaults. Neither may be client-backdated in MVP (R46).
- **Accepted recording limitation:** if staff/admin record physically received money later, software reports it on the server-recording day. Distinct historical receipt-time support is outside MVP; client backdating is not permitted.
- Saved payments cannot be edited, deleted, reversed, or retrospectively corrected, even on nonterminal invoices. Invoice deletion is prohibited on frontend/backend; payment invoice FKs use ON DELETE RESTRICT (R49–R50).

## Final waiver and metadata

- Waive the whole invoice's residual collectible balance, not individual invoice lines. A partial waiver that leaves the invoice PARTIALLY_PAID is outside this MVP model (R51, R55).
- WAIVED requires a nonempty free-text reason. UI reason presets may insert text. Ordinary PAID settlement needs no reason. Waiver/write-off share WAIVED without a WRITE_OFF enum or structured reason category (R52).
- Use settled_at and settled_by for final PAID or WAIVED closure; waiver_reason is mandatory on WAIVED and null on PAID. Nonterminal combinations and final enforcement SQL remain audited (R53).
- Keep metadata on the invoice; no separate invoice_waivers event table or redundant is_waived boolean is needed for this one-final-waiver model (R51).
- Discount workflow: issue standard prices, record actual money, then admin forgives the residual in settlement. The result is WAIVED; lines do not change. Reason examples mentioning an error do not create a retroactive invoice/clinical correction feature (R56).

## Derived money and reports

~~~text
billed = SUM(immutable issued line amounts)
received = SUM(positive payment amounts)
if final status is WAIVED:
    forgiven = billed - received
    collectible_outstanding = 0
else:
    forgiven = 0
    collectible_outstanding = billed - received
~~~

No independently authoritative header total, waived_amount, or outstanding_amount is selected. Views/functions and concurrency checks must preserve this arithmetic without multiplying line/payment totals through an unsafe join (R54, R66).

Daily collections sum actual dated positive payment rows, including receipts on still-PARTIALLY_PAID and later-WAIVED invoices. Do not equate cash with billed totals, PAID-only invoices, or forgiven amounts. Use the server-recording date limitation above, with exact local cutoff/time-zone implementation audited (R57).

No separately numbered/printed/sent patient payment receipt artifact is required. Confirm saved amount, methods, state, and balance only after backend-confirmed success; failed/unknown outcomes cannot claim success. Preserve the form for manual retry after confirmed rollback; lost responses require the separate idempotency audit (R58–R61).

## Package payment interaction and refund workflows — S25–S37 reconciliation

- **Payment-Independent Package Activation (`[USER—DECIDED]`, S29 D04, S30 Q01–Q02):** A package purchase invoice's balance is financially independent from package entitlement. Registration grants the package's full entitlement usable from its valid start date (`start_date`: $T$ or $T+1$ based on prior same-day coverage under `S30 Q01–Q02`), regardless of whether ₹0, partial, or full payment is received. A replacement package with `start_date = T+1` cannot cover visits today, regardless of payment. Unpaid balances do not suspend, expire, or close package coverage.
- **Financial Waiver Entitlement Invariance (`[USER—DECIDED]`, S31 Q25):** Waiving a package purchase invoice (e.g. for a complimentary package) settles the financial receivable only; the package remains active with all 10 or 26 covered visits intact.
- **Package Upgrade Financial Rules (`[USER—DECIDED / LOCKED]`, S37 §16, §24, §25):**
  - **Pricing Formula:** The upgrade invoice charge is calculated as:
    $$\text{upgrade\_charge} = \max(0, \text{current\_price\_26} - \text{original\_10\_invoiced\_charge})$$
    Subtracting the original short-package purchase-line invoiced charge ensures unpaid receivables remain on the original invoice without being charged twice.
  - **Waivers Preserved:** If the original package charge was partially or fully waived, that waiver remains a valid concession; waived money is not added back or clawed back into the upgrade calculation.
  - **Zero-Floor & Refunds:** Floored strictly at ₹0. A negative price difference does not create a negative invoice line or an automatic cash refund; any actual cash reimbursement must follow the separate authorized refund workflow.
  - **Calculated-Zero Settlement (`S37 §25`):** If the calculated price difference is zero, issue and return the invoice with the ₹0 charge, followed by a separate **Mark as paid** settlement action with no payment row and zero collections.
  - **Nonblocking Invoice-Refund Context (`S37 §24, §25`):** Refunds recorded against the original package's invoice are displayed in the upgrade confirmation dialog as invoice-level context. Refunds do not automatically block the upgrade; admin exercises discretion after inspecting invoice details.
- **Ordinary Care Unblocked by Debt (`[USER—DECIDED]`, S37 §11):** Real completed complaint-visits can always be recorded even if the patient has numerous unpaid visit invoices. The clinic cannot refuse to log care that has physically occurred. Logging care does not waive old or new debt.
- **Future Receptionist Package-Debt Restriction (`[USER—SETTLED FUTURE POLICY]`, S37 §10, §22):** When a receptionist role is introduced post-MVP, reception cannot sell an additional package if the patient has *any* package-linked purchase invoices that remain unsettled (UNPAID or PARTIALLY_PAID), across all complaints and older purchases. Admin retains discretion to proceed despite debt.
- **Planned Refund Module (`[USER—DECIDED]`, S31 Q22, S33 §4):** The refund module is in scope and being built. Every recorded refund requires **admin-only authorization (T73)** and a **mandatory, non-empty reason** explaining the reimbursement. Early termination settlements and refunds are discretionary case-by-case clinic decisions; software does not automate or guarantee prorated refunds.
- **Strict Tender Boundaries (`[USER—DECIDED]`, S31 Q22):** Payment methods are strictly **Cash and UPI only**. No patient credit wallet, no unallocated patient balance, no inter-invoice payment reallocation, and no dummy tender methods exist.
- **Mistaken Package Financial Correction (`[USER—DECIDED]`, S31 Q20–Q23):**
  - *Unpaid single-line invoice (Q20):* Admin closes mistaken package; if invoice contains only that package, no visits were used, and ₹0 paid, admin terminally waives the whole invoice balance.
  - *Unpaid mixed invoice (Q21):* Admin closes package with reason; if ₹0 paid, terminally waives the whole invoice balance and issues a fresh replacement invoice for valid items (e.g. ₹350 exam). If the legitimate charge was already paid, admin closes the package and terminally waives the remaining package balance (₹4,500), leaving the legitimate payment intact.
  - *Partially paid mixed invoice (Q22):* Admin closes package, issues a refund against the old invoice for money attributable to the package with a mandatory reason, terminally waives residual balance on the old invoice, and if a replacement package is purchased, registers an ordinary new payment on the replacement package invoice.
  - *Fully paid package (Q23):* Admin closes package, records full refund with mandatory reason, and if a replacement package is purchased, enters an ordinary new payment on the replacement package invoice.
- The owner approved recording separate refund and new-payment entries, with a mandatory reason for each refund, as sufficient clinic bookkeeping without a credit wallet or physical cash verifier; the assistant's gross-cash reporting warning was noted but is not an approved policy requirement or extra audit hurdle (`S31 Q22`).
- **Needs Attention & Debts Model (`[USER—DECIDED]`, S32 Q39):**
  - Packages carry no payment status or balance; invoices alone carry payment status and outstanding debt.
  - There is no separate package unpaid category or special priority. General due-invoice attention is sorted by remaining balance descending.
  - **No minimum balance threshold:** Any positive balance, including ₹100, remains visible until fully settled by payment or waiver.
  - In addition to financial dues, Needs Attention can surface clinic-defined patient warning notes (e.g. behavioral notices) across multiple relevant application locations.
- **Deferred Void Mechanism (`DR-01`, S31):** Adding generic invoice-level or invoice-line void/reversal features is deferred for post-MVP review.
- **Terminal Whole-Balance Waiver (`DR-02`, S31):** Waivers apply strictly to the whole remaining balance; partial or line-level waivers are not supported in MVP.

## Open audit and future boundaries

**PC08 settled / S24:** browser interruption does not require an admin takeover. Future reception may resume initial collection after checking the stored outcome. Pay Later ends that initial interaction, while R39 independently permits later receipts. Exact retry/recovery and reception-role/RLS enforcement remain implementation audits.

Exact DDL, RLS, recorder/ownership integration, transition enforcement, RPC/API payloads, confirmation/cancel/error state, concurrency, payment indexes, queries, and migration remain unfinished (A06–A09, A12). Suggested invoice_id/received_at indexes are not approved merely by being suggested. Key/hash/invoice idempotency remains provisional. No refund, reversal, saved-payment correction, debt automation, or cross-invoice allocation is introduced.

## Evidence

Current rules and source references: [reconciliation register](reconciliation-2026-10-27.md#decision-register). Original versions are in [history/](history/); their wording does not override the current rules above.

Related: [checkout](checkout-and-invoice-lines.md), [packages and activation](packages.md), [examination charges](examination-fees.md).

Current authority clarification: [collection permissions and delivery phases](history/clinic_module2_reception_payment_authority_2026-10-29.md) (S24). Q01–Q07 and clarity question PC08 are settled; implementation and parked audits remain unfinished.
