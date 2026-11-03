# Package Operations — Call Transcript & Follow-up Discussion Log

**Recorded:** 03 November 2026, 22:31 IST (UTC+05:30)  
**Scope:** Only the call-recording transcript pasted in this conversation and the discussion that followed it. This is **not** a consolidation of earlier addenda or a replacement for the existing Package Module specification.  
**Evidence standard:** The transcript is unreliable in places. Statements below are classified as **confirmed**, **illustrative**, **rejected/deferred**, or **open**. A proposed design is not an approved requirement.

## 1. Context and language — confirmed

- Participants were **the software designer (user) and the clinic owner**; both proposed business scenarios. The topic was general clinic operating practice, not a special rule for one location.
- The recording uses “10 days” informally for the **10-covered-visit offering**, with Sundays excluded from the relevant 10-day count; “one month/30 days” refers to the **26-covered-visit offering associated with a 30-day period/review**. These expressions must not be converted into a new calendar-expiry rule.
- Figures such as **₹1,800 for Routine 10, ₹200 per Routine visit, and ₹300 per Rehabilitation visit** are examples of the clinic's current pricing, **not hardcoded software values**. Prices are clinic-configurable. Other unclear numbers in the transcript are not reliable enough to reconstruct transactions. The recorded use of “free” was not established and produces no requirement.

## 2. Package sale, partial collection, and unpaid balances — confirmed

- Partial payments occur fairly frequently. Even if a patient initially pays only ₹2,000 toward a 26-visit package, the system records a **purchase of the complete 26-visit package**, an invoice for the full configured price, the actual ₹2,000 payment, and the remaining invoice balance. There is no formal “intention to buy” or provisional mini-package in the system.
- The purchased package has its **full configured visit entitlement** in software. Neither partial payment nor an internally discussed payment deadline automatically reduces coverage, blocks visits, closes the package, or changes its offering. **No fixed software payment deadline** is needed.
- The clinic owner may **manually assess the economics** of a partially paid or discontinued package: value the first 10 used visits at the configured 10-visit package price, then value subsequent visits at the appropriate ordinary per-visit rate. Whether to continue treating, insist on payment, charge separately, close the package, or settle a balance remains a **deliberate clinic decision**, not an automatic entitlement algorithm.
- **Illustrations only, not automated rules:** ₹2,000 can be understood internally as ₹1,800 for 10 visits plus ₹200 for one more visit (11 visits). ₹3,000 can be understood as ₹1,800 plus six ₹200 visits (16 visits). ₹3,600 is **not** automatically two 10-visit packages; the second ₹1,800, under the owner's described method, would cover nine ordinary ₹200 visits. The software nevertheless continues to recognize the originally purchased package until an explicit lifecycle action.
- If the clinic permits a 17th package-covered visit despite a ₹3,000 partial payment, **that is allowed**. Staff see outstanding-amount indicators in relevant patient workflows and decide whether to continue. There is no need for software to enforce the owner's manual arithmetic. If coverage should stop, staff can **explicitly close the package**, subject to the closure rules.
- **Invoice amounts are immutable.** A promise to pay is not a payment; actual collections are recorded as payments, unpaid balances remain due, and a deliberate write-off is recorded through **waiver**, not by reducing the original charge. Package closure does **not** automatically waive the invoice or initiate a refund. There is no automatic nonpayment closure and no automatic reopening upon subsequent payment.

## 3. Clinical category and complaint handling — confirmed

- **Routine and Regular are the same category; Rehabilitation is a separate category.** Each complaint is classified **when that complaint is created** by the staff member registering it (including receptionist or admin). The classification is **immutable** for that complaint and determines the applicable treatment/pricing category.
- There is **no mid-course Routine ↔ Rehabilitation category conversion** and no cross-category package conversion. If a different classification is clinically required, the route discussed is to **close the old complaint and open a new complaint**, respecting any applicable existing closure restrictions.
- Fracture/surgery history, “three to six months,” and “one year” were clinical illustrations, **not software cutoffs**. The late-disclosure and ₹200+₹300 examples were hypothetical/irrelevant to billing design. The clinic determines how to classify a later, newly registered complaint; the software does not infer it from clinical history.

## 4. Early termination, settlement, and refunds — confirmed

- An accident interrupting treatment after approximately 12–15 visits was cited as a **real type of case**. Whether a patient receives a refund, and the amount, are **case-by-case clinic decisions**.
- The owner described a possible **manual** settlement calculation: charge the 10-visit package price for the first 10 visits, then the applicable ordinary rate for extra visits; consider actual visits used and the circumstances. Fewer-than-10-visit cases are also discretionary: the clinic may allow remaining visits, decide on a refund, or otherwise settle. **The software must not calculate or guarantee refunds using this example.**
- The financial/lifecycle steps remain distinct: staff explicitly **close** a package when appropriate, and handle **refunds, payments, and any residual invoice waiver separately**. The original financial records are not silently rewritten.
- **Refund authorization: admin only** (explicit answer to T73). This is an authorization rule; it does not create an automatic refund formula.

## 5. Package upgrade — proposed, NOT yet approved

**Real request:** A patient who bought a **10-visit Routine or Rehabilitation package** may later ask to pay the difference and obtain the corresponding **26-visit package**. The clinic sells both existing offerings, but there is **no current “upgradable package” product**. Only **same-category** upgrade is contemplated (Routine 10 → Routine 26; Rehabilitation 10 → Rehabilitation 26). A normal new 26-visit sale is **not equivalent** to an upgrade.

**Intended economics and entitlement if approved:**

- Total allowance becomes **26 visits including those already consumed**, i.e., a net increase of **16**, **not** 10 + 26. If five visits were used, they remain counted in the total 26.
- The intended additional charge is **the difference between the relevant 26-visit and 10-visit offering prices**, recorded as a genuine additional invoice charge. Past payments remain attached to their original invoice: **a waiver is not a payment transfer**. How to price an upgrade when catalog prices have changed remains open.
- The original package's **coverage-start anchor** should govern the longer package's **30-day review milestone**. Example discussed: original start **1 November**, upgrade requested around **11 November**, longer review milestone **30 November**, not a newly restarted 30-day period. **Review is advisory, not entitlement expiry**; precise date/Sunday-count rules need validation during design.

**Design alternatives discussed:**

1. **Modify original package's entitlement:** Appears simple for an active package, but mutates a purchased entitlement and is incompatible with reactivating a terminal Completed package without changing earlier rules.
2. **Linked upgrade record / dependent “upgrade package” (user is inclined toward this):** Initiated **from an eligible existing 10-visit package**, with a required parent-package ID and matching complaint/category. Adds only the needed allowance within a combined 26-visit entitlement, preserves the original review anchor, and creates an auditable extra charge. This is **not independently purchasable**. Original Completed/Closed history must not be rewritten or reactivated.
3. **Ordinary standalone 26-visit purchase with discount/waiver:** Rejected as an equivalent upgrade because it grants a new 26 visits and new review anchor, and misuses waiver to simulate credit. A fifth independent 16-visit catalog product is also **not sufficient by itself**; it would require the same parent-linkage and coverage rules.

**Proposed engineering safeguards, not individually approved implementation details:** deny standalone upgrade creation, duplicate upgrades, mismatched category/complaint, and double-counted visits; ensure entitlement and invoice operations are consistent. These are **constraints** to design around, not evidence that upgrading is impossible.

**Still open:**

- **Eligibility after completion:** upgrade only while original 10-visit package is Active, **or** also after it becomes Completed? (The next question awaiting the user's answer.) A Completed/Closed package is terminal; the suggestion to revert Completed → Active **was not approved**.
- If post-completion upgrades are permitted, is there a time/window limit, and how does the linked extension become usable while preserving terminal history?
- Final architecture, authorization to initiate upgrades, catalog-price snapshot/difference rule, exact dates/review computation, and **MVP inclusion** are not finalized. The user is receptive **if reasonably bounded in complexity**, not committed to shipping it yet.

## 6. Freeze/pause — explicitly rejected for MVP

- **No package-freeze feature in the current release.** The user sees no practical benefit: passing the review date does **not** destroy unused covered visits, and the clinic can still allow eligible visits afterward. Freezing can be reconsidered only if a distinct future requirement arises.
- Therefore the original freeze questionnaire **T77–T86 requires no further answers** for MVP.

## 7. Questionnaire disposition and remaining work

- The original **T01–T90** questionnaire was judged overly repetitive; no need to answer each prompt mechanically. Context, payments, complaints/classification, manual refunds, and freezes have been sufficiently clarified for this discussion.
- **T37–T41** (garbled individual transaction figures) and questions seeking a clinical decision algorithm or software refund formula were set aside as irrelevant. **T42–T43** were answered by immutable invoices, actual payment recording, and waiver for forgiveness. **T73** was answered: admin-only refund authorization.
- **T87–T88** (written cancellation terms) did not yield an application requirement; no display/acceptance/enforcement feature was approved. **T89–T90** reduce to the remaining upgrade decision; the existing four offerings are the baseline, without an approved standalone fifth product.
- **Only substantive open Package Module feature from this call:** the **10-to-26-visit same-category upgrade**, including whether it may follow completion and how to represent a linked extension while keeping original records terminal.

**Next discussion point:** *May an upgrade be initiated after the original 10-visit package reaches Completed, or only while it remains Active?* No answer has yet been recorded.
