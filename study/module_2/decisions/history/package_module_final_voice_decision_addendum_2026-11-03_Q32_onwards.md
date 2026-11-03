# Package Module — Final Consolidated Q01–Q40 Decision Addendum and Voice Handoff

**Finalized for this discussion:** 3 November 2026  
**Scope:** This is a standalone, updated continuation of `package_module_Q18-Q40_decision_addendum_2026-11-03.md`. It includes the earlier Q01–Q31 baseline, preserves the exact reconstructed questions and A/B/C alternatives for Q18–Q40, and records *all additional decisions, reversals, clarifications, and scope boundaries made during this voice session*.  
**Status precedence:** (1) the user's latest explicit statements in this session; (2) earlier statements in this session; (3) the prior addendum's earlier voice decisions; (4) prior source/review files; (5) historical questionnaire recommendations. **A historical recommendation is not an approved business rule.**  
**Evidence limitation:** This is a decision/specification handoff, **not** a verified application audit, an executed test report, a code change, or a verbatim voice transcript. Some earlier GitHub documents describe planned architectures and prototype components. They must not be treated as proof that a feature has shipped. The questionnaire wording below was preserved from a previously reconstructed questionnaire, not independently authenticated against an earlier original.

## Executive handoff — read before using the older Q descriptions

**The numbered Q01–Q40 package questionnaire has been discussed to completion for the current scope.** Q32 and Q33–Q36 are **deliberately deferred** to a separately developed and separately charged historical-data-entry module, not permanently rejected as business needs. Q37 is approved as A; Q38 as A; Q40 as A. **Q39's earlier apparent approval for a separate Unpaid Packages category was explicitly superseded by the user's later clarification: there is no separate package unpaid category and no package payment status.** Package-related balances are ordinary **invoice** balances. The broader *Needs Attention* function can show due invoices and clinic-defined warning notes in multiple relevant screens. The complaint-reopening question belongs to the Complaint Module and must not be reopened here.

**Do not ask the user to choose the old options again.** The original options remain preserved below for source integrity, even when the final policy is a custom answer rather than an A/B/C selection. No more package business questions are required merely to satisfy this questionnaire. Development validation can later check conformance with approved behavior without manufacturing new business decisions.

### Final Q18–Q40 disposition register

| Original Q | Current disposition | Exact meaning / caveat |
|---|---|---|
| **Q18** | **Dropped as a package-module decision; complaint-reopening policy unresolved** | Complaint lifecycle independent; packages already Closed/Completed never reopen; active-package complaint cannot close |
| **Q19** | **Decided: B for a single checkout** | One invoice with distinct lines for package, exam and other items in same checkout |
| **Q20** | **Decided: custom / A in simple unpaid case** | Close wrongly created unused package; if its separate invoice has no other charge and nothing paid, waive all outstanding; mixed cases under Q21 |
| **Q21** | **Decided: custom workaround, no line void approved** | Mixed invoice: collect valid charge first then terminally waive rest; if nothing paid, waive full erroneous invoice and recreate a clean invoice for valid items |
| **Q22** | **Decided: refund workflow + waiver + new payment where needed** | Mandatory refund reason; existing Cash/UPI payment methods; no wallet, credit, internal transfer or additional audit facility |
| **Q23** | **Decided: same refund/recreate principle when fully paid** | Close erroneous package, refund paid amount with mandatory reason; if replacement purchased, create new invoice and enter new payment |
| **Q24** | **Scenario rejected/dropped as framed** | Complaint-specific coverage cannot cover the other complaint; no retroactive coverage/reassignment was authorized; general wrong-visit issue is still Q40 |
| **Q25** | **Decided: A** | Valid package stays active after its invoice is fully waived; financial waiver does not revoke entitlement |
| **Q26** | **Dropped as inapplicable** | 'Refunded' is **not** an approved package-closure preset; refunds are separate and have a mandatory reason |
| **Q27** | **Decided: prices clinic-configurable** | No fixed rupee prices in specification; values determined at clinic setup |
| **Q28** | **Decided: A** | Only **admin** may change package catalog prices/settings; only two roles exist: admin and receptionist; no owner role |
| **Q29** | **Decided: A** | Keep configured invoice price; payment collected and subsequent terminal waiver separate; no checkout-time selling-price override |
| **Q30** | **Decided: B** | Even complimentary package carries ordinary listed invoice charge; admin subsequently waives entire amount; not a zero-price sale |
| **Q31** | **Decided: B** | No MVP disable/new-offering/custom-rules UI; package catalog creation/disable/rule builder deferred |
| **Q32** | **Decided: custom — no migration in current module; future work deferred** | Real preexisting paper packages may exist and remain honored on paper. No active-package onboarding/import. Historic digitization is a separately charged future module. Do not select original B or C as fact. |
| **Q33–Q36** | **Deferred, not out of scope forever** | Historical entry method, incomplete records, older terms and old financial balances are concerns for a future separately scoped and priced historical-record-entry module. No current import and no preapproved A/B/C implementation. |
| **Q37** | **Decided: A** | Package covers only flat eligible complaint-linked visit charge; separately billed standalone treatments/services remain chargeable. |
| **Q38** | **Decided: A** | Nonblocking informational warning when staff choose a complaint different from one with an active package; correct complaint-driven billing continues, no mandatory acknowledgement. |
| **Q39** | **Decided: custom — latest clarification supersedes provisional A** | No package Paid/Unpaid field and no dedicated Unpaid Packages category. Invoice dues appear in general Needs Attention ordered by amount outstanding; clinic warning notes are separate attention signals. |
| **Q40** | **Decided: A** | Committed erroneous visit, invoice and package redemption remain unchanged for MVP; rely on thorough precommit verification. No generic visit correction or void/edit mechanism. |


### New in the 3 November 2026 voice session — complete change log

**1. Starting scope and questionnaire flow.** The conversation resumed from the earlier 465-line Q18–Q40 decision handoff, initially asking Q32 and subsequently examining Q33–Q40 one by one. The owner repeatedly insisted on exact original numbering and on not reopening decided questions. Several assistant attempts to shift from business decisions into repository audits, repeat Q38, or revive complaint-reopening questions were rejected. These detours did **not** approve new features.

**2. Q32 — paper packages exist, but no opening migration.** The clinic may have legitimate earlier package purchases documented on paper. The owner wants the new software to **start fresh**, without entering or importing those existing active records at launch. Preexisting patient commitments are still to be honored operationally on paper until fulfilled. This is a **custom decision**; it is not evidence that old active packages do not exist, and it is not authorization to discard real obligations. Earlier assistant simplification "Q32 = no historical packages" would be wrong.

**3. Q33–Q36 — future paid historical record entry, not current active package onboarding.** The owner further specified that any future digitization would concern **legacy historical information: things already done**, not converting running paper package entitlements into live new-system packages. This is **retrospective/manual data entry** from notebooks or paper, requiring a **separate module build and separately charged work**, including substantial manual transcription. During this conversation the owner corrected the terminology **"deferred, not out of scope"**. Accordingly, no import mechanism (Q33 A/B/C), estimate policy (Q34 A/B/C), contract conversion scheme (Q35 A/B/C), or old-debt migration method (Q36 A/B/C) has been designed or approved for that future engagement. Existing old balances remain in historical external records for the current rollout. The future project can define which legacy facts and balances to enter when commissioned; do not imply that it automatically imports active running packages. **No historical transactions should be misrepresented as new launch-day package sales.**

**4. Q37 — precise coverage unit.** The owner expressly explained that a package pays **only the standard flat visit charge** for an **eligible covered visit linked to that package's complaint**. Procedures included *within* that ordinary visit are part of the visit, not additional separately charged standalone sales. Anything **outside the covered visit**, if separately billed as a standalone service/treatment, remains independently payable. Original option **A** is approved. Existing separate exam, lapse and specialty charge rules in the earlier specification are not implicitly cancelled by the word "visit"; do not broaden package zeroing to unrelated money buckets.

**5. Q38 — selection of another complaint: A, not B.** The owner first challenged whether this had already been settled and demanded a review of the two GitHub repositories. The previously agreed **complaint-specific package eligibility** was confirmed as an architectural rule, but that rule alone **did not decide the warning UI**. The owner was later asked the **actual original Q38** again and explicitly answered **A**. Implement/show an **informational, nonblocking notice identifying the other active complaint-linked package** when a different complaint is selected. Continue ordinary correct billing of the selected complaint; do not transfer allowance; do not require C's explicit mandatory acknowledgment. A prior assistant statement that no warning was approved was superseded by this explicit A answer. Documentation and prototype status cannot establish that the warning is implemented.

**6. Q39 — provisional "separate unpaid package section" was corrected and superseded.** Earlier in voice, when asked whether unpaid packages should appear in Needs Attention, the owner initially said yes, and the assistant incorrectly recorded "dedicated Unpaid Packages section/category." The owner later gave the definitive model: **packages have no Paid/Unpaid tag**; the **invoice** is the financial object carrying payment status and outstanding balance. **Needs Attention** contains a general **due-invoices** set, ordered by **amount the patient currently owes, descending**. Packages normally have a high sale price and may naturally rise high in this ranking, **but they are not a separate category and have no special classification or priority**. A ₹100 invoice residual still appears, even if originally linked to a package; there is **no minimum amount threshold**, and it remains while any amount is owed, unless subsequently **fully settled by payment or waiver**. Never try to attribute a combined invoice's balance to a specific package line merely for this ranking. Sorting by actual remaining amount means a nearly settled package invoice can rank below a smaller unpaid service invoice; this follows the owner's criterion.

**7. Needs Attention is broader than dues.** The owner also clarified that Needs Attention is a **general-purpose clinic attention/notice mechanism**, not a package-debt report. It can surface **clinic-decided/custom warning notes** about a patient, for example behavior-related or any other warning the clinic finds relevant, along with due amounts and other agreed attention signals. Such information may surface **in multiple relevant locations within the application**, not exclusively in the dashboard. The precise UI/reporting arrangement is **for later design**; this remark does **not** authorize a specific new patient-flag database schema, automation, reminder scheduler, extra role, custom rule engine, or global audit subsystem. Existing Q09–Q10 package review attention concepts are not erased; the specific idea of a distinct *Unpaid Packages* payment category is rejected by the latest clarification.

**8. Q40 — committed wrong visits left unchanged, option A.** An example was explained: staff accidentally select shoulder when treating knee; the visit, invoice, and a package redemption have already been committed. The owner decided to **leave committed records as they are in MVP** and rely on substantial verification **before creation/commit** to reduce mistakes. No visit attribution edit, retroactive package redistribution, generic invoice void, line void or new postcommit correction workflow is approved. This maps to original **Q40 A** (investigate manually; committed record unchanged). Q40 A's phrase "flag for manual investigation" must **not** be expanded into an unapproved technical flagging feature. Likewise, the assistant's mention of a "clear confirmation step" is a high-level verification expectation, not approval of a specific new multistep confirmation UI. The owner explicitly accepts that prevention does not technically guarantee zero mistakes.

**9. Complaint reopening — removed from this discussion.** After Q40 the assistant raised an earlier separate question about whether a clinical complaint may reopen. The owner explicitly rejected discussing it in a package review: it belongs to the **Complaint Module**. The cross-module invariants still stand: a complaint with an active package cannot be closed, and terminal Closed/Completed packages never reactivate. No complaint reopening policy was chosen, and it must not be reintroduced as a remaining package gap.

**10. Separate earlier ideas remain exactly deferred.** Future invoice/line void or reversal (**DR-01**) remains a **future review idea**, not MVP scope. The existing waiver still settles an invoice's **whole remaining due** (**DR-02**). Admin-configurable prices for four fixed offerings are already approved, while a broader offering-creation/disable/rules builder remains deferred (**DR-03**). The independently planned refund module requires reasoned Cash/UPI refund records; it was **not verified to be deployed** during this conversation. New paid historical-entry work is an independent future proposal, not bundled into this Package Module.

**11. What the GitHub check did and did not establish.** The owner confirmed that **only two repositories are relevant**: `omsrivastava512/clinic-demo` (frontend UI catalogue/prototype) and `omsrivastava512/clinic-demo-be` (backend specifications/code). Files examined during this voice discussion included `clinic-demo/src/features/visit/ComplaintSelector/index.tsx`, `clinic-demo-be/study/business_logic/packages_logic/README.md`, `.../DEFERRAL.md`, `clinic-demo-be/README.md`, and frontend workflow documentation. The backend material documents complaint-ID-based entitlement matching and isolated therapy-fee handling, and the frontend complaint selector was checked; however, **no complete implementation audit, deployment verification, test run, financial reconciliation audit, or proof that Q38's warning exists in the built UI was performed**. Some older repo documents propose alternate or superseded package architectures; later owner decisions in this file govern business requirements. The owner rejected confusing code inspection/implementation checks with still-unanswered owner decisions.

**12. "Engineering verification" means future acceptance testing, not a new questionnaire.** Previously mentioned examples—four prices configurable by admin without an offering builder; accurate refund + full-residual-waiver balance arithmetic on a mixed invoice; fully paid/waived balances absent from open dues while ₹100 residual remains visible; correct verification and nonblocking cross-complaint notice—are **checks against existing approved decisions**, not new requirements and not questions to ask the owner now. Nothing in this session verified that those features are already implemented. A practical handoff may list them as conditional developer acceptance checks, but must not present them as pending business-policy choices.

**13. Final scope boundary and terminology corrections.** No patient portal. Only **admin** and **receptionist** roles. No wallet, inter-invoice credit/transfer, line-level waiver, full invoice edit/void in MVP, or dedicated package payment status. Historical backfill is **deferred and separately priced**, not categorically out of scope for future business. Clinic-defined notes and invoices can drive broader attention display without implying package status. Retain exact Q-number-to-subject mapping. Distinguish **user-approved business requirements**, **proposals/deferred concepts**, and **actual verified implementation**.

### Approved cross-cutting facts (needed to interpret answers)

- **Product and roles.** This is **internal clinic software**, not a patient-facing portal. Only **admin** and **receptionist** roles exist. References to an "owner" role in legacy question scenarios are hypothetical wording and **do not create a third role**. The application keeps clinic operational, invoice and financial records.
- **Package catalog.** Four current offerings: Routine 10 visits, Routine 26 visits, Rehabilitation 10 visits, Rehabilitation 26 visits. The numbers ₹4,500/₹350 in examples are illustrative, not approved prices. Prices are clinic-configurable; admin is the only role authorized to modify them. The owner has **not** approved a self-service UI to create/disable arbitrary offerings or set new package rules in MVP; that is future scope. Admin's price-setting authority versus the exact MVP setup/edit interface remains an implementation detail; do not silently add a full catalog builder.
- **Complaint binding.** Each purchased package is attached to one specific complaint. A shoulder package never covers a knee-complaint visit, and vice versa; the selected complaint determines package eligibility. A wrong-complaint visit is ordinarily billed at the normal rate if that complaint has no eligible package. A complaint with an **active** package cannot be closed. Future complaint reopening rules have **not** been decided, and no complaint action resurrects a terminal package.
- **Entitlement separate from money.** Registering/purchasing a package creates entitlement independent of ₹0/partial/full payment, subject to its start-date eligibility. A later invoice waiver does **not** close or impair a valid package (Q25). A package deliberately Closed is irreversibly terminal and forfeits unused visits; Completed is also terminal. Payment/refund/waiver operations do not reopen it. Debt alone does not close or suspend valid coverage.
- **Purchase/start dates and review.** Purchase date and **package start date** are separate. Replacement package starts the same day if the old same-complaint package did not cover an earlier visit that day, otherwise the next clinic-local day. The review clock begins on package start date. 10-visit review counting excludes Sundays as previously specified; 26-visit review window is 30 calendar days. Review is advisory, not automatic allowance expiry. Ordinary default covered-visit cap is 1 per clinic-local day per purchased package; admin can change this specific cap at sale or later. Do not build a new audit facility just for cap edits.
- **Invoice model.** **One checkout gives one invoice** with a separate line for each package/service/examination. Different checkouts need not be consolidated. Invoice charges are the configured price; package-sale price is **not** overridden at checkout. Existing waiver is **invoice-level and terminal**: it settles *all currently outstanding amount*; it is **not a selectable partial or line-level waiver**. Closing an erroneous package does **not** automatically waive its invoice. Payments and refunds are separate records.
- **Internal invoice screen/presentation.** The owner said the clinic's invoice view should show the **original invoiced/billed amount and the amount currently due**; payments and waivers are handled in their own records/workflows, rather than being a user-facing/patient portal. This came from an assistant's *misnumbered, non-questionnaire* Q30; preserve the substance as a cross-cutting instruction, **not** as the answer to original Q30.
- **Payment/refund.** Payment methods presently discussed are **Cash and UPI only**; no built-in patient-wallet/credit-balance/transfer method. A refund module is **planned/being built** (owner's explicit clarification later than the source document's 'not added' baseline); do not state it is already deployed. Every recorded **refund has a mandatory reason** explaining why, e.g. erroneous package, overcharge. The owner accepts recording a cash refund against the old invoice and a separate cash payment against a corrected invoice, without a credit wallet, 'reallocate payment' module, or verification of momentary hand-to-hand cash transfer. Do **not** reintroduce the assistant's repeated gross-cash-reporting warning as an approved decision or impose extra audit requirements. The owner considers the recorded entries and mandatory reasons sufficient for the intended workflow.
- **Waiver and refund distinction.** Waiver disposes of a receivable/current amount due; a refund records return/reversal of previously registered payment. Full waiver after a correctly created complimentary package affects *only* receivables. A paid mistaken purchase uses a refund (not waiver alone). No generic true invoice/line void subsystem, partial waiver control, or general clinical-visit correction has been approved merely by talking about examples.
- **Authorizations and reasons.** Package closure requires its existing mandatory reason and irreversible confirmation (show exact patient, complaint, package, unused visits and forfeiture). Refund reason is mandatory **for each refund**. A package-closure reason of 'Refunded' is **not** in the approved preset list; refund reason belongs to the refund transaction. Submitted post-review covered-visit notes are immutable. No new application-wide audit framework, cash-movement verifier, future scheduled alert engine, or new patient role was approved.
- **Latest clarification: Needs Attention and invoice debts.** There is **no package-specific payment status, balance, or Unpaid Packages Needs Attention category**. Invoices alone carry payments, waivers, and residual debts; general due-invoice attention is sorted by remaining balance, descending, with **any positive outstanding amount visible**, including ₹100. Packages are frequently high-ticket, not categorically prioritized. Clinic-selected patient warning notes, including behavior-related notices, are distinct reasons to draw staff attention; both notices and relevant dues can appear in multiple parts of the application. Exact reporting display remains to be designed; neither an additional warning rule engine nor a dedicated package-dues report is approved.
- **Additional cross-complaint UI rule, Q38 A.** When a different complaint is selected despite another active complaint-linked package, show **nonblocking** information about the other package; ordinary complaint-specific coverage/billing continues, with no required acknowledgement. Do not confuse this UI warning with a package transfer or retroactive coverage.
- **Committed clinical entry, Q40 A.** No editing of already committed visit/attribution/invoice or visit-void capability in the MVP. Staff verify details before committing; erroneous committed rows may be investigated manually and remain unmodified. Do not automatically invent a new error-flagging subsystem.
- **Historical paper records, Q32–Q36.** Old packages and their rights can exist outside the software; prelaunch live packages remain on paper, and historical data entry is a separately quoted future build/labor task. Do not declare that historical records do not exist or create launch-date fictional purchases.
- **Additional extra rule from an assistant's wrong Q31:** When asked separately whether sale should be blocked if a package price was not configured, the owner explicitly answered **yes, block**. This is a cross-cutting validation requirement, **not** original Q31 and not authority to implement a full catalog-management UI.
- **No retroactive coverage.** Asked separately after an incorrectly framed visit-misattribution hypothetical, the owner answered **No** to turning a previously billed visit retroactively into a package-covered visit. This does not automatically answer original Q40 about an actually erroneous committed record.

### Decision examples and business reasoning — retain these in future discussions

**Example 1 — single erroneous package invoice, unpaid.** Staff accidentally register a ₹4,500 package. No visits used, no payment, and the invoice has **only that package line**. Admin closes the erroneous package, recording the closure reason, and terminally waives the entire ₹4,500 invoice. The closed package is never reinstated. The waiver is a **separate financial step**, not an automatic side effect of package closure. The owner supported this simple use of the existing waiver mechanism (Q20).

**Example 2 — mixed invoice; exam fee already collected.** One checkout creates a ₹4,850 invoice: incorrect package ₹4,500 plus valid exam ₹350. If ₹350 has actually been recorded as paid, the remaining ₹4,500 can be terminally waived after the wrong package is closed. The existing all-remaining waiver is sufficient here; no per-line waiver is needed. This logic reflects Q19/Q21 discussion.

**Example 3 — mixed invoice; nothing collected.** Same ₹4,850 invoice, **₹0 paid**. Existing waiver cannot forgive only the erroneous package ₹4,500 while retaining ₹350 due. Owner's ultimate practical decision: **close erroneous package; waive the whole original invoice; create a fresh correct invoice** for the legitimate ₹350 examination (and any other genuine lines). That replacement invoice carries ₹350 due. The owner used 'void' colloquially while explaining this, then specifically selected **full invoice waiver + new invoice** rather than requesting a new true invoice-void engine. This later answer **supersedes** the earlier historical review log's statement that the mixed case was left with no workable resolution.

**Example 4 — mixed invoice partially collected, corrected package required.** Original invoice has mistaken ₹4,500 package and valid ₹350 exam, total **₹4,850**. Suppose **₹1,500** is registered as payment. The legitimate examination consumes ₹350 of that amount, leaving **₹1,150** economically attributable to the erroneous package. Owner's proposed/accepted workflow: close erroneous package with reason; use the **refund module** to record a **₹1,150 refund against the original invoice** with a mandatory reason; **waive the erroneous package balance / all remaining amount on the old invoice** so it is terminally settled; create a **new ₹4,500 invoice for the correct package** and record a **new ₹1,150 payment** on that invoice (Cash, as illustrated). The owner expressly rejected building patient credit or an inter-invoice transfer system for this. Exact low-level refund/waiver ordering must leave the original invoice settled and the valid exam charge retained as paid; the transcript did not prescribe a database transaction order. On conventional net-payment arithmetic this can be represented as net paid ₹350 and residual ₹4,500 waived on the old invoice, with ₹1,150 newly entered against the new invoice. **This is an acceptance-calculation check, not a new business question.**

**Example 5 — full erroneous package payment.** Patient paid the entire **₹4,500** on a wrongly created package invoice. Admin closes the wrong package and records a **full ₹4,500 refund, mandatory reason**. If the right package is then purchased, a fresh full-price invoice is created and a **new ₹4,500 payment** recorded against it. There is no outstanding debt to waive on the original paid invoice unless other real charges exist. Owner explicitly confirmed this for Q23.

**Business rationale voiced by owner:** do not fragment the MVP into special exceptions for every possible incorrectly registered package, visit, examination or service. The current invoice-level terminal waiver plus planned reasoned refund and ordinary new invoice/payment can deal with the discussed cases. The clinic's bookkeeping relies on the recorded financial operations and their reasons; the owner does **not** want a separate patient credit wallet, internal payment-reallocation feature, cash-handover verification procedure, or general new audit infrastructure. The owner distinguished **closure of package entitlement** from **waiver of invoice debt** and **refund of already collected money**, and insisted a legitimate charge remain billable through fresh invoicing when the original mixed invoice was wholly waived.

### Follow-up audit of decisions — owner-requested review log, NOT application audit logging

**DR-01 — Future generic invoice and invoice-line void/reversal. Status: DEFERRED IDEA, NOT APPROVED FOR MVP.** The owner asked that the proposal be retained **for later personal review/audit of the decision to defer it**. The idea is to add a **generic** mechanism to void/reverse an entire issued invoice or individual invoice line, applicable across packages, visits, examinations and standalone services, instead of adding one-off corrections. Motivation: a mixed invoice with erroneous package ₹4,500 and genuine exam ₹350 could be corrected at the line level, retaining only the genuine ₹350 charge without fully waiving and recreating the invoice. The owner weighed this against the proliferation of exceptions and accepted the existing MVP workaround. The user explicitly said he may audit/reconsider these deferrals later; that is **a document-level future decision review**, not approval to implement a general audit-log table. No full void/line-void mechanism is considered approved today. The technical characteristics of a possible future implementation were not specified by the owner and should be researched only if reopened.

**DR-02 — Existing waiver limitation. Status: ACCEPTED MVP BEHAVIOR.** A waiver applies to the **entire remaining invoice balance** and terminally settles the invoice. It does not currently support an arbitrary partial-value or single-line waiver. This is why a wholly unpaid mixed invoice is waived completely and valid charges re-invoiced. Do **not** silently turn waiver into a partial waiver while implementing Q21 or Q29.

**DR-03 — Potential overlap in setup language. Status: NOTE FOR FUTURE IMPLEMENTATION REVIEW, not another owner choice now.** Q27 approved configurable *prices* at clinic setup; Q28 restricted catalog price/settings changes to admin; Q31 deferred UI for enabling/disabling offerings, adding offerings and editing package rules. These decisions are compatible if simple seeded offering prices can be configured by admin, while the future extensible catalog/rules builder remains out of MVP. Whether the current implementation already has the required simple price setup/edit screen is **unverified**. Do not infer the full builder from the word 'configurable'.

### Assistant question-drift and decisions NOT to carry forward

The voice assistant repeatedly gave the *wrong* question while retaining the original Q number. This addendum explicitly corrects the record:

- **Original Q24** = mistaken package already covered treatments; it is **not** the mixed unpaid invoice/waiver question. The mixed-invoice solution nevertheless was **actually explained and approved**, so it belongs under Q21 as well as cross-cutting examples.
- **Original Q25** = whether waiving an invoice affects valid package coverage, **answered A**. An earlier question mislabeled Q25 about transferring used visits between complaints was rejected as nonsensical. Do **not** mark the real Q25 as dropped.
- **Original Q26** = hypothetical closure reason 'Refunded', **dropped because that preset does not exist**. The assistant's improvised 'should past visits receive retroactive package coverage' question was answered **No**, but it was **not Q26**.
- **Original Q29** = override price at purchase, **answered A**. Assistant's earlier 'track erroneous registrations automatically / by note / not at all' was **not Q29** and has no owner-selected option.
- **Original Q30** = whether clinic may issue a zero-selling-price package, **answered B**: full-price invoice then waive. Assistant's earlier 'what does a patient see / show waivers' was **not Q30**. User's internal invoice display clarification has been preserved above but is not a patient portal or new Q30 option.
- **Original Q31** = disable offering, **answered B**, with future creation/disable/custom package rules. Assistant's earlier question about unconfigured price was **not Q31**, although the owner independently approved **blocking sale** if price missing.
- **Original Q32** = existing active outside-system packages at launch, **NOT ANSWERED**. Assistant invented a 'nearing visit-limit notifications' question; no notification threshold or 'no notifications' decision was approved.
- **Q18/Q24 invalid hypotheticals:** Neither the original Q18 premise about clinical complaint reopening nor the fabricated implied cross-complaint coverage behavior justifies a new package-reactivation or retroactive attribution feature. Clinical complaint lifecycle decision remains separate. Do not convert the user's rejection of an unrealistic example into blanket approval or disapproval of all clinical correction workflows (Q40).
- **Recommendations are not approvals.** The original author's recommendations beside questions below may contradict later decisions and are deliberately retained as *historical source text*, not controlling instructions. Do not reopen settled choices casually or extrapolate proposed workflows into implemented functionality.

### Earlier Q01–Q17 baseline to preserve alongside this addendum

This addendum focuses on Q18–Q40. For a new conversation, the source's earlier Q01–Q17 decisions are summarised here, so the resumed discussion does not contradict them:

| Earlier Q | Operative rule |
|---|---|
| Q01–Q02 | Package purchase date separate from coverage start date/review start; replacement starts today if no same-complaint covered visit earlier today, else tomorrow; prior overlapping package must be terminal |
| Q03 | Daily covered-visit cap initially 1 on all packages; admin may adjust on a purchased package at purchase or later; no separate cap-edit audit in MVP |
| Q04 | Irreversible closure with patient/complaint/package identity and unused-allowance forfeiture confirmation |
| Q05 | Old unpaid invoice never blocks renewal; show warning and invoice details; debt remains collectible; no special delinquent-renewal report |
| Q06–Q07 | Receptionist gives a reason for **each** post-review covered visit; committed reason cannot be edited by anyone |
| Q08 | No admin Needs Attention/pings for post-review coverage the admin itself authorized; receptionist coverage does get attention |
| Q09–Q10 | Persistent separate Needs Attention queries for (a) overdue-unused packages and (b) actual post-review covered visits; new confirmation on each post-review visit; no reminder scheduler |
| Q11 | Browser-local autocomplete for post-review reasons; actual submitted reason stored with committed visit |
| Q12–Q14 | Closed package never reopens even if mistaken; unused entitlement forfeited; nonpayment alone not a closure trigger; payment after closure does not reactivate |
| Q15 | Closure reason is mandatory with suggestions + editable custom text; exact preset set not newly established here; 'Refunded' is not a preset |
| Q16 | Completed/Closed package visible in ordinary patient history with allowance, usage, closure reason and invoice link |
| Q17 | Reclassification/allowance-transfer question explicitly **dropped**, no A/B/C approved; no transfer of package allowances established |

---

## Original questionnaire Q18–Q40 — preserved questions and updated dispositions

**Reading rule for all entries:** Text labeled **Original scenario / A / B / C / Original recommendation / Original note** is preserved from the supplied reconstructed questionnaire. It is historical question text, not a selected outcome. Text labeled **Voice decision / reconciliation** below each question records the later owner response. All entries are retained to prevent misnumbering.
## Q18. What if a previously closed complaint is reopened?

**Original scenario.** A complaint was closed after its package reached Completed. Two months later, the same complaint is reopened clinically, assuming the future complaint module permits that.

- **A.** Reopening the complaint does not reactivate the old package. A new package must be purchased.
- **B.** If the old package was manually Closed with unused allowances, admin may decide whether to restore it.
- **C.** Do not support complaint reopening in the initial complaint-lifecycle MVP.

*Original caveat:* Conditional on complaint reopening becoming supported at all.



**Voice decision / reconciliation — DROPPED / COMPLAINT REOPENING NOT DECIDED.** The owner rejected the presupposition that clinical complaints can or should be reopened; when/why complaints close or reopen is outside the package question and not yet specified. Package lifecycle is **independent**: if a package is Closed or Completed it stays terminal under any later complaint action. Conversely an active package prevents closure of its complaint. Do not choose C (ban complaint reopening) without a separate complaint-module decision; do not choose B (restore allowances). Original A's *package nonreactivation* effect is already established, but its complaint-reopening premise is not an independently accepted feature.


## Q19. Must package purchases have their own invoices?

**Original scenario.** At the counter, a patient purchases a ₹4,500 package, receives a covered treatment, and owes a separate ₹350 examination fee.

- **A.** Issue a dedicated ₹4,500 package-purchase invoice and a separate treatment/examination invoice.
- **B.** Put the package purchase, treatment, and examination fee on one combined invoice.
- **C.** Allow both formats depending on checkout workflow.

*Original recommendation:* A. Isolates package-purchase corrections from legitimate examination and treatment charges; compatible with same counter interaction.



**Voice decision / reconciliation — OPTION B FOR SAME CHECKOUT.** The owner insisted: if a purchase, examination and other charges occur in **one checkout**, put them on **one invoice** with **separate invoice lines** for package, examination and each other item. Do not create an extra invoice simply to isolate package accounting. The owner did not say unrelated later checkouts must be combined. This choice directly contradicts the original recommendation A; the owner decision controls. ₹4,500 and ₹350 remain illustrative amounts.


## Q20. How should an entirely mistaken, unpaid package purchase be handled?

**Original scenario.** Admin registers a ₹4,500 package for the wrong patient. No treatment has been redeemed and no payment collected.

- **A.** Close the package with reason “Made by mistake”; admin separately waives unpaid purchase invoice with a reason.
- **B.** Close the package but retain collectible invoice balance until a future void mechanism exists.
- **C.** Require a genuine void/cancellation capability in MVP rather than using waiver.

*Original context:* A was the previously documented working direction, but not an explicitly accepted final procedure. It leaves an auditable waived invoice rather than erasing the sale.



**Voice decision / reconciliation — CUSTOM, SIMPLE CASE CLOSE + WAIVE.** When a package is registered by mistake, admin may Close it with a mandatory reason such as wrong registration; closure is irrevocable and separate from the invoice. If the resulting invoice contains only the invalid package charge and **no payment**, use the existing **whole-invoice terminal waiver** to settle its outstanding balance. There is no partial line waiver in MVP. **Mixed invoices** must follow Q21's later-approved sequence; earlier historical review-log language that there was only an unresolvable gap is superseded. Do not auto-waive every package closure or automatically delete a valid invoice.


## Q21. What if an erroneous package purchase shares an invoice with legitimate services?

**Original scenario.** A ₹4,500 package was added incorrectly to an invoice that also contains a valid ₹350 examination charge.

- **A.** Prevent this situation by requiring a separate package-purchase invoice.
- **B.** Introduce selective invoice-line correction or voiding.
- **C.** Accept current invoice-level waiver limitation and defer correction.

*Original recommendation:* A, consistent with Q19. Waiving entire ₹4,850 would wrongly forgive valid ₹350 charge.



**Voice decision / reconciliation — CUSTOM WORKAROUND; NO A/B/C SELECTED LITERALLY.** Same-checkout mixed invoice remains allowed, so original option A (prevent this situation by forcing a separate invoice) was **rejected by Q19**. No selective line void (option B) was approved; it was recorded for later owner review. The terminal waiver is **all currently outstanding balance**, not a chosen part. **Case 1: valid exam ₹350 already collected** on a ₹4,850 package-plus-exam invoice; Close erroneous package and waive the remaining ₹4,500. **Case 2: none collected**; Close erroneous package, **waive the whole ₹4,850 invoice**, then **create a clean ₹350 examination invoice**. This keeps the legitimate fee due on its new invoice. If additional valid charges existed, recreate those as appropriate; the owner did not request invoice-line edits. The owner at one point used 'void the whole thing' colloquially, but described **waive original + create new**, not approval for a real delete/void API. Historic review-log DR-01 on generic invoice/line voiding remains **deferred**.


## Q22. What if a mistaken package has already been partially paid?

**Original scenario.** A ₹4,500 package was purchased by mistake; clinic collected ₹1,500 and ₹3,000 remains outstanding. Current invoice model can waive ₹3,000 residual but cannot return ₹1,500.

- **A.** Close package, waive residual, and handle actual refund outside MVP, explicitly accepting software cannot record refund correctly.
- **B.** Include minimal refund-recording capability in MVP.
- **C.** Defer paid mistaken-purchase correction until future module; do not pretend waiver is refund.

*Original warning:* Manual cash return must not be reported as if no return occurred.



**Voice decision / reconciliation — REFUND MODULE + TERMINAL WAIVER + ORDINARY NEW PAYMENT, LOCKED.** The older original scenario assumed refunds cannot be recorded; owner clarified a **refund module is being built / in scope**. For partially paid erroneous package invoices, Close the wrong package, use the planned refund workflow for the money attributable to the mistake, record a **mandatory refund reason**, and terminally waive remaining erroneous receivable as needed. If the correct package is then purchased, issue a fresh normal invoice and enter a **new ordinary payment** against it (Cash or UPI, as applicable). **Worked mixed-charge example:** invoice ₹4,850 = ₹4,500 erroneous package + ₹350 valid exam; ₹1,500 was paid. Attribute ₹350 to the valid exam and **₹1,150** to the mistaken package; record a **₹1,150 refund** on the old invoice, settle the erroneous package balance via waiver, and register **₹1,150 as a new payment** against the invoice for the correct package, if that package is sold. The business result is old invoice settled with valid ₹350 retained, and the new invoice independently showing its charges and payment. The owner explicitly rejected patient-credit/wallet, inter-invoice transfer, fictional new payment *method*, extra cash-handover verification, or an additional audit subsystem. He regards ordinary Cash/UPI refund/payment entries with mandatory reasons as adequate. The assistant's cash-reporting objection was not owner-approved and must not recur as a scope mandate. **Implementation detail to verify, not a new choice:** refund and terminal-waiver ordering/accounting must yield correct settled balances even though the current waiver has no partial mode.


## Q23. What if the entire mistaken package was already paid?

**Original scenario.** Patient paid ₹4,500 before error discovered.

- **A.** Close package and handle reimbursement outside software, accepting reporting limitation.
- **B.** Require minimal auditable refund workflow before supporting correction.
- **C.** Handle only by exceptional administrative process outside MVP and explicitly mark it unsupported in software.

*Original note:* Waiver cannot substitute for refund on already-PAID invoice with no collectible balance.



**Voice decision / reconciliation — CONFIRMED FULL REFUND.** For a fully paid wrong package (e.g. ₹4,500), admin Close(s) the mistaken package and records **the full ₹4,500 refund in the planned refund module with a mandatory reason**. Nothing remains to waive when the original invoice contains only the fully paid package. If the clinic purchases/registers the correct package, that is a **new normal-price invoice** and the refunded amount can be entered as a **new ordinary payment** against it. The owner explicitly said Q23 follows the Q22 principle and confirmed the full ₹4,500 example. Do **not** say refund is 'outside software' or invent a credit-wallet feature.


## Q24. What if the mistaken package has already covered treatments?

**Original scenario.** A package was registered against wrong complaint and has already covered two recorded visits with issued ₹0 therapy lines. Mistake discovered afterward.

- **A.** Preserve committed records and close package; correction outside MVP.
- **B.** Narrowly scoped admin correction for package-to-visit attribution with safe financial consequences.
- **C.** Require broader visit/invoice correction capability before launch.

*Original distinction:* Different from an unused unpaid mistaken purchase.



**Voice decision / reconciliation — ORIGINAL SCENARIO REJECTED / DROPPED AS FRAMED; NO A/B/C SELECTED.** Owner's operational counterexample: a package registered for **shoulder Complaint B** cannot pay for treatment recorded for **knee Complaint A**. At the time of a knee visit the staff selects the knee complaint; since the knee complaint has no package, its visit is **normally chargeable**. A shoulder package remains unused; the user found the hypothetical of repeatedly covering knee care with a wrong shoulder package implausible under this selection mechanism. No cross-complaint package consumption or entitlement transfer was requested. In a later wrongly numbered question, owner also explicitly refused **retroactive conversion of an already billed knee visit into package coverage**. However, an **actually misrecorded committed visit**, if it ever occurs, is separately addressed by original Q40, still unanswered; don't silently answer Q40 here. The old assistant erroneously labeled mixed-invoice handling as Q24, but that valid answer belongs to Q21.


## Q25. Does waiving a valid package invoice affect its coverage?

**Original scenario.** Clinic deliberately gives a complimentary package by waiving its ₹4,500 purchase invoice. Package remains active.

- **A.** Coverage continues. Waiver changes receivable, not entitlement.
- **B.** Waiving package-purchase invoice must also close its package.
- **C.** Package-purchase invoice may only be waived after closing its package.

*Original recommendation:* A, consistent with payment-independent activation.



**Voice decision / reconciliation — A EXPLICITLY APPROVED.** A full waiver of a **legitimately created** package invoice changes only the financial receivable. The package remains active, subject to normal start date/daily cap/review rules, and its 10 or 26 covered visits are still usable. Do not conflate full invoice waiver with package closure. Crucial numbering correction: the assistant earlier called an unrelated used-visits hypothetical 'Q25' and dropped it; **this real Q25 was later asked correctly and answered A**.


## Q26. Should a closure reason labeled “Refunded” require proof of actual refund?

**Original scenario.** Admin closes a paid package and chooses “Refunded”, although money has not yet returned.

- **A.** Permit free-text closure reasons but make clear they are descriptive, not financial proof.
- **B.** Use neutral reasons such as “Refund requested”/“Refund-related closure”; reserve “Refund completed” for verified financial record.
- **C.** Exclude refund-related presets until actual refund module exists.

*Original recommendation:* B as long-term rule; A potentially for MVP if evidence unavailable. Closure label must never alter collections reporting.



**Voice decision / reconciliation — DROPPED / INAPPLICABLE PREMISE.** Owner stated the existing closure-reason presets do **not** contain 'Refunded' and refund is **not a package-closure reason**. No 'Refunded', 'Refund requested', or 'Refund completed' preset was approved. Package closure has its existing reason flow; a refund is a **separate transaction with its own mandatory reason**. The assistant's B recommendation was rejected as an unnecessary hypothetical. Do not manufacture a refund-proof requirement for package closure.


## Q27. What are the actual prices for all four offerings?

**Original catalog:**

| Category | 10-visit offering | 26-visit offering |
|---|---:|---:|
| Routine | ₹? | ₹? |
| Rehabilitation | ₹? | ₹? |

Provide current prices, or answer **“admin-configurable; actual figures to be entered during clinic setup”** if values not needed for specification. Earlier illustrative figures must not become clinic-approved prices.



**Voice decision / reconciliation — CLINIC-CONFIGURABLE, NOT FIXED.** Owner said prices are each clinic's configuration/setup decision, not something he is to supply as four fixed numbers. Leave actual four rupee prices **unspecified**, for the clinic to enter. Values like ₹4,500 in examples are not authoritative. Price-setting role is admin (Q28), while advanced offering/rule catalog management is future scope (Q31).


## Q28. Who may change package offering prices and settings?

**Original scenario.** Receptionist notices long Rehabilitation package price is incorrect.

- **A.** Only admin may edit package prices, availability, and commercial configuration.
- **B.** Receptionist may edit prices under restrictions.
- **C.** Only owner-level accounts may change package pricing.

*Original recommendation:* A, matching existing master-service pricing permissions.



**Voice decision / reconciliation — A, ADMIN ONLY.** Only admin may edit package prices and catalog-level commercial settings; a receptionist cannot. Owner explicitly corrected the question's third-role assumption: **there is no 'owner' role in this app**. The *only* roles are **admin and receptionist**; discard C as a product-role option. The assistant asked redundant 'locked?' confirmation and owner replied 'Obviously'. No new third role is to be built. This does not imply a general add/disable/custom-rule interface is in the MVP (Q31).


## Q29. Can staff override a package's catalog price during purchase?

**Original scenario.** A ₹4,500 package is being registered; clinic offers exceptional ₹500 discount.

- **A.** Sell at configured ₹4,500; admin waives ₹500 residual after ₹4,000 collected.
- **B.** Admin may set different purchase price before invoice issuance.
- **C.** No negotiated package prices/discounts in MVP.

*Original note:* A preserves invoice-line immutability while reporting ₹4,500 billed and ₹500 forgiven; B introduces overrides.



**Voice decision / reconciliation — A EXPLICITLY APPROVED.** On a ₹4,500 catalog-priced package, the **issued invoice remains ₹4,500**, even when clinic intends to collect ₹4,000. Collect ₹4,000 in the separate payments workflow; later **admin waives the full ₹500 remaining balance** (terminal invoice-level waiver). Do not modify the invoice line to ₹4,000, do not apply discount at sale, and do not confuse this with a partial waiver (₹500 happens to be **all** remaining). Owner stated: 'invoice will be the same; collection waiver happens later.' The assistant once incorrectly described Q29 as error tracking; that was an invented question and has no selected decision.


## Q30. Can the clinic issue a genuinely free package?

**Original scenario.** Owner gives staff or charity patient a 10-visit package at ₹0.

- **A.** Package can have zero purchase price when explicitly authorized.
- **B.** Always issue normal catalog price and use admin waiver.
- **C.** Free packages unsupported in MVP.

*Original distinction:* ₹0 purchase price is different from a covered ₹0 therapy-visit invoice line.



**Voice decision / reconciliation — B EXPLICITLY APPROVED.** Even when clinic intends to gift a 10-visit/26-visit package, register/package-price the sale at its **normal configured catalog price** and create an invoice for the **full original amount**. If the clinic wants to provide it free, admin **subsequently fully waives** the invoice; there is no true zero-price package sale in MVP. Owner emphasized ordinary price should 'show up on the invoice' and that free provision is a separate later waiver decision. Package stays valid/active despite financial waiver under Q25. The legacy scenario's word 'Owner' does not create an owner role. The assistant wrongly labeled a separate internal invoice-display question as Q30; the content is preserved as a cross-cutting view rule, not this answer.


## Q31. Can admin disable an offering for future purchases?

**Original scenario.** Clinic stops selling 10-visit Rehabilitation packages but existing patients retain valid purchased entitlements.

- **A.** Admin may disable new sales, existing entitlements unchanged.
- **B.** Offerings cannot be disabled in MVP; prices only modifiable.

*Original recommendation:* A. Disabling catalog offering should not revoke existing entitlements.



**Voice decision / reconciliation — B EXPLICITLY APPROVED, FUTURE-SCOPE EXPANSION NOTED.** There is **no MVP UI to disable an existing package offering**. The larger self-service configuration module—creating more offerings, disabling offerings, and configuring arbitrary new package rules—is **after MVP**. Price entry/modification authority (admin-only, Q27–Q28) remains separate from designing a general package rules builder. Owner directly said: 'not giving any UI to disable it yet' and 'future scope where they can disable or create new.' Assistant invented a Q31 about missing price; owner did accept **block sale if package price unconfigured**, recorded separately above, but it **does not replace actual Q31**.


## Q32. Are there already active packages outside this new system?

**Original scenario.** On deployment day, patients may have previously purchased packages recorded on paper, Excel, older software, or prototype database.

- **A.** Yes, real active patient packages must be carried forward.
- **B.** No, launching with no active historical packages to import.
- **C.** Only prototype/test data exists; may be discarded/rebuilt.

*Original note:* Determines real business-data migration versus technical schema migration.



**Voice decision / reconciliation — CUSTOM; NO HISTORICAL IMPORT IN CURRENT MODULE.** Real package commitments may exist on paper. The user explicitly wants a fresh software start with **no active-package migration or opening-balance import**. Preexisting real treatments continue to be honored and tracked on paper until completed. Future historical digitization is a **separately developed, separately charged retrospective entry module** covering past transactions, not an active entitlement onboarding requirement. Thus original options A/B/C do not precisely capture the accepted policy: **do not label it B (no historical active packages) or C (discardable tests)**. The earlier assistant's suggestion that this work was entirely outside scope was corrected: **deferred, not abandoned**.


## Q33. If historical packages exist, how should they be introduced?

**Original scenario.** Patient purchased package 20 days before launch, used 12 of 26, with 14 remaining.

- **A.** Import original purchase date, actual usage, remaining allowances, status, commercial terms, without fresh sale.
- **B.** Admin manually registers opening package balance, preserving original entitlement info.
- **C.** Start all imports as new purchases on launch date.

*Original recommendation:* A or B; C wrongly resets review clock.



**Voice decision / reconciliation — DEFERRED TO FUTURE SEPARATELY PAID HISTORICAL RECORD ENTRY.** The owner explicitly rejected designing active-package opening balances for the current module. Any future work is retrospective recording of events already completed, manually entered from paper/notebooks as needed, not fresh or still-running package registrations. No A/B/C selected for the future module. Option C must not be used to manufacture false new sales or reset historical dates. **Deferred**, rather than permanently out of scope.


## Q34. What if old package data is incomplete?

**Original scenario.** Paper register confirms purchase of 26-visit package but cannot reliably determine 13 versus 16 consumed treatments.

- **A.** Admin verifies and approves opening balance before migrating as usable.
- **B.** Best available estimate, explicitly marked estimated.
- **C.** Do not import unverifiable packages; handle manually outside new system.

*Original recommendation:* A. Never silently invent remaining allowances.



**Voice decision / reconciliation — DEFERRED.** The user confirmed incomplete paper history is not a question for the current Package Module, because historical record entry would be separately built and priced. No estimate/verification/rejection method (A/B/C) has been chosen for future transcription work; these are requirements to decide only if that paid project is commissioned. Do not assume missing data or silently invent used/remaining allowances.


## Q35. What happens to packages purchased under older commercial terms?

**Original scenario.** Patient bought older calendar-month package with terms different from new fixed 30-day/26-visit model.

- **A.** Honor original purchased terms; migration preserves legacy entitlement where applicable.
- **B.** Convert everyone to new policy after explicit clinic approval.
- **C.** Do not migrate incompatible packages; settle obligations outside app.

*Original recommendation:* A, consistent with immutable terms.



**Voice decision / reconciliation — DEFERRED.** Old offerings and their historical commercial terms are not imported into the live package engine at launch. The existing external paper commitments remain to be honored; the separate retrospective digitization module may later capture previous activities and terms. No A/B/C policy for conversion or importing legacy active entitlements was approved. This subject was explicitly placed in future separately charged scope, **not eliminated permanently**.


## Q36. Must historical unpaid package balances also be imported?

**Original scenario.** Preexisting patient has 10 treatments remaining and unpaid ₹2,000 package balance.

- **A.** Import active entitlement and independently collectible opening balance.
- **B.** Import only package coverage; old debt stays in previous accounts/register.
- **C.** No financial migration because no historical balances.

*Original note:* Financial migration scope must not control entitlement activation.



**Voice decision / reconciliation — DEFERRED; NO OPENING DEBT MIGRATION IN CURRENT MODULE.** Any preexisting unpaid balance stays in existing paper/accounting records. The clinic's new software starts without backfilling old balances; a future separately priced historical data-entry project might address prior financial history after its own scoping. **Do not adopt C's unsupported factual assertion that no old debts exist**, or A/B as an active import procedure. Money/entitlement remain conceptually separate. Owner corrected the earlier phrase "out of scope" to **"deferred"**.


## Q37. Can packages ever cover separately billed standalone services?

**Original scenario.** Patient with active knee complaint package receives packaged knee therapy plus separately purchased standalone traction service.

- **A.** Package covers eligible complaint-therapy visit only; standalone traction independently chargeable.
- **B.** Some standalone services included when configured in offering.
- **C.** Standalone services covered only by special admin authorization.

*Original recommendation:* A for MVP, following separation of complaint therapy and standalone purchases.



**Voice decision / reconciliation — A EXPLICITLY APPROVED, WITH OWNER'S PRECISION.** Packages pay the **flat standard charge for an eligible visit associated with that package's complaint**. Anything included as part of that ordinary visit is part of the covered visit. A separately billed standalone treatment or service is **not** covered by the package and remains independently chargeable. No B offering-level inclusion rule, no C special admin standalone coverage. Preserve the existing separation of eligible therapy-visit charge from examination, lapse and standalone service buckets; this answer does not silently waive other distinct charges.


## Q38. What warning should appear when a different complaint is selected?

**Original scenario.** Patient has active knee package but receptionist selects shoulder complaint for today's therapy.

- **A.** Nonblocking warning identifying other active package; continue ordinary shoulder billing.
- **B.** No warning; selected complaint independently determines coverage.
- **C.** Confirmation requiring acknowledgement that shoulder visit is not covered by knee package.

*Original recommendation:* C if not repetitively disruptive when treating multiple complaints routinely.



**Voice decision / reconciliation — A EXPLICITLY APPROVED (AFTER GITHUB CHECK).** Patient has knee package but shoulder complaint is selected: show a **nonblocking informational warning that identifies the active package on the other complaint**, then allow ordinary shoulder billing to proceed. There is **no mandatory acknowledgement** and no cross-complaint coverage. The user explicitly selected A after the assistant had temporarily left UI warning undecided. The GitHub review supported the intended complaint-specific matching design but was **not a complete implemented-feature audit**; do not claim the warning already ships.


## Q39. Should unpaid packages appear in admin Needs Attention separately from overdue review events?

**Original scenario.** Patient has active package with ₹4,500 outstanding but legitimately consumed five covered visits.

- **A.** Separate unpaid-package category in Needs Attention.
- **B.** Debt only in ordinary invoice/payment reports.
- **C.** Balance indicator in package view but no Needs Attention item.

*Original note:* Informational only, never automatic coverage stop.



**Voice decision / reconciliation — FINAL CUSTOM RULE; EARLIER PROVISIONAL A SUPERSEDED.** In an earlier voice exchange the user appeared to accept a separate unpaid-package Needs Attention section. Subsequently, when asked about mixed invoices, the user explicitly corrected this: **packages do not carry Paid/Unpaid status; invoices do**. Therefore there is **no package-specific unpaid category**, balance allocation, or package special ranking in Needs Attention. General due-invoice attention sorts invoices by **current unpaid amount descending**, regardless of their line items. **Any positive balance, including ₹100, remains visible until fully paid or waived**. Package invoice amounts often rank high because package sales are large, but this is contingent on actual outstanding amount, not guaranteed. Besides money dues, Needs Attention can contain **clinic-defined patient warning notes** (e.g. patient behavior or any other clinic-selected concern), and attention information can appear in **multiple relevant application contexts**, not just one dashboard. UI/report design is future work. Existing post-review and overdue-unused-package attention rules from Q08–Q10 are distinct; this latest answer only rejects a dedicated *unpaid-package* category. The original A/B/C options do not precisely describe this final general-invoice policy, so do not force them into a selected letter. Debts do not block package coverage.


## Q40. What is the minimum safe response to an incorrect committed visit?

**Original scenario.** Receptionist records treatment against wrong complaint; invoice and package redemption are already committed. Current MVP lacks issued-invoice editing and general retroactive clinical correction.

- **A.** Accept MVP limitation: flag for manual investigation; committed record not changed.
- **B.** Narrow admin-only correction for visit/package attribution, with invoice integrity checks.
- **C.** Require full future correction/void framework before real production use.

*Original recommendation:* B for live clinic use if carefully designed; A risks unfixable integrity problem; C expands project scope.



**Voice decision / reconciliation — A EXPLICITLY APPROVED, WITH NO NEW VOID TOOL.** If staff record a treatment against the wrong complaint and commit an invoice/package redemption, the committed record **remains unchanged** in MVP. The user prefers substantial verification during initial record creation to minimize entry errors and does not authorize visit editing, package-visit reassignment, retroactive coverage or invoice/visit voiding. Manual investigation may occur outside a new correction mechanism. Although original A uses "flag for manual investigation," this is **not authorization for a new technical flags/audit workflow**. The exact precommit verification UI was not fully specified here. A generic future invoice/line void idea DR-01 remains deferred. The user expressly accepts the resulting MVP limitation.


---

## Final integrity checklist for implementation, handoff and future discussions

- **Numbering:** Q32 was a historical-import decision, not any fabricated visit-notification question. Q33–Q36 are **deferred to separate historical-entry development**, not somehow already decided as an import algorithm. Q37 = A, Q38 = A, Q40 = A. Q39 = **custom general invoice/attention model**, superseding the intermediate apparent A.
- **No repeated questions:** All Q01–Q40 package questions were addressed by approved answer, rejection of an invalid premise, or explicit deferral. Do **not** reopen the Q38 warning decision or demand invoice-line payment allocation to support a nonexistent package unpaid category.
- **Do not turn cross-module issues into package requirements:** The reopening of a closed complaint belongs to the Complaint Module and remains undecided there. It does not reactivate a terminal package. Clinic-defined warning notes relate to general Needs Attention behavior; the user gave examples, not a full new UI/database architecture for those notes.
- **Keep the business objects distinct:** package = complaint-specific treatment entitlement; visit = the event potentially using that allowance; invoice = billed amount, payment status, due balance, waiver and payment/refund linkage; warning note = a separately relevant clinic concern. A ₹100 positive invoice residual can merit attention even if a package was involved, without needing an `unpaid_package` status.
- **Previously resolved accounting remains unchanged:** Full current balance waiver only; separate reasoned Cash/UPI refunds; corrected new invoice/payment when required; package closure not equivalent to waived invoice; correct package entitlement independent of invoice payment. No generic editing, void, wallet or transfer system was approved.
- **Exactly two relevant GitHub repositories (owner confirmed):** `omsrivastava512/clinic-demo` and `omsrivastava512/clinic-demo-be`. Partial document/code inspection in this conversation is **not** proof of live deployment. Any later code audit should be requested as such and compare the implementation against these final decisions.
- **Engineering validation, not new owner questions:** When implementing, verify admin price configuration for the four offerings, block purchase when price unconfigured, accurate invoice/refund/waiver arithmetic, truthful outstanding-invoice attention for any positive balance, precommit verification flow, nonblocking Q38 notice, and no accidental entitlement dependence on payment. These are examples of developer acceptance checks, **not** additional package policy questions or a claim that tests were run.
- **Source transparency:** The exact reconstructed historical Q18–Q40 question and option wording is retained above; subsequent voice decisions supersede old recommendation prose and old unanswereds. If later authors add decisions, preserve version precedence and document their source/date. No changes to GitHub, database or deployment were made as part of this handoff.

**Final checkpoint: This Package Module questionnaire is closed for present business decisions.** Historical record transcription is a separately chargeable deferred project; complaint reopening is a separate module; any unfinished implementation work should be tracked as engineering tasks, not represented as unanswered owner choices.
