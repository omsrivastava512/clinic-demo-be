# Package Module — Voice-Session Decision and Discussion Log

**Session date:** 2 November 2026 (IST)  
**Scope:** Only subjects actually raised in this conversation, including the opening handover assessment and the subsequent voice discussion. This is a **conversation decision record**, not an implementation specification or confirmation that code is deployed.  
**Reference checked:** `omsrivastava512/clinic-demo-be`, particularly `study/module_2/decisions/README.md`, `packages.md`, and the 2 November package patch. Older records are background; later explicit statements in this session control where they differ.  
**Labels:** **DECIDED** = directly affirmed here; **RECONFIRMED** = an existing decision acknowledged here; **WORKING DIRECTION** = chosen direction with mechanics not final; **OPEN** = actively raised but unanswered; **DEFERRED** = expressly outside current package MVP; **ENGINEERING CHECK** = a safeguard or verification, not a new owner policy.

## A. Decisions made or reaffirmed in this session

### D01 — Bought terms cannot be reduced retroactively — DECIDED
A patient buying a **26-visit package retains the 26 covered-visit allowance** even if the catalog offering is later changed to 24. A subsequent catalog price increase **does not change the amount already invoiced or owed** by an existing purchaser. Purchased quantities and prices must be snapshotted, not read afresh from editable catalog settings. The broader proposal to snapshot daily caps, review formula, category, etc., was discussed as a sound design extension; the exact representation of every field remains an **engineering design**, not a newly approved DDL.

### D02 — Clinical visits are immutable in the MVP — DECIDED / RECONFIRMED
A visit is recorded by reception **after the patient finishes the therapy and comes to the counter**. Recorded visits cannot be edited or deleted. There is **no current visit-correction feature**; a future **auditable void** capability was mentioned, but is not part of this MVP. An incorrect entry is not repaired by inventing a package-specific cancellation operation.

### D03 — Redemption must correspond to genuine saved visits — WORKING DIRECTION
Package consumption should be traceable to the **actual committed covered complaint-visit** and its specific purchased package, rather than a manually editable attendance count. This was discussed and is consistent with the existing package design records, but the final foreign-key/redemption schema was **not approved** during the session. A second genuine treatment is different from an accidental duplicate network submission. The latter requires checkout safeguards, not permission to edit historical visits.

### D04 — Purchase/invoicing versus payment — RECONFIRMED
**Package purchase and purchase-invoice creation happen together atomically.** The **collection of money is separate**: the invoice can subsequently receive no payment, partial receipts or full payment, according to the normal payments workflow. Having an invoice does not mean the package is paid. Whether an unpaid or partly paid package may *redeem* visits remains a **separate owner-pending policy**, not settled here.

### D05 — Closing a package never silently settles its invoice — DECIDED
Manual closure **stops future package coverage** but **does not cancel the original purchase invoice, erase the unpaid balance, create a refund, or waive money**. The invoice remains with its current financial state until a separate authorized action. Any clinic-versus-patient money adjustment is for the admin to assess, **not an automatic prorated refund/amount-due calculation by the package module**.

### D06 — Financial follow-up after closure — DECIDED DIRECTION
Immediately after closure, show a **factual closure summary**: purchased price, amounts collected, current invoice balance, elapsed time/days, used visits and relevant package dates. Offer navigation to **the exact purchase invoice** (for collection or an existing authorized final-waiver action) and, when the separate feature exists, to **the refund workflow**. The admin chooses any financial resolution. These actions **are not prerequisites** for closing the package; closing may finish while financial follow-up remains outstanding. The package system should not label any amount as a computed refund entitlement. Existing invoice-waiver behavior is **final waiver of the residual**, not an approved partial-refund calculator.

### D07 — Exhaustion means Completed, not Closed — DECIDED / RECONFIRMED
A ten-visit package becomes **Completed** upon its 10th genuinely covered visit; a long package becomes **Completed** upon its 26th covered visit. Completion is automatic and **does not ask for a manual closure reason**. **Closed** is a distinct, deliberate termination action before exhaustion.

### D08 — Manual closure requires a recorded reason — DECIDED / RECONFIRMED
A closure reason is **mandatory** for admins and for the future receptionist role. User wants **predefined selectable textual reasons plus a custom Other reason**, not a hardcoded reason enum. Selecting a standard preset is sufficient as the recorded reason; choosing **Other requires custom explanatory text**. Stable internal preset keys for reporting were suggested but **no exact implementation** was approved.

### D09 — Final agreed closure-reason presets — DECIDED
1. **Duplicate Package Purchased**
2. **Wrong Package Registered**
3. **Patient Stopped Attending**
4. **Patient Requested Termination**
5. **Patient Deceased**
6. **Doctor Advised Stopping Treatment**
7. **Other — written reason required**

The following candidates were explicitly rejected or replaced: *treatment completed early* (that is Completed), *clinic discontinued service* (a single service being unavailable does not end the treatment package), vague *clinic-initiated closure / administrative correction* labels, and the jargon *medical contraindication*. **Doctor Advised Stopping Treatment** refers to a doctor recommending the entire treatment course stop, not merely changing one therapy modality. A patient request may justify closure **without implying refund eligibility**.

### D10 — Special constraint on “Patient Stopped Attending” — DECIDED
Keep the **short preset label** exactly as above. It may be selected **only after the package's review date has passed** (when **Review Overdue** begins), **not** on review day or earlier. **No additional minimum number of days since the last visit** is required. Disable/hide the option in the UI before eligibility, and **enforce the same date restriction on the backend**. The system can validate the date; it cannot independently prove why a patient stopped attending. Reaching review/overdue **does not automatically close a package**.

### D11 — Closure permissions — RECONFIRMED
**Admin** may close an Active package **at any time**, with a reason. A **future receptionist** may close only when the package is **Review Overdue**, i.e., **starting the calendar date after the review date**, also with a reason. Current clinic software operation is **admin-only**; receptionist permissions refer to a planned role. This actor restriction is separate from D10's restriction on one particular preset: even an admin cannot choose **Patient Stopped Attending** before overdue, under the newly selected rule, but may use a truthful other reason when permitted.

### D12 — Review date is not hard expiration — RECONFIRMED
A package with remaining credits **stays Active after its review date** unless completed or deliberately closed. Review day gets the previously settled small informational warning, and Review Overdue permits discretionary further coverage subject to the established warnings/allowance rules. There is **no automatic termination solely because a patient has not returned**.

### D13 — No simultaneous eligible packages for the same complaint — DECIDED / CLARIFIED
The final choice **overrides the assistant's queued-renewal recommendation**. **Do not allow purchase of a new package for a complaint while that complaint has an Active package.** No queued/advance package sale and **no automatic future activation of a previously sold replacement**. The user rejected the queuing model explicitly. A new purchase can be made only after the existing same-complaint package has **Completed** or been **Closed**. Merely crossing the review date is **not enough**, since the package remains Active. This restriction is **per complaint**, not across the entire patient; packages for different complaints are independent. The proposed scenario of a second package starting after an existing one ends was discussed, but **not adopted as a queued-purchase feature**.

### D14 — No automatic package picker needed for overlapping entitlements — DERIVED, NOT A NEW BUSINESS POLICY
Under D13, two same-complaint packages cannot both be eligible for redemption. Therefore the ordinary checkout should not prompt reception to choose between two simultaneously active same-complaint packages. Exactly how checkout detects and locks the eligible purchase remains an **engineering check**.

## B. Subjects discussed, but NOT finalized

### O01 — Owner payment-enforcement rule — OPEN / OWNER
Can a registered package cover visits with **₹0 received**, partial payment, or only full payment? If a payment pause/grace rule is approved, what precisely triggers and releases it? The **review clock's purchase-date start** was already established in the reference documents; this session did **not** choose an entitlement-payment gate. Keep purchase, invoice status, review clock and redemption eligibility distinct.

### O02 — Original “one month” versus fixed 30-day review — OPEN / OWNER
The existing long-offering implementation direction uses 30 consecutive calendar dates; explicit owner acceptance of this rather than the originally described calendar month remains to be obtained.

### O03 — Same-day completed/closed package replacement — OPEN
The user settled **when a new package may be purchased** (only after the old package has ended), **not** every coverage detail of a new package purchased on that **same day**. Does a new long package cover a second genuine treatment already following a covered treatment on the same date? Does replacing a 10-visit package with a long package differ? A same-day cross-package daily-limit rule was raised but **not decided**. Do not infer one from the no-queue policy.

### O04 — First treatment before package registration — OPEN TECHNICAL RULE
The opening handover explicitly raised the clinic's real workflow: treatment may finish first; purchase is registered at the counter; then the visit is logged. The intended treatment should not be mistakenly rejected because its real-world treatment time precedes the purchase timestamp. Exact checkout transaction/eligibility behavior was **not newly finalized here**.

### O05 — Reopening and corrections after closure — OPEN / DEFERRED
“No routine reopening” was **suggested**, but never affirmatively adopted. Whether a Closed package can be reopened was **not settled**. Generic visit/package/invoice voiding, retroactive correction, refund calculations and reversal cascades remain **future work**, not additions to this MVP.

### O06 — Mistaken package and financial settlement — WORKING DIRECTION / OPEN INTEGRATION
**Duplicate purchase** and **wrong package registered** are approved closure reasons. Previously described handling is closure followed by a separate invoice waiver, where appropriate. But waiver (forgiving a legitimate billed remainder), true void (invalidating an erroneous bill), and refund (returning money) have different meanings; the exact money-handling integration was **not finalized**. No refund module was built or specified here.

### O07 — Post-review justification notes and alerts — OPEN PRODUCT / DEFERRED
The prior document's rule remains conditional: **if** a justification note for continuing overdue covered visits exists, it would be mandatory for reception and optional for admin. Whether to include it in MVP remains undecided. Query-derived admin Needs Attention and a two-stage review warning were background settled directions; persistent dismissible notification infrastructure remains deferred. Do not confuse a **reason to close a package** (D08–D10, mandatory) with a **reason to continue overdue coverage** (not yet required).

### O08 — Other offering specifics and legacy migration — OPEN
Exact package prices, final purchased-terms snapshot schema, older patient/package migration, complaint closure interaction, and catalog edit permissions/DDL for package offerings still need design or audit as noted in the opening handover. They were **not independently settled** in this voice session.

### O09 — Checkout/redemption safety — ENGINEERING CHECK
Verify one package-to-visit record of truth, transactional 10th/26th credit exhaustion, the long offering's already-confirmed one-covered-visit-per-day policy, idempotency against double-click/retry, zero-covered-line invoice integrity, mixed-invoice treatment and legacy-schema compatibility. **Do not impose a blanket one-real-visit-per-complaint-per-day uniqueness constraint:** the referenced policy allows a real second treatment even when it is not package-covered. No specific SQL/RPC implementation was approved in this conversation.

## C. Explicit exclusions and interpretation boundaries

- **No queued package purchases**, despite earlier exploration of stacking and delayed activation.
- **No general editing or deleting** committed visits/payments/invoices in this MVP; future voiding is distinct.
- **No automatic refund or debt forgiveness** merely because coverage was closed.
- **No automatic package closure or loss of remaining credits on review day**.
- **No automatic inference of refund eligibility** from any closure reason, including patient death or doctor advice.
- **No automatically generated missing-visit, absence, holiday or grace-buffer engine** based on the brainstorm.
- A **preset closure reason** is a staff declaration subject to permission and date checks, not independently verified medical or attendance evidence.

## D. Next decision order (without reopening closed issues)

1. Obtain the owner's answers on **payment gating** and **30-day commercial wording**.
2. Decide **same-day replacement coverage** across successive packages, now that advance sales are forbidden.
3. Resolve the remaining small product boundaries: **package reopening**, post-review justification note, and mistaken-invoice handling.
4. Perform the **database/transaction audit**, entitlement-source tests and legacy package migration review before implementation.

**Record provenance:** This file documents this chat only and is not a GitHub commit. Earlier package handover/addendum/patch remain historical evidence, not rewritten by this file.
