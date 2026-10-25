# Clinic Module 2 — MVP Scope and Checkout/Payment Follow-up Addendum

**Date:** 25 October 2026 (Asia/Kolkata)
**Type:** Independent successor decision note created after `clinic_module2_successor_addendum_2026-10-25_171327_IST.md`.
**Purpose:** Preserve discussion decisions and future requirements without retroactively editing older handoff files. This records requirements and discussion status only, not implementation, deployed schema, or changes to GitHub/Supabase.

## Status vocabulary

- **LOCKED — MVP:** User explicitly agreed that the behavior belongs in the current release.
- **DEFERRED — FUTURE:** Excluded from MVP; preserve the idea or agreed constraints for later analysis.
- **CONDITIONAL AGREEMENT:** A principle agreed if the future capability is developed. Not full feature or DDL approval.
- **OPEN:** Requires further discussion; do not infer implementation or permission.
- **SUPERSEDED FOR MVP:** Earlier requirement or scope was explicitly changed by the newer decision, without erasing history.

## 1. New release-scope decision — retroactive corrections

**LOCKED — MVP SCOPE / DEFERRED — FUTURE:** Do not implement retroactive correction mechanisms in the first release. This includes after-issuance invoice-line additions for underbilling, invoice/line reduction or re-pricing to correct overbilling, and modification/voiding of previously recorded clinical occurrences due to overreporting/underreporting. Corrective actions may be genuinely necessary in the real world, but their dedicated application workflow is excluded from MVP. The earlier October 25 addendum's 'mandatory invoice-line append' classification is **SUPERSEDED FOR MVP** by this later explicit scope decision. Keep its design examples for future evaluation, not as launch commitments.

**LOCKED — MVP:** Invoice lines become immutable when the invoice is issued. The admin cannot append a forgotten service to an already issued Pending, Due, or Paid invoice in the MVP. Paid invoice financial immutability remains in force. Normal front-of-issuance preview allows errors to be caught before confirmation; it is not a retroactive amendment mechanism.

**LOCKED — MVP:** Preserve already agreed *ordinary financial settlement* capabilities: positive payments, partial receipts, old-Due follow-up payments, and owner/admin-approved waiver of an outstanding balance, including after earlier partial receipts and before final Paid settlement. A waiver represents deliberately forgiven collection; it is not automatically a clinical correction or invoice-line amendment. Do not introduce a dedicated 'overcharge correction via waiver + note' feature in MVP simply because an ordinary admin waiver exists. No retrospective editing of charges is authorized by a waiver.

**SUPERSEDED / CLOSED by later session decisions (§6):** The initial checkpoint left payment-entry corrections OPEN. The user subsequently rejected edits and in-app corrections of saved payment records for MVP, including errors discovered after submission. Accurate first-time recording is mandatory. Keep this older question only as chronological provenance; it is **not an active MVP decision**.

## 2. Future retroactive amendment requirements — captured, not an MVP commitment

**CONDITIONAL AGREEMENT:** If post-issuance invoice-line append is reintroduced in a future version, at minimum:

1. Existing issued invoice lines are not rewritten or deleted merely to append a new one; an omitted genuinely performed treatment would produce a new line with a real, validated clinical source.
2. Mark the new line explicitly as added **after invoice issuance**, rather than relying only on generic timestamps to infer an amendment.
3. Capture *when* it was added, *who* authorized/performed it, and *why* (short reason). Preserve the original line history and identifiable amendment provenance.
4. Authorization belongs to admin; do not grant post-issuance edits to receptionist. The prior candidate allows Pending/Due only, never Paid.
5. Handle the resulting revised bill, outstanding balances, existing payments and waivers atomically and prevent overcollection, duplicated source charges, repeated submissions and inconsistent settlement.

These are high-level conditions. **OPEN:** exact table fields, trigger versus RPC enforcement, reason constraints, append order, issuance-versus-current billed figure/history, financial reporting semantics, whether the whole feature is worth implementing later, and specific correction/credit/refund paths. No `is_amended`/`is_appended` column, trigger or new table is approved for MVP.

**DEFERRED — FUTURE (supersedes earlier optional-for-MVP stance):** Because the user has now excluded **all retroactive corrections** from MVP, clinical voiding/entered-in-error flows will **not ship in MVP**. Do not preemptively add voiding columns to the initial migration. Previously recorded conditional behaviors (admin-only; individual child void; parent visit effectively invalidates its children; block Paid-linked voids; prevent adjusted billed amount falling below collected money) remain a **future** discussion basis, not new schema authorization.

## 3. Additional ordinary checkout/payment agreements made during this continuation

**LOCKED — MVP:**

- Review/preview happens **before** issuing the invoice, after staff request Create Invoice and before final confirmation. It shows clinical entries, billable services/lines, their prices and **total billed**. Confirmation saves/issues the invoice (initially Pending); **payment selection follows successful invoice creation**, rather than being required before issuance. The after-issuance payment UI displays methods/amounts and the resulting invoice state; it is not the same as the pre-issuance clinical/billing preview.
- Initial receptionist checkout with **no money received** follows **Pay Later → Pending**. Receptionist cannot mark the new invoice Due with a ₹0 receipt. Actual partial receipt leaves remainder **Due**; full receipt produces **Paid**. This does not forbid the previously described admin process of establishing Due with ₹0 on an old Pending invoice.
- Every **strictly positive payment** must specify its method. Initial supported examples: cash, UPI and card. Exact method enum and expansion policy can be finalized during implementation.
- A patient may split payment across methods on the **same invoice and checkout**, e.g. part cash and part UPI. Record each actual tender as a **separate positive payment row** referencing that invoice; handle the combined operation transactionally with the required status transition. This extends the pre-existing centralized payments-table decision, not a second aggregate payment amount on the invoice header.
- The daily **cash collection** report is based on actual payment receipts (including partial receipts on still-Due invoices) and their **actual received dates**, not invoice creation or final Paid timestamps. Thus an unpaid newly appended hypothetical charge would not itself change daily cash collection. A separate persistent daily-collection table is not required for MVP; a query/view may compute it.

**OPEN — UI/technical implementation, not new product features:** exact preview screen layout and confirmation copy; split-tender form and extensible method enum; payment request idempotency, backend amount validation, authorization and concurrency. **CLOSED for MVP by §6:** manual receipt-time selection/backdating and later payment-entry correction are not permitted; backend records the save time automatically. Separate timestamps may still be considered in a future late-entry design.

## 4. What NOT to reopen automatically

- One central `payments` table with one row per positive receipt; separate billing, payments and waivers.
- Receptionist's limited initial-checkout rights; admin controls older Pending/Due balances and waiver decisions.
- Normal clinical-record and invoice creation is atomic; payment recording is a later operation.
- Paid invoices are final and locked for MVP.
- Ordinary clinic workflow, standalone-treatment and package modeling remain in their respective independent Module 2 issue registers. This scope note does **not** settle catalog, examination, package, correction or migration DDL questions.

## 5. Priority / next step

**Priority:** Complete mandatory ordinary payment-state and waiver-settlement edge cases, then the remaining catalog/package/checkout business decisions and implementation invariants. Do not spend another MVP design session on rare retrospective error correction unless a demonstrated regulatory or safety requirement forces it.

**Former clarification — CLOSED by §6:** The user explicitly requires accurate **initial** payment entry, not a later edit/reversal/correction UI. Once successfully saved, payment records are immutable in MVP. Preserve the operational risk of a mistaken saved entry; do not reopen it automatically.

## Provenance / supersession

Parent chain: `clinic_checkout_billing_decision_log_2026-10-18.md`; `clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md`; `clinic_module2_successor_addendum_2026-10-25_171327_IST.md`; related October 23 canonical billing decisions and successor addendum. Later explicit user scope choices take precedence over earlier 'mandatory' classifications **only for the changed MVP scope**. Historical documents remain unchanged.

---

## 6. Continued-session decisions — 25 October 2026 (later decisions prevail)

This section **adds to**, rather than edits, the historical discussion above. Its LOCKED statements supersede earlier OPEN questions or contrary proposals **for MVP only**. No schema, code, or implementation change is implied.

### A5 — Payment recording, immutability, timing and UI

**LOCKED — MVP: Payment amount correctness at first entry.** Staff must enter the **amount actually received**, verify it, and submit the payment. A successfully saved payment record is **immutable**: no edit, reversal, backdating, or in-app correction workflow in MVP, whether the invoice is Pending, Due, or Paid. Errors noticed *before* submission can be fixed in the entry form; errors discovered *after* saving require an out-of-app operational resolution pending future design. Do not treat this as permission to alter invoice charges or modify already recorded receipts. **SUPERSEDES:** Section 1's and Section 5's OPEN payment-correction questions.

**LOCKED — MVP: Same-day recording only.** Enter payments when money is received. The system sets the actual recording timestamp automatically; the operator cannot select a historical payment date, manually backdate, or later edit the timestamp. Daily collections attribute payments to their system-recorded local collection day. Receiving a payment today against an invoice issued earlier is allowed and counts toward **today's** collection. If staff fail to record a payment when received, retrospective entry is **not supported by MVP**; this is an explicit operational limitation. A future late-entry exception would require separate discussion on permissions, reason and audit trail; none is approved now. **SUPERSEDES:** Section 3's OPEN received-date-versus-entry-date UI question.

**LOCKED — MVP: No distinct payment receipt artifact.** Do not build a separately generated, numbered, printed, or sent payment receipt for the patient. The existing invoice/payment view and on-screen confirmation are sufficient for this offline clinic MVP; this does **not** eliminate recording the payment in the backend.

**LOCKED — MVP: Payment feedback is mandatory.** Only **after the backend confirms a successful save** should the UI display a clear recorded-payment confirmation, its amount and method(s), and the invoice's updated paid/due balance and status. If saving fails or is not confirmed, the UI must not report success. This is operational feedback, **not** a separate receipt issuance system.

### Checkout and payment-state transitions

**LOCKED — MVP: Receptionist can record actual partial payments at initial checkout.** After preview and invoice creation, if an invoice of ₹500 receives ₹200, record the ₹200 immediately and mark the unpaid ₹300 as **Due**. Do not force the receptionist to choose Pay Later for money that has already been received. If **nothing** is paid, Pay Later leaves it **Pending**; if **fully** paid, the status is **Paid**. The receptionist's partial-payment permission applies to **initial checkout** and does not grant authority to later amend issued charges, waive money, or independently manage aged Pending/Due settlements.

**LOCKED — MVP: One or more positive tenders can be recorded at checkout.** Multiple methods (e.g. cash + UPI) remain represented by separate immutable payment records linked to the same invoice; every positive payment has a specified method. Backend validation and consistency checks must prevent overcollection and inconsistent invoice status. Exact user-interface presentation and transactional implementation remain engineering details, not additional feature approvals.

### Waiver authority

**LOCKED — MVP:** **Only admin/owner** can waive an outstanding balance during permitted settlement. A **reason is mandatory**. The receptionist **cannot** waive any amount. This is an ordinary financial waiver of collectible balance, not a retroactive correction to clinical entries or issued invoice lines. Post-issuance overcharge/undercharge editing, charge append/reduction, and clinical record correction remain **DEFERRED — FUTURE**.

### A6 — Packages and scope boundary

**LOCKED — MVP SCOPE:** Packages must be supported in MVP because the user reports roughly **60–80%** of patients use them. The earlier suggestion to defer packages **from MVP** was explicitly withdrawn. **OPEN — DESIGN IN ANOTHER CHAT:** Whether a “30-day” package is a day window or session count, coverage/consumption rules, balances, pricing and exact data-model requirements have **not** been resolved in this conversation. Do not infer specifications or implement them from this note.

**PROVISIONAL DEFERRAL DIRECTION, not individually specified:** The assistant suggested deferring refunds of unused package value, transfers to another patient and validity extensions; the user replied “True,” then explicitly moved package design to another chat. Treat this as agreement with a lean approach, **not** as a complete set of individually ratified package rules. Confirm scope in the dedicated packages discussion.

### Revised handoff / decisions closed by this continuation

- **CLOSED — MVP:** May a previously saved payment be edited/corrected? **No.** Require accurate entry and confirmation before submission; no in-app correction path.
- **CLOSED — MVP:** Can an operator backdate a payment? **No.** System timestamp and same-day collection rule.
- **CLOSED — MVP:** Is a separate receipt needed? **No.** Explicit backend success and UI/payment/invoice status confirmation **are** required.
- **CLOSED — MVP:** Can receptionist accept a partial payment at initial checkout? **Yes.** This makes the invoice **Due**, not Pending.
- **CLOSED — MVP:** Who can waive outstanding balances? **Admin/owner only**, with mandatory reason.
- **CLOSED — MVP scope, OPEN details:** Packages belong in MVP, but their precise rules belong to the dedicated package design thread.

**Remaining implementation/security risks to address, not permission for new correction features:** Guard against duplicate payment requests, invalid amounts, conflicting concurrent collections and UI false success. Preserve immutable payment rows, role checks and consistent status/balance calculations. Operational reconciliation of mistakenly entered payments is a known **MVP limitation**, not a solved workflow.


## 7. Post-save cross-check — full conversation audit (25 October 2026)

**Audit scope:** Matched the full discussion in this migration conversation against §§1–6 and the October 18 / 23 / 25 inherited decision chain. Checked *decisions*, *deferred subjects*, *unaccepted suggestions*, and *supersession*, not just whether a keyword appears. Earlier log files remain historical; their obsolete OPEN or “mandatory” flags do not override newer explicit choices here. This is a document audit, **not** a deployed database or code audit.

### Coverage / current status

| Discussion topic | Audited result and location |
| --- | --- |
| Payment stages and initial Pending state | **LOCKED** — clinical/billing preview → confirmation/issuance → payment choices; §3 |
| Receptionist initial ₹0 / partial / full collection | **LOCKED** — Pay Later/Pending; actual partial/Due; fully received/Paid; §§3, 6 |
| Single payment method required | **LOCKED** — each positive payment has a method, with cash/UPI/card initial examples; §§3, 6 |
| Hybrid cash + UPI collection | **LOCKED** — separate positive payment records for one invoice, transactional checkout collection; §§3, 6 |
| Repeated admin payments on old Due | **PREVIOUSLY LOCKED and retained** — each actual new receipt records today's cash, no retrospective editing; §§1, 4, 6 |
| Daily collection versus invoice charges | **LOCKED** — sum actual positive receipts, including partial-Due receipts; invoice line aggregates describe billing, not cash; §3 |
| Post-issuance line append authorization / requirements | **FUTURE ONLY** — admin, reason, actor, timestamp, explicit post-issuance marker; formerly approved workflow **superseded for MVP**; §§1–2 |
| Erroneous charge/clinical record over- or underreport | **DEFERRED** — no specialized retroactive amendment, repricing, or clinical voiding in MVP; §§1–2, 6 |
| Ordinary waiver versus correction feature | **LOCKED distinction** — existing admin settlement waiver remains; overcharge-correction workflow not provided; §§1, 6 |
| Waiver note | **LOCKED** — admin-only with mandatory reason for MVP; §6 |
| Payment edit / correction after save | **LOCKED prohibited** — not even while Due; record amount accurately before submission; §6 |
| Backdating and late cash entries | **LOCKED prohibited in MVP** — server timestamp at receipt recording, no historical input; §6 |
| Physical payment receipt vs on-screen confirmation | **LOCKED** — no separate printed/numbered receipt; backend success + UI payment confirmation and current invoice balance/status; §6 |
| Package priority | **LOCKED scope** — packages required in MVP given user-reported 60–80% prevalence; package semantics belong to separate chat; §6 |
| Package refund, transfer, extension | **TENTATIVE lean-scope direction** — explicit full rules not closed; §6 |
| Original retired-chat frontend-change validation | **PARKED** — complaint deselection/procedure cleanup and analogous form validation were raised and explicitly postponed; not an agreed MVP UX rule |
| Reports displaying original-versus-amended invoice total | **NOT RATIFIED, DEFERRED with amendments** — earlier assistant suggestions did not receive user approval; §2 |

### Corrections made by this audit

1. Earlier parts of this same addendum still described payment edits and receipt-date selection as OPEN; §6 had already resolved them. These old entries now carry explicit **SUPERSEDED/CLOSED** labels.
2. Earlier wording made clinical voiding sound merely optional for MVP; the later general no-retroactive-corrections decision supersedes that stance and defers it **outside MVP**.
3. The invoice preview description now explicitly includes **total billed** and separates *pre-issuance review* from *post-issuance payment confirmation*.
4. Package refund/transfer/extension discussion now records the user's brief assent without pretending detailed package rules were finalized.
5. Preserved the parked frontend validation question and never converted an assistant suggestion to an approved requirement.

### Known deliberate boundaries (not omissions)

- Exact SQL DDL, RPCs, RLS policies, source FK design and concurrency strategies remain engineering/audit tasks; none was implemented by this file.
- The user moved package definition, use/expiry and pricing decisions to a **different chat**; this file intentionally does not decide 30-day = 30 sessions versus 30 calendar days.
- A saved incorrect payment can produce a wrong daily cash total with **no in-product repair mechanism** under the chosen strict MVP policy. This is a consciously accepted limitation, **not** a claim that the system can always guarantee correct bookkeeping if a user enters a false amount. A future reconciliation policy is deferred.
- No specialized error-correction or clinical-voiding solution is implied by the ordinary admin waiver; original financial/clinical truth must not be silently rewritten.
- The earlier October 25 predecessor's “mandatory invoice-line append” status is historical and now **superseded for MVP**. Keep its conditional requirements for future design.
