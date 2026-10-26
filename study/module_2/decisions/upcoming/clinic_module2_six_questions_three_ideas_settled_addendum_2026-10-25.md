# Clinic Module 2 — Settled Decisions After the Six-Question / Three-Idea Checkpoint

**Recorded:** 25 October 2026, Asia/Kolkata  
**Document type:** New, independent decision **addendum**; does **not** overwrite or replace the 18 October comprehensive log or later Module 2 records.  
**Scope:** Resolutions relating to the original six checkout/payment gaps, the three user ideas set aside around the break, and the directly connected payment/schema discussion.  
**Status standard:** `LOCKED` = accepted business rule or required system behavior; `LOCKED DIRECTION` = settled architectural concept, with exact SQL/UI implementation not dictated by this document; `SUPERSEDED` = earlier guidance replaced.  
**Not an implementation record:** No SQL migration, repository edit, deployment, or database audit is implied.

## 0. Source, precedence, and reading guidance

This addendum reconstructs the resumed discussion and cross-checks directly relevant decisions that the user said had already been resolved in other Module 2 conversations. It is **not** a verbatim transcript or a claim that the old full-length log has been reprinted here. For the full historical architecture and unrelated catalog/package/clinical backlog, continue to consult the earlier records.

Principal source records consulted:

- `clinic_checkout_billing_decision_log_2026-10-18.md` — original post-break payment/invoice log and intentionally parked questions.
- `clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md` — subsequent explicit repayment, payments-table, atomicity, and correction-scope decisions.
- `clinic_module2_scope_decision_addendum_2026-10-25.md` — later checkout, payment-entry, reporting, and MVP restrictions.
- `clinic_module2_waiver_catalog_addendum_2026-10-25.md` — later explicit `WAIVED` final-status and invoice-level waiver-metadata decisions.
- The 25 October resumed conversation in which the original six gaps and three ideas were recovered and the user confirmed the fully/partially waived outcomes, duplicate-prevention layers, and prior cross-chat resolutions.

**Precedence:** Later, explicit approvals supersede older conflicting status explanations or proposals. An agreed *outcome* does not automatically approve a particular database column, enum spelling, indexing strategy, constraint, or UI layout. The unclosed engineering/design matters are intentionally kept in a separate open-items code block in the accompanying response, not silently decided here.

## 1. The original six questions — disposition

### Q1. Multiple partial repayments on an existing Due invoice — LOCKED

- An existing `DUE` invoice may receive **multiple later partial repayments** over time.
- The **admin/owner** records a later payment against an already-issued Due invoice. This permission does **not** extend to the receptionist merely because the receptionist may collect partial payment at *initial* checkout.
- After a further partial receipt, if a collectible balance remains and no waiver occurs, the invoice remains `DUE` with the smaller balance. It must **not** be forced to `PAID` or prematurely waived.
- There is **no arbitrary cap** on the number of repayment events.
- Each positive receipt becomes an additional, individually preserved `payments` row; it does not overwrite an earlier payment.

**Example:** Invoice ₹500; payment ₹200; remaining Due ₹300. Admin later records ₹100; total receipts ₹300; remaining Due ₹200. Further partial payments are permitted.

**Provenance:** Resolved explicitly in a different Module 2 discussion and confirmed as settled by the user when this six-question backlog was revisited.

### Q2. Admin waives the remaining Due amount after partial payment — LOCKED

- The **admin/owner alone** may forgive the **entire outstanding collectible balance**, including when positive payments have previously been received.
- A waiver is **not** a payment. Previously received amounts retain their original payment rows; the waived remainder does not generate a fake payment row.
- The final invoice status for such a settlement is **`WAIVED`**, not `PAID`.
- Waiver requires a **reason**, the **acting admin/owner identity**, and a **timestamp**.
- This MVP supports waiver of the **entire remainder**, thereby closing the invoice. It does not support forgiving only some of the remainder while leaving the rest Due.

**Example:** Original bill ₹500, payment ₹200, admin waives final ₹300 → status `WAIVED`; actual collected ₹200; forgiven ₹300; amount outstanding ₹0.

**Provenance:** User confirmed the Due-waiver issue had already been resolved elsewhere; later waiver records explicitly establish the above rule, including status semantics.

### Q3. Atomic treatment/invoice checkout — LOCKED BEHAVIOR

- In the **normal receptionist checkout path**, saving the confirmed clinical/treatment data, invoice header, and issued invoice lines is a **single all-or-nothing database operation**. Failed invoice creation must not leave an unbilled, partially committed normal checkout.
- A clinician may still save a treatment clinically **earlier**, with receptionist billing happening later through the separately recognized path. This exception must not be mistaken for permission to partially commit the normal combined checkout.
- **Invoice issuance and payment recording are separate workflow stages.** Payment capture, when it occurs, must itself update the invoice's relevant settlement state **atomically** with the receipt insert; do not infer that the clinical write, invoice issue, and subsequent payment must all live inside one transaction lasting across screens or user decisions.
- Exact database RPC/transaction design and recovery strategy are implementation choices, not defined here.

**Provenance:** Earlier records and later Module 2 scope decisions confirm normal-checkout atomicity; an old 18 October OPEN label is superseded as a current requirement, not evidence that the final behavior remains undecided.

### Q4. Duplicate prevention when submission is retried — LOCKED REQUIREMENT

- The frontend must **block repeated submission while a checkout/payment request is in flight** (e.g., disable the submit control).
- The backend must also provide **idempotent retry behavior**: if the operation committed but the response was lost, submitting/retrying the same logical operation must not create a second invoice or second receipt.
- Frontend button disabling **alone is insufficient**: server success followed by network-response loss can cause a user to retry.
- The intended result is **one logical checkout/payment**, even across an uncertain network response. Uniqueness constraints and correctly scoped keys/locking may be needed to enforce this, but their exact implementation is not specified by this approval.

**Provenance:** User explicitly agreed to “both layers” in the resumed conversation. The requirement is decided; SQL/RPC specifics remain engineering work.

### Q5. Fully waived invoice with ₹0 collected — LOCKED

- A ₹500 bill may be **fully waived by admin** with **no patient money received**.
- Its **outstanding balance becomes ₹0** and its status becomes **`WAIVED`**; it is **financially closed**.
- **No payment record** is created when ₹0 was received.
- This invoice is **not** included among outstanding Pending/Due balances and is **not counted as ₹500 cash collected**.

**Example:** Original ₹500; collected ₹0; waived ₹500; due ₹0; status `WAIVED`; payment rows 0.

**Provenance:** Explicitly confirmed in the resumed six-question discussion.

### Q6. Editing Paid invoices — LOCKED / MVP CORRECTION SCOPE

- Once an invoice reaches **`PAID`**, it is **final and locked** in the MVP. `WAIVED` is likewise terminal/final under the later waiver policy.
- The more recent MVP scope also freezes **issued invoice lines** after issue; issuing a Pending/Due bill does not make its original line descriptions or amounts freely editable.
- There is **no retroactive invoice correction, price adjustment, post-issuance line append/remove, replacement/void-and-reissue, or clinical-record correction workflow in MVP**. These were discussed but later explicitly deferred rather than being turned into approval to edit locked financial records.
- Ordinary **admin waiver of an outstanding collectible balance** remains permitted; it is not a general-purpose mechanism for rewriting inaccurate clinical history.

**Provenance:** Paid immutability was already established; later Module 2 scope documents settled deferral of retrospective correction features.

## 2. The three user ideas set aside before the break

### Idea A. Explicit invoice waiver marker/flag — PARTLY RESOLVED AS A DIFFERENT, CONCRETE DESIGN

**Settled related outcomes:**

- `WAIVED` is a **distinct final invoice status**, including both 100%-waived unpaid invoices and partially paid invoices whose **entire remaining balance** is forgiven.
- Store required waiver **reason**, **admin/owner identity**, and **timestamp** **on the invoice** for this limited MVP design.
- A separate `invoice_waivers` event table is **not required** for the agreed single final waiver per invoice.
- In this restricted model, waived amount can be **derived** as billed total minus actual positive receipts once the invoice is finalized as `WAIVED`.
- This resolves the *need to identify* waived invoices. **It does not, by itself, ratify a separate redundant boolean `is_waived` column**; that exact extra-flag proposal was posed again immediately before the latest file request and was not answered in the supplied continuation.

### Idea B. Receptionist accepts partial payment at initial checkout — LOCKED

- At **initial checkout**, the receptionist may enter the amount **actually received**, even when less than the invoice total.
- The system records that receipt immediately and the invoice becomes `DUE` for the remainder. Receptionist does not need to mark it `PENDING` or hold unrecorded money.
- If **nothing is paid**, the receptionist uses **Pay Later** → `PENDING`.
- If the **entire bill is paid**, the invoice becomes `PAID`.
- The receptionist **cannot waive** any part of the balance and cannot independently modify previously issued, older Due invoices.
- Every positive payment needs a method. A single initial checkout may use **split tenders** (for example cash + UPI); each actual tender is a separate payment record associated with the same invoice.

**Example:** ₹500 invoice, ₹300 taken at initial checkout, ₹200 collectible later → one payment ₹300; state `DUE`; remaining ₹200.

### Idea C. Call/text admin when patient pays extra toward old dues — LOCKED MVP HANDOFF

- If a patient pays for today's invoice and offers additional money intended for an **older Due invoice**, the receptionist communicates with the **admin/owner manually** (call, text, or in person).
- The **admin**, rather than the receptionist autonomously reallocating amounts across old invoices, records the old-invoice receipt in the system.
- No new automatic in-app notification, general unallocated-payment wallet, or cross-invoice auto-allocation workflow was approved for the MVP.
- Historical and today's receipts remain distinct, correctly attributable payment events.

**Example:** Today's bill ₹500 collected, plus ₹100 offered toward older Due ₹300. Today's ₹500 is handled normally; receptionist contacts admin regarding the ₹100; admin records the older-invoice payment. No automatic linkage between the two invoices is implied.

## 3. Directly connected settlement and data-model decisions

### 3.1 Central `payments` table — LOCKED DIRECTION

- Use a **centralized payments collection** for all actual receipts against invoices.
- Cardinality is **one invoice to zero, one, or many payment rows**.
- One **positive amount actually received** = one new immutable payment row, including subsequent admin repayments and each method in a split tender.
- No row is created for a ₹0 amount, a mere invoice creation, or a waived monetary balance.
- Conceptually retain at least **invoice association, amount, payment method, and system recording time**; the earlier schema discussions also considered receipt/recorded-by information. Exact physical column list and enum types are separate implementation decisions.
- Saving a payment and applying a corresponding valid invoice status transition is **atomic**. Failed operations must not leave a recorded receipt with an inconsistent invoice state.

### 3.2 Financial truth is separated — LOCKED DIRECTION

Maintain these different quantities:

```text
Original billed amount = SUM(issued invoice-line amounts)
Actual amount received = SUM(positive payments for invoice)
WAIVED invoice: forgiven remainder = original billed - actual received
WAIVED invoice: collectible outstanding = 0
Unwaived, open invoice: collectible outstanding = billed - actual received
```

- **Invoice lines** explain what was billed; **payments** explain what money came in; **waiver metadata/status** explains why a portion was not collected. They are **complementary**, not competing relationships.
- The derived-amount model was accepted as the MVP direction; avoid treating a mutable `paid_amount` or `due_amount` header field as an alternative source of truth. Keep the workflow state concept distinct from its computed arithmetic.
- An issued invoice line captures the charged amount as a **historical snapshot**; later catalog price edits cannot rewrite issued invoice totals.

### 3.3 Invoice statuses and transitions — LOCKED BUSINESS MEANINGS

- `PENDING`: invoice issued, payment deferred through **Pay Later**, with no recorded receipt in that initial path.
- `DUE`: recognized collectible outstanding amount, including when a receptionist records a **partial initial receipt** or when the admin records another partial repayment without forgiving the balance.
- `PAID`: fully settled by **actual receipts**; final and locked.
- `WAIVED`: fully settled because **admin forgave the entire remaining balance**, even if some actual money was received earlier; final and locked.
- Both `PAID` and `WAIVED` mean **no collectible outstanding amount**. The difference matters for revenue/collections and audit: forgiveness is not cash.

**SUPERSEDED:** Earlier descriptions that a full waiver or a partially paid/waived invoice could be finalized as `PAID`. The later explicit `WAIVED` terminal status wins.

### 3.4 Payment-entry restrictions and collection reporting — LOCKED MVP

- Staff must enter the **accurate amount actually received at first submission**. Once successfully recorded, a payment is **immutable**: no later edit, reversal, deletion, retrospective correction, or backdating in MVP.
- Payments are recorded when received; the backend assigns the system timestamp. Staff cannot freely enter a prior receipt date. An admin recording a legitimate repayment **today** on an older invoice contributes to **today's** collection.
- A separate numbered/printed/sent **payment receipt artifact is not required** for MVP. Following confirmed backend success, the UI must show a clear confirmation of saved amount, method(s), updated status, and outstanding balance. Do **not** show success while server outcome is unconfirmed.
- **Daily cash-collection reporting sums actual receipt rows for the day**, including partial receipts on invoices still `DUE`, and actual receipts on invoices later finalized as `WAIVED`.
- Do **not** derive actual cash collected by simply summing invoices whose status is `PAID`, by summing their billed totals, or by including forgiven balances. This **supersedes** older simplified instructions to filter the end-of-day report to Paid invoices only.
- The absence of in-product corrections/backdating is a deliberately accepted operational MVP limitation; it does **not** guarantee human input is infallible.

## 4. What these settlements do *not* imply

The following are **boundary statements**, not newly resolved open issues:

- A general approval of every proposed schema field, boolean waiver flag, table index, enum value, or SQL implementation.
- Permission for receptionists to change old invoices or apply discounts/waivers.
- Permission to treat a waiver as collected cash or to rewrite the original issued bill.
- A decision that one visit necessarily equals one treatment, one invoice, or one invoice line. The clinic's visit/treatment/invoice **cardinality question was deliberately parked**.
- An approval to implement speculative clinical voiding, later-payment correction, invoice replacement, or package-redemption rules merely because they were mentioned.
- Evidence that the required behavior is already enforced by deployed code or Supabase policies.

## 5. Completion checkpoint

**All six original question topics have now been accounted for at the business-rule level** (with the relevant cross-chat settlements verified in later records). The three user ideas are also tracked: **receptionist initial partial payment** and **manual old-Due admin handoff** are settled; the **waiver-identification need** is substantially handled through the new terminal status and invoice-level waiver metadata, while the **extra `is_waived` boolean itself has not been explicitly accepted or rejected**.

The companion response supplies the live OPEN/DEFERRED questions in a **separate code block**, as requested. This file contains the **settled record only**, not a new unapproved design proposal.
