# Catalog identity and naming — discussion log

**Date:** 4 November 2026 (Asia/Kolkata)  
**Status:** Active discussion, staged for later incorporation into the Module 2 decision records.  
**Scope:** Canonical service names and identity. This log does not change the application, schema, or existing decision records.

## D01 — Canonical service names after launch

**Status: USER-SETTLED.** Once the clinic begins registering visits, block renaming canonical service names through the application. A service ID remains associated with the same procedure: for example, an ultrasound identity must not be repurposed as cupping. The user prefers any necessary early setup or migration corrections to be handled manually in the database before visits begin.

Alternate wording can be handled through aliases. When the underlying procedure is different, create a distinct catalog identity rather than repurposing the old one. This decision concerns canonical service names and identity; it does not revoke the existing admin authority over service prices and availability.

**User rationale:** The user sees no operational reason to change ultrasound into something else after records exist, and prefers to prevent accidental or malicious changes. This builds on the settled standalone clinical-record choice: store `service_id` and display its catalog name through a join; issued invoice descriptions and amounts remain preserved separately.

## Implementation considerations — not separately settled

- Enforce the naming restriction beyond the UI, so direct requests cannot bypass it. Exact permissions and database enforcement remain technical design work.
- The exact aliases representation and management rules remain open. Do not infer that every catalog needs its own aliases table.
- The mechanism for recognizing the transition from setup to recording visits is not yet specified. Post-launch correction of a canonical name is not approved by this decision; do not invent an exception.
- This discussion used `services` as its concrete example. Whether the same rule applies to every other catalog remains to be decided.

## Provenance and later incorporation

Source: user's 4 November 2026 messages in this conversation expressing the no-rename rule and explicitly asking to mark it settled and maintain an `upcoming` log. The earlier Q06 decision is recorded in `../clinical-records.md` and `../history/clinic_module2_seven_review_answers_2026-10-27.md`. The current catalog authority is in `../service-catalog-and-branch-pricing.md`.

This is a staged session log, not yet reconciled into topic pages, the decision register, or the tracker. Append later discussion here; incorporate into those records only when requested. No implementation or database enforcement has been verified.

---

# Package, billing and architecture discussion — 4 November 2026

**Status:** Active discussion; staged for later reconciliation. Source is the user's outline and subsequent annotations and clarifications in this chat, together with the assistant's clearly identified recommendations. This is a faithful discussion summary, not a verbatim transcript of the entire exchange.

**Scope and authority:** The user requested that the new file in `upcoming/` be updated as a persistent log while the discussion continues. Retain the original catalog-identity entry above. Do not infer approval of technical recommendations or implementation. This staged checkpoint has not been incorporated into the topic pages, reconciliation register or tracker. Those records document earlier decisions; the user's subsequent clarifications below must be considered when later reconciling the relevant subjects. No application code, schema, deployed database, or Git history was changed by this checkpoint.

## 1. Context: comparison with the October 22 architecture review

The user requested a comparison of the October 22 architecture resolution with the Module 2 decisions, separating compatible, changed, and still-open recommendations. The assistant identified continued support for complaint-specific visits, flat-therapy invoice lines, clinical/financial separation, atomic invoice issuance, historical invoice snapshots, and occurrence-level duplicate protection. It also identified changed scope: flat-only ordinary therapy; standalone services and packages within MVP; current single-clinic catalog pricing; and independent first/return fee events with settled calendar bands.

The assistant then incorrectly narrowed "what needs my attention" to four package business-policy questions, omitting open architectural choices. The user challenged this. The assistant acknowledged that technical work can be investigated and prepared independently, but consequential architecture choices still need to be presented for the user's review under the user's coding instructions. The user requested direct investigation rather than a promise to investigate later.

The assistant also acknowledged a cutoff error: the current package records had since incorporated S28–S33 through November 3, resolving the original payment-independent coverage, fixed 30-day software review, and adoption of post-review notes. A later conversation outline proposed a queue and a debt block; these were subsequently refined by the user's annotations below. The outline is retained as submitted for provenance, not as a single approved policy.

## 2. User's original outline — preserved as submitted

```vbnet
package_management:

  business_rules:
    payment:
      - Patients can use a package before paying its full amount.
      - An invoice is created when the package is purchased.
      - Unpaid amounts remain due on the invoice.
      - Outstanding payment must not block package usage.

    advance_purchase:
      - Patients can purchase a replacement package before the current one ends.
      - Advance purchase is allowed only if the previous package invoice is settled.
      - Settled means either fully paid or explicitly waived.
      - An unpaid, unsettled previous package invoice blocks purchasing
        another package.

    activation:
      - An advance-purchased package remains queued.
      - It becomes active after the previous package reaches a terminal state.
      - Activation is intended to happen the day after the previous
        package's last covered visit.
      - The package start date remains NULL while queued.
      - The start date is registered upon activation.

    duration:
      - The long package review period is exactly 30 days.

    review_notes:
      - Continuing treatment after the review date requires a written reason.
      - Reception must provide the reason.
      - Admin may optionally provide the reason.

  package_states:
    - queued
    - active
    - closed
    - completed

  state_transitions:
    queued:
      - Waiting for the previous package to reach a terminal state.
      - Start date remains NULL.

    active:
      - Package is available for use.
      - Start date is registered.

    completed:
      - All package visits have been consumed.

    closed:
      - Package is stopped before all visits are consumed.
      - Remaining unused visits are forfeited.
      - Closure is a clinical/package-management decision.
      - Closure is NOT a financial or payment status.

  package_assignment:
    - A package is registered against a complaint, not merely a patient.
    - Package usage must therefore be associated with the relevant
      complaint visit.
    - Multiple complaints belonging to the same patient must remain
      distinguishable.

  package_usage:
    - One package credit is consumed per completed complaint visit.
    - An incomplete visit must not consume a credit.
    - Remaining allowance is calculated rather than maintained
      as a manually updated counter.

  database_design:
    proposed_tables:
      - packages
      - patient_packages

    proposed_relationships:
      - Patient purchases are represented in patient_packages.
      - Package assignment is complaint-specific.
      - Package usage may be represented directly on visits
        through a nullable foreign key to the relevant package.

    redemption_table:
      - A separate redemption table was considered.
      - Recommendation: avoid a separate table for now.
      - Store package usage directly against visits.
      - Reconsider a separate table if multiple packages can fund
        one visit, partial credit deductions are needed, or
        detailed usage/reversal auditing becomes necessary.

    allowance_calculation:
      - Total package visits minus completed complaint visits
        associated with that package.
      - Credit consumption must be transactionally protected
        against simultaneous requests.

  discussion_still_open:

    visit_completion:
      - Visits do not currently have lifecycle states.
      - We agreed that only completed visits consume package credits.
      - Manual completion was discussed as a possibility.
      - We have NOT finalized how completion is represented
        or recorded without introducing confusing visit states.

    terminology:
      - We identified possible confusion between completion and closure.
      - Closure is independent of payment.
      - The precise UI terminology and actions still need definition.

    activation_timing:
      - We discussed activation after the previous package reaches
        a terminal state.
      - We also agreed that the next package starts the day after
        the previous package's last covered visit.
      - The exact handling of early closure needs to be specified
        so these two rules cannot conflict.

    database:
      - The proposed visit-to-package foreign key is not yet
        a finalized schema decision.
      - Invoice relationships and transaction handling still
        need architectural design.
```

## 3. User's subsequent clarifications — current discussion

### Package purchase, queue and start date

- **[USER CLARIFICATION]** Queueing and buying a replacement while an existing same-complaint package remains active are deferred: potentially post-MVP or, only if time permits, toward the end of MVP. The user explicitly said not to design queued activation now. A replacement or an upgrade can be bought only after the existing same-complaint package is **Closed or Completed**. Reaching the review date alone does not terminate an active package.
- **[USER CLARIFICATION; ALREADY RECORDED IN S30 Q01–Q02]** For an ordinary first purchase, start today. For a later purchase for the same complaint, check whether **any package covered a visit for that complaint today** in the clinic's local calendar: yes → start and review clock begin tomorrow; no → start and review clock begin today. A different complaint does not trigger this rule. The already established daily cap and other existing policies do not need to be repeated in a focused outline.
- **Examples:** A package wrongly registered and closed without any covered visit today → a replacement can start today. A package covered a visit today and was then closed or completed → a replacement starts tomorrow. A patient absent for five days returns and the earlier package is closed without a covered visit today → the replacement starts today. The prior review date or a visit several days ago does not itself determine the new start.
- **[FUTURE DISCUSSION]** For a future queue, the user considered activation on the day after the previous package terminates, whether by completion or closure. How queued items can be cancelled, how absence and delayed early closure affect activation/review, and what happens when there was no last covered visit remain future design questions. Do not conflate this future anchor with the decided current no-queue rule. A proposed queued state or NULL start date is not approved for current MVP merely because it appears in the original outline.

### Package debt and future reception policy

- **[USER EXPLORATION, THEN DEFERRED]** The user considered blocking later package purchases until the prior package invoice was Paid or Waived, to discourage accumulated debt. The user then proposed allowing admin to proceed despite outstanding debt while blocking future reception staff, and suggested deferring that role-specific mechanism because the current operating phase is admin-only. Preserve this evolution; do not record a blanket debt block or a reception implementation as settled MVP policy.
- **[CURRENT RECORDED BASELINE]** Package usage is independent of whether the purchase invoice is paid. Current topic records allow admin renewal after termination with an unpaid-balance warning. The package's lifecycle state does not equal invoice settlement. A future rule could follow the purchase line to the invoice and check financial state rather than adding a package paid/unpaid flag; exact relationship and permission enforcement remain open.
- **[ASSISTANT ANALYSIS / FUTURE DESIGN]** A package may share an invoice with an exam or other charge. A whole-invoice debt block can therefore block over debt not individually allocated to the package. Checking only the immediately preceding purchase may miss older debts if admin has allowed exceptions. Decide scope and mixed-invoice treatment when the future policy is revisited; do not silently add allocation rules or turn a warning into an MVP hard block.

### Duration, review notes and closure

- **[RECORDED BASELINE]** Long-package review uses 30 consecutive calendar dates beginning with the package start date, including that start day; day 30 is a soft review, not automatic loss of remaining visits.
- The outline says continuing after review "requires a written reason," while also saying reception must supply one and admin may optionally do so. The recorded S30 Q06–Q07 rule resolves the role distinction: a reason is required **per post-review covered visit by future reception**, optional for admin, and submitted reasons are immutable. The first blanket sentence should not be interpreted as making admin's reason mandatory.
- **[RECORDED BASELINE]** Completed means the full allowance was consumed; Closed means deliberately terminated before exhaustion, with remaining visits forfeited. Closing the package does not pay, waive, cancel, or refund its invoice. Distinguish package closure from financial settlement. The outline's general wording does not authorize new UI terminology or a financial correction workflow.
- **[USER QUESTION / ASSISTANT RECOMMENDATION]** The user questioned why "Duplicate Package Purchased" is a closure preset if a second active package for the same complaint is blocked. The assistant recommended considering its removal, since "Wrong Package Registered" can cover a mistaken registration; this is not a user decision to change the preset. Duplicate-submission safeguards still need to be effective.

## 4. Visit completion and package consumption

The user said the assistant's earlier question about visit completion was unclear. The recorded workflow states visits are entered after treatment finishes and are immutable once committed. **[ASSISTANT RECOMMENDATION, NOT APPROVED SCHEMA]** Treat a successfully committed complaint visit as the record of completed care, without adding a manual "complete visit" button or extra lifecycle states solely for package usage. A covered committed visit consumes one credit; an unfinished form or rolled-back transaction consumes none. Payment is separate from clinical completion.

Derive remaining allowance from the purchased allowance snapshot minus committed visits associated with that purchased package. A visit's association to a purchase remains a proposed design, not a finalized FK. Never treat a ₹0 invoice line alone as proof of a legitimate redemption. Clinical visits and invoice lines remain separate facts; covered-only zero invoices follow the existing validated-coverage and no-payment-row rule.

## 5. Catalog and purchased-package naming — still proposed

The user asked whether `patient_packages`, `complaint_packages`, or another name is most appropriate, and suggested using `package_catalog` for definitions.

**[ASSISTANT RECOMMENDATION]** Use `package_catalog` for offering definitions, retain the existing schema document's `packages` name for individual purchased entitlements assigned to complaints, and consider a nullable `visits.package_id` to identify the purchase covering a visit. This fits the existing `complaint_catalog` naming convention and avoids renaming an existing purchase concept. `patient_packages` and `complaint_packages` remain alternatives; exact names, fields, schema changes, migration and relationships are not approved by this discussion.

A separate redemption table was discussed; the assistant recommended a direct visit-to-purchase link as a candidate for the present one-package/one-credit-per-visit model. Revisit a dedicated redemption model if future approved requirements need multiple funders per visit, partial credits or reversal/audit behavior. This is architectural advice, not proof of implementation.

## 6. Read-only integrity investigation and proposed safeguards

The user explicitly requested that the assistant check complaint ownership, the last-credit race, daily cap, and purchase/visit invoice relationships. The assistant inspected the repository migration document and current decision records. In the inspected migration text, `packages` is a purchased patient-package record with separate attendance/day-log values; `visits` has no package FK. The visit/patient-clinic access check does not by itself establish that the complaint belongs to the same patient and clinic. Historical handover proposals are not current schema evidence. No live database, deployed backend or actual production data was inspected, and no concurrency or integration test was executed.

**[ASSISTANT PROPOSAL / OPEN ARCHITECTURE]** Subject to user approval and a fresh schema audit:

1. Enforce that the purchased package, its complaint, the visit and patient/clinic context agree; a bare package ID alone does not guarantee that match. Deny unauthorized direct writes.
2. Enforce at most one active purchased package per complaint with a database uniqueness backstop and an atomic purchase flow. Review how a package purchased today with tomorrow's `start_date` is represented without enabling overlapping coverage or adding the deferred queue.
3. At redemption, coordinate on a common complaint/package lock, check active state, start date, local date, allowance and applicable daily limit, then create the clinical visit and validated covered invoice line atomically. Purchase, closure, redemption and relevant cap changes must use a consistent concurrency protocol. If one credit remains, concurrent distinct submissions cannot both consume it.
4. Calculate allowance from committed linked visits rather than a manually updated counter. A server-derived, snapshotted total allowance protects existing purchases when a catalog offering changes.
5. Link a package purchase line to the specific purchased entitlement and a covered therapy line to the specific visit. Ensure exactly one valid source and correct invoice membership, including combined purchase/fee invoices; other fees remain independent. Prevent the same actual source being billed twice, without banning repeated genuine visits.
6. Use a stable submission identity for checkout retries so a lost response does not create another visit or purchase. Exact key/hash storage, lock ordering, authorization, and transaction boundary still require design and testing. When a second *distinct* visit loses the last-credit race, do not silently charge it; return the result for staff review.

The proposed controls are not a claim that a direct FK alone enforces every cross-row invariant. Database constraints and controlled transactional writes serve different roles; exact implementation and concurrency acceptance checks are open. No SQL or application changes have been made.

## 7. Upgrade versus replacement — user's next decision remains open

**[USER CLARIFICATION]** Both replacement and upgrade purchase require the existing package to be Closed or Completed; no active-package upgrade or queue is approved by this clarification.

**[EARLIER S33 PROPOSAL, NOT APPROVED]** A same-category 10-to-26 upgrade could charge the difference and grant a total of 26 visits including the original 10, preserving the original review anchor through a linked extension. A normal fresh 26-visit package would instead provide 26 new visits at its configured price and a new start date. S33 also leaves price changes, elapsed time, exact implementation and inclusion in MVP unresolved.

**[OPEN QUESTION ASKED; NOT ANSWERED IN THIS CHAT]** Once a 10-visit package is complete, does "upgrade" mean 16 additional visits for the price difference with the original anchor, or a fresh 26-visit replacement at the normal price and new anchor? If early closure is involved, the relationship between forfeited unused credits and any later extension also requires explicit discussion. Do not silently carry one choice into schema or policy.

## 8. Continuation and verification boundary

Continue appending future clarifications to this staged file when the user asks. Reconcile topic pages, register and tracker only on a later requested incorporation, following `../AGENTS.md`. Preserve the distinctions among user clarifications, existing recorded policy, assistant recommendation, future proposals and unresolved questions. In particular, the original outline is historical input; Section 3 records its later queue and debt refinements.

The live backend default `main` was checked read-only during the architecture discussion: local HEAD and live main were both `9670880cdbd7be86fa20dba16130f6abad986532` (0 ahead / 0 behind), with existing local changes present. Relevant package work was found in decision records; no backend pull requests were returned. This is a point-in-time repository check, not a live database or implementation verification. No Git mutation was performed.

---

# Continuation — 5 November 2026

**Source:** The user's next message in this chat, including explicit scope, future reception debt policy, ordinary visit entry, the duplicate-package preset, naming, guardrail acceptance, and a request to inspect the staged upgrade design. The user reaffirmed that current and future scope must be documented persistently. This is a staged continuation; no topic-page, tracker, code or schema incorporation is implied.

## 9. Admin-only MVP and future role scope

**[USER-SETTLED / REAFFIRMED]** MVP has **admin only**. There is no receptionist application role in the current MVP. References to reception permissions, restrictions and role-specific enforcement are future scope and must be documented as such. Admin-only MVP does not mean unauthenticated or unrestricted access: existing patient/complaint ownership and financial/clinical integrity requirements still apply.

The user requested explicit preservation of future plans rather than omitting them because reception is absent today. Queueing remains deferred, not silently included as part of upgrade implementation. Its timing, cancellation and delayed-closure questions remain recorded in Section 3.

## 10. Future receptionist package-purchase debt restriction

**[USER-SETTLED FUTURE POLICY]** When a receptionist role is eventually introduced, reception cannot sell an additional package if the patient has **any one or more package-linked purchase invoices that remain unpaid/unsettled**, including older invoices even when admin subsequently allowed more package purchases. Do not check only the immediately preceding package or restrict the debt search to the currently selected complaint. The user's wording scopes this to the patient's package-linked invoices.

- Admin may permit subsequent package purchases despite outstanding balances. This debt discretion does not bypass the separate prohibition on buying another package while the same complaint's existing package is Active.
- Invoice settlement means PAID or WAIVED under the recorded finance model. Package Closed/Completed is independent of payment; packages do not gain a duplicate paid/unpaid flag.
- **Current delivery:** retain admin-only debt visibility/discretion; reception-specific purchase blocking is deferred with the reception role. This refines Section 3's earlier exploration into an explicit future policy, not a current admin hard block.
- **Future integration consequence / implementation open:** the user's restriction is expressed at invoice level. A combined package/service invoice therefore remains a relevant unsettled invoice even if staff informally considers its package portion paid; the model has no settled per-line receipt allocation. Make this consequence visible when designing the future rule rather than inventing allocation rules or automatically reopening the policy. Check each distinct relevant invoice once, rather than multiplying debt through package/line joins. Exact query, permission checks, concurrency protocol and later refund/correction interactions are technical design work.

**User rationale:** Packages commit the clinic to future covered treatments; additional package credit therefore deserves a different restriction from recording care that has already occurred.

## 11. Ordinary completed visits despite existing debt

**[USER DIRECTION]** Allow ordinary non-package complaint-visits to be logged even if the patient has many unpaid earlier visit invoices. In the future, receptionist entry should remain permitted in that situation as well. The user considered a debt threshold but expressed that it would be a bad idea; **no debt/count threshold or new hard block is approved**.

**User rationale:** A visit records treatment that actually happened. The software should not refuse the clinical entry because earlier treatment has not been paid. Actual performed care still follows the established atomic visit/fee/invoice issuance and separate settlement workflow; logging it is not a waiver of either old or new debt. This does not authorize fabricating visits, deleting records or making package coverage transferable across complaints.

**[PARKED BY USER]** How to manage or resolve hundreds of old unpaid invoices is a separate discussion the user will return to. No bulk waiver, cross-invoice payment allocation, automatic collection, new threshold, or debt-management module was selected.

## 12. Duplicate-package closure preset

**[USER DIRECTION]** "Duplicate Package Purchased" has no ordinary role in the current no-overlap/no-queue workflow; reserve reconsideration for future scope. The user proposed retaining it as commented-out code rather than an active reason.

**[ASSISTANT RECOMMENDATION; IMPLEMENTATION NOT SELECTED]** Preserve the deferred preset in the discussion/future-scope record and omit it from the active MVP reason set when that implementation is undertaken, rather than adding commented-out executable code now. Existing purchase uniqueness and retry safeguards should prevent an exact duplicate; "Wrong Package Registered" covers a mistaken complaint/patient/offering choice. Queueing alone would permit intentional sequential purchases, not make them erroneous duplicates; an actual accidental duplicate would need a defined meaning before the preset is restored.

No code or existing operative preset list was changed in this staged update. The earlier topic page's preset list still needs reconciliation on a later requested incorporation.

## 13. Naming and integrity guardrails accepted

**[USER-ACCEPTED NAMING DIRECTION]** The user accepts `package_catalog` for offering definitions and `packages` for purchased complaint-specific entitlements, as recommended in Section 5. A visit-to-purchased-package link remains the recommended attribution direction. This acceptance supersedes the "still proposed" status of the naming choice in Section 5 for this conversation; it does not apply a migration or decide every field and FK.

**[USER-ACCEPTED GUARDRAIL DIRECTION]** The user agrees with the safeguards presented in Section 6: matching complaint/patient/clinic ownership, one active purchase per complaint, derived allowance from real committed linked visits, concurrency-safe consumption and daily limits, correct purchase/covered-visit invoice provenance, and retry duplicate protection. Exact constraints, locks, function/API shapes, migration and tests remain design/implementation work. Acceptance of these goals does not certify that the inspected schema already enforces them.

## 14. Newly supplied staged upgrade log — inspected in full

**Reference:** [Package Upgrade — Discussion, Decision & Database Design Log](package_upgrade_design_discussion_log_2026-11-04_0220_IST.md), recorded 4 November 2026 at 02:20 IST. The user describes it as still being discussed. The assistant read the complete file on 5 November. The file is preserved unchanged and is a staged source, not automatically incorporated current policy or implemented schema.

### What the uploaded log answers

- It explicitly describes a **same-complaint, same-category 10-to-26 upgrade**, contributing **16 additional visits**, rather than a new independent 26-visit purchase. It is initiated through a parent 10-visit package and does not create a fifth freely purchasable catalog offering.
- It preserves the original 10-visit package's coverage-start anchor for the long package's Day-30 review milestone, rather than starting another 30-day review period on extension purchase. Review remains soft.
- It records the intended latest purchase opportunity as the **30th inclusive calendar date from the original coverage start**; late purchase does not restart review.
- It records a preference for a linked extension row contributing 16 visits, keeping original terminal history intact. Field names, physical relationships and a separate offering FK were still proposals, not approved DDL.
- It records user confirmation of flooring a negative price difference at **zero**, rather than a refund or negative invoice line. Exact integration of a zero-charge upgrade with the previously narrower zero-total invoice exception is still open.

**Correction to Section 7:** The assistant's prior question contrasting 16 additional visits with a new independent 26 visits is answered by this source's explicit feature description. Do not ask the user to repeat that distinction. Specific pricing, closure and implementation questions below remain open.

### Precedence against this chat's later clarifications

- The upgrade log describes purchase before the first 10 visits are exhausted as confirmed intent, and a Queued child as its proposed implementation. The user's later clarification in this chat requires waiting for the existing package to be Closed or Completed and defers queueing. **Use the later terminal-only/no-queue restriction for the current discussion; early purchase is not current MVP approval.**
- The upgrade log's admin-and-reception initiation wording is subject to the user's reaffirmed **admin-only MVP**. Reception authorization is future scope, including the package-invoice debt restriction in Section 10.
- The upgrade log's old assistant preference to omit an offering FK does not override the user's accepted `package_catalog` naming/definition direction. Exact catalog-to-purchase representation still needs design.
- A fresh replacement's T/T+1 coverage start and an upgrade's inherited review anchor are distinct concepts. Keeping the parent review anchor does not authorize coverage before the extension is purchased or duplicate same-day coverage. Do not apply the replacement's new review cycle automatically to the extension.

### Upgrade-specific unresolved work to preserve

1. **Pricing basis:** the upload says the user initially spoke of subtracting money "paid," whereas the assistant proposed `max(0, current long-offering price - original short-package invoiced charge)`. The billed-versus-collected basis still lacks explicit sign-off in that source. The assistant recommends the invoiced-charge basis so unpaid original debt is not charged twice. Existing payments and debt remain on the original invoice; waiver is not a transfer.
2. **Early Closed parent:** the current conversation permits considering a purchase only after Closed/Completed, but does not specify the extension's allowance when the parent was deliberately closed before using all 10. Forfeiture and combined entitlement must be reconciled explicitly. Do not infer that every Closed parent must grant the same remaining credits as a naturally exhausted parent.
3. **Gap visits:** the upload flags that already committed ordinary-price visits between original exhaustion and later extension must not silently become retroactively covered. Existing no-retroactive-correction decisions remain the baseline; inheriting an earlier review anchor is not retroactive redemption permission.
4. **Zero-charge upgrade:** the user-confirmed zero floor conflicts with treating every zero-total invoice as exclusively package-covered treatment. A narrow upgrade-sale exception and settlement behavior need design and approval if upgrades are included; do not create fictitious payment rows or automatically convert zero into a waiver.
5. **Delivery scope and design:** upload supports inclusion only if bounded in complexity. Final MVP inclusion, linked-extension DDL, uniqueness of an upgrade per parent, exact price snapshots, review/activation dates, transaction boundaries and tests remain unresolved. No active/queued extension is silently added under the guardrail approval.

## 15. Assistant assessment and next discussion boundary

Today's user clarifications settle the current role scope, future reception search across the patient's package-linked invoices, allowance of real ordinary visits despite old debt, the naming direction and acceptance of integrity goals. They do not require reopening first-purchase dates, the decided T/T+1 replacement cases, or settled policies merely because those were omitted from a focused outline.

**[ASSISTANT ASSESSMENT]** The main present upgrade choice still requiring business input is pricing basis; closed-parent forfeiture also needs a concrete scenario before extension eligibility is implemented. Reception mixed-invoice debt handling and queue cancellation/activation belong to the deferred feature designs. The large-old-debt workflow remains parked at the user's request. No new business block, receptionist role, code comment, preset implementation or database mutation was introduced by this log update.

---

## 16. Upgrade pricing basis settled; negative difference and refunds clarified

**Source:** The user's subsequent 5 November reply in this chat. This continuation supersedes the open pricing-basis item in Section 14; historical wording above is retained for chronology.

- **[USER-SETTLED]** Subtract the original short-package **immutable invoiced charge**, not the actual money collected, when calculating the upgrade price. The user explicitly accepted the assistant's recommendation and clarified that "price" had meant invoice price. Original unpaid receivables remain on their original invoice.
- **[USER-CLARIFIED]** The previously discussed zero floor prevents a negative upgrade invoice or a negative new amount owed if the current long-offering price becomes lower than the original short-package charge. The user considers that unlikely in ordinary business, not a reason to allow a negative sale.
- **Example:** Original short charge ₹1,800; hypothetical current long price ₹1,500; nominal difference is -₹300. The user says any ₹300 returned should go through the refund mechanism, not a negative upgrade charge. This distinguishes the upgrade price calculation from a separate refund transaction.
- **[ASSISTANT FINANCIAL-INTEGRATION CLARIFICATION]** A negative price comparison alone does not prove ₹300 was actually received or automatically authorize paying it out. Refunds must relate to actual collected money and follow the existing authorized refund workflow. An original partly unpaid charge, prior refunds, and other relevant finance facts still matter. Do not automatically rewrite the original invoice, erase debt, or generate a refund just because a catalog price changed.
- **[IMPLEMENTATION STILL OPEN]** A zero upgrade charge, its invoice/settlement representation, and any separately authorized refund require coherent implementation. The clarification does not itself approve a generic zero-sale exception, automatic refund or negative invoice line.

## 17. Package termination meanings — three approaches raised by user

**[USER BRAINSTORM; NO LIFECYCLE CHANGE CHOSEN]** The user is reconsidering the meanings of Active, Closed and Completed to support sensible closure reasons and upgrade rules. Queued remains future scope. The user explicitly needs more time to decide upgrade eligibility following early closure.

### Concrete clinic scenario

A 26-visit package reaches its review date with 25 visits consumed because the patient missed a day. The clinic elects to end the package rather than permit discretionary further covered treatment. The user asks how this should be represented and what closure reason should be selected. This is a deliberate closure scenario; it does not establish automatic termination on the review date or a universal obligation to extend. The existing software model uses soft review and an explicit closure action.

### Alternatives, preserved separately

1. **Approach 1:** Keep Completed for exhausted allowance and Closed for deliberate termination, but add a suitable closure preset. The user considered a "completed" closure reason in the 25/26 scenario.
2. **Approach 2:** Add a manual Mark Completed action even when visits remain unused, so Completed can mean the package has ended by clinic decision as well as by consumption. Closed would then describe a narrower pre-closure concept.
3. **Approach 3:** Remove the Completed status and use Active/Closed. Automatic exhaustion sets Closed with an automatically populated "Completed" reason; other termination reasons are entered manually.

**User rationale:** These meanings affect how the system distinguishes allowance exhaustion from discretionary termination, and therefore how upgrade-after-closure eligibility can be expressed. No choice above is to be treated as approved merely because it was discussed.

## 18. Assistant recommendation on lifecycle — awaiting user's decision

**[ASSISTANT RECOMMENDATION]** Retain three clear states:

| State | Proposed invariant / meaning |
| --- | --- |
| Active | Package has not been deliberately ended or exhausted; coverage still checks start date, allowance, daily cap and any review confirmation. |
| Completed | All purchased allowances actually consumed; automatic transition when usage reaches the allowance. |
| Closed | Deliberately ended with unused allowances forfeited; mandatory reason, regardless of whether closure is before, on or after the review date. |

In the 25/26 scenario, record **Closed**, 25 used, 1 unused forfeited, with a descriptive reason such as **"Ended at review — unused visits forfeited"**. This is a **proposed additional preset**, not an update to the operative list. Its possible at/after-review availability must be selected if adopted. Do not reuse "Completed" to imply a 26th real treatment happened. The existing "Patient Stopped Attending" reason has its own overdue-only restriction and is not a universal fit for an on-review-date closure.

Closed need not mean "before the review date"; it means ended before all allowance was consumed. Review remains a separate date milestone. Keeping the review soft does not prevent admin from explicitly closing the package with a proper reason. Closure does not create a clinical visit or automatically settle the invoice/refund money.

**Trade-offs discussed by assistant:** Approach 1 retains a simple exhaustion invariant and the existing status model. Approach 2 can represent general administrative completion, but loses the ability to infer exhaustion from Completed and would need another distinction for upgrade/reporting rules. Approach 3 is viable if closure reason and actual usage are retained, but consolidates the distinction into reason data and requires every exhaustion-sensitive rule to inspect those facts. There is no demonstrated need to change the state model merely to add a better reason for unused-credit forfeiture.

**Upgrade consequence:** Keep the question "Was the original allowance exhausted, or were unused credits forfeited?" separate from the clinical judgment that treatment is finished. The user can still choose different upgrade rules for Completed versus Closed parents. A new preset or UI label must not decide those entitlement rules implicitly. After a choice, exact fields, constraints, automatic transition and reporting remain implementation work under the previously accepted guardrails.

**Status at this checkpoint:** Pricing basis is settled. The no-negative-charge/refund distinction is clarified. The lifecycle alternatives, proposed at-review closure reason, and upgrade-after-early-closure entitlement remain under discussion. Only this staged log is updated; no code, schema, operative topic page, tracker or uploaded upgrade source is changed.

## 19. Compact preset, closure timing and upgrade eligibility — follow-up

**Source:** The user's two response annotations on 5 November. The user finds "Ended at review with unused visits" too long for a preset and expresses agreement with Approach 1, **but explicitly wants to clarify closure timing and upgrade-after-pre-closure before locking it in**. Preserve this as conditional agreement, not a finalized lifecycle or upgrade decision.

**[ASSISTANT PROPOSAL]** Shorten the preset to **"Review closure"**. Keep the precise package usage/forfeiture figures in the closure confirmation and history, rather than encoding them in the label. The label is not yet user-approved. This preset is intended for a deliberate closure on or after review; other valid reasons remain available according to their permissions.

**[ASSISTANT RECOMMENDATION]** Treat exhaustion, manual termination and review timing as distinct facts:

- Completed: actual usage reaches the purchased allowance.
- Closed: deliberately terminated with unused allowance forfeited.
- For a Closed package, derive the timing from its recorded closure timestamp converted to clinic-local date, compared with that package's preserved review date: before = early closure; equal = closure on review day; after = post-review closure. Do not add another lifecycle state solely for this comparison or infer timing from a free-text reason. Exact fields remain schema work.

This avoids ambiguity in "early": a package may have unused visits both before and after review, but only the former is chronologically early relative to review. A short package's own review date uses its established ten-counted-date/Sunday rules. The upgrade purchase deadline is separately Day 30 from the original short package's start; those are not the same date or eligibility test.

**[ASSISTANT UPGRADE RECOMMENDATION, AWAITING USER DECISION]** For the bounded current upgrade design, permit the dependent 16-visit extension only after the original short package naturally reaches Completed, with purchase within its original inclusive 30-calendar-day window and no prior upgrade. Do not permit upgrades from manually Closed parents, whether before, on or after their review date. Such parents can instead be followed by an ordinary fresh package after termination under the existing replacement rules. This does not change the no-active-purchase/no-queue restriction.

**Rationale:** Manual closure deliberately forfeits unused credits. Allowing it to unlock an extension immediately introduces a choice about restoring those credits or granting fewer than 26 usable visits across the chain, and could make closure a workaround for the deferred active-upgrade path. A timing distinction is useful for history and reports, but crossing a review date alone should not determine upgrade entitlement. Exceptional closed-parent upgrades can be considered separately if the user wants them; none are approved here.

**Still awaiting user confirmation:** Final lifecycle choice; shortened preset wording; whether to adopt Completed-only upgrade eligibility or design a specific Closed-parent exception. No existing topic page, tracker, schema or application behavior is changed by recording this discussion.

## 20. Narrow review-closure upgrade exception — user challenges blanket exclusion

**Source:** The user's subsequent response annotation on 5 November. **[USER DIRECTION / EXCEPTION UNDER DESIGN]** The user mildly disagrees with excluding all Closed parents from upgrades. They distinguish a patient who missed two dates but continues treatment from someone who stopped attending/abandoned treatment. They want upgrades from some Closed packages to be considered under strict, narrow guardrails; exact eligibility and quantity are not yet selected. This rejects treating the assistant's Completed-only recommendation as settled policy.

The user refers to "closed after review." As discussed, this is a timing description of Closed, not an independently approved lifecycle status. **[ASSISTANT CLARIFICATION]** Neither passage of review nor absence alone automatically consumes visits or closes the package under the existing soft-review model. A later manual closure ends that purchase's coverage; timing alone cannot establish absence or distinguish continued treatment from abandonment. The user's proposed exception may recognize unused allowance through a new linked upgrade without reopening or fabricating usage in the original closed record.

**[ASSISTANT REVISED RECOMMENDATION; NOT YET APPROVED]** Explore an exception only for a same-complaint, same-category original 10-visit package explicitly closed at/after its own review for a review-related reason. Require upgrade purchase within the original inclusive Day-30 window, no prior upgrade, and no intervening replacement/other extension for that complaint. Exclude mistaken registration, abandonment, patient-requested termination and doctor-advised cessation from this particular exception unless separately approved. Admin is the only MVP actor. Original usage/closure stays immutable; invoice-price-difference basis and original long-review anchor remain intact. Eligibility and duplicate/allowance checks must be atomic and backend-enforced. These are proposed guardrails, not approved schema or an absence-tracking engine.

**[QUANTITY QUESTION OPEN]** For a parent closed at 8/10, a fixed +16 extension yields 24 actual covered visits across the chain. An upgrade promising 26 actual visits would instead grant 18 remaining visits (`26 - 8`), explicitly carrying the two unused allowances into the new entitlement. The assistant recommends the latter if the user's intended commercial promise is 26 actual covered visits overall; this would be a deliberate exception to the earlier fixed-16 child sketch. No parent visit is invented and no terminal record is reopened in either model. Await the user's choice before finalizing allowance/forfeiture semantics or implementation.

## 21. Fixed upgrade quantity and narrow confirmation flow settled; reason veto under discussion

**Source:** The user's four subsequent response annotations on 5 November.

- **[USER-SETTLED]** The upgrade extension is **always 16 visits**. A parent closed with 8/10 used still grants only 16 new visits through the extension, giving 24 actual visits across those two records. Do not restore or carry forward the two unused parent credits. This explicitly rejects the assistant's proposed 18-visit extension in Section 20 and settles its quantity question.
- **[USER-SETTLED UI DIRECTION]** Initiate upgrade only from the existing eligible original 10-visit package's detail/history view (the user describes going to the already Closed package and pressing Upgrade). No independent "purchase upgrade" entry flow or freely purchasable upgrade product. The system opens a narrow preselected confirmation dialog explaining what will happen, followed by Proceed. The operator cannot select a different complaint, category or custom date. The backend determines and enforces the parent, matching context, dates, fixed allowance and price; UI restrictions alone do not establish validity. This clarifies the Closed-parent flow without newly deciding an exclusion of the already discussed Completed-parent flow.
- **[USER QUESTION / OPEN POLICY]** The user challenges whether closure reasons should disqualify an upgrade, asks how this could be enforced, and considers relying on review dates instead. The proposed reason exclusions in Section 20 are not approved requirements.

**[ASSISTANT EXPLANATION]** Closure reason is separate from timing. Consider the same short package start of 1 November, own review on 11 November, closure on 12 November and upgrade request on 13 November, with 8 actual visits used. That chronology could describe review-related termination after missed attendance, a patient-requested termination followed by a change of mind, or a record closed because it was registered against the wrong complaint. All satisfy the same date comparisons but have different reasons. A date-only rule deliberately treats them alike for upgrade eligibility. It is not technically invalid; the assistant's earlier "insufficient" wording should not be read as a mandatory reason filter.

If the user chooses a reason-based veto, enforce against a stable structured reason identifier/action classification on the backend, not matching editable preset labels or free-text prose. This would validate the recorded reason, not prove the real-world circumstances or prevent an operator choosing another permitted reason. Exact representation and reason-policy selection remain open; no new table is required solely by this explanation.

**[ASSISTANT RECOMMENDATION, NOT YET USER-SELECTED]** For admin-only MVP, prefer objective timing/structural eligibility with the closure reason visible in the confirmation, rather than an extensive list of reason-based vetoes. Preserve the proposed safeguards: eligible original short parent, terminal lifecycle, any approved closure-versus-own-review boundary, original inclusive Day-30 purchase deadline, matching complaint/category, one upgrade per parent, no intervening replacement, fixed +16 allowance and immutable source records. Under that approach an admin can consider a returned patient even if the earlier reason was "Patient Stopped Attending." A record actually registered for the wrong complaint cannot be repaired or reassigned by an upgrade; correct registration/correction policy still applies. Review-date boundary and whether any closure reason automatically vetoes remain policy choices for the user, not silently finalized here.

The inherited Day-30 review anchor remains distinct from the extension's coverage eligibility date, and no custom-date input is approved. Negative-charge/refund separation and the settled original-invoiced-price basis remain as recorded. Only this staged log was changed; no UI, schema, operative topic page or tracker implementation is implied.

## 22. Locked decision — admin discretion without closure-reason vetoes

**Source:** The user's next 5 November response explicitly agreeing with the recommendation and asking to lock the discussion. **Status: USER-SETTLED.** This closes the reason-filter question in Section 21; earlier recommendations and exploratory alternatives remain historical context.

### Settled MVP policy

- **Admin-only MVP reaffirmed.** Reception is the user's **first planned post-MVP addition**, not an MVP application role. Earlier references to reception during a later/final MVP phase are superseded by this delivery clarification for the current discussion.
- Retain the objective date and structural eligibility gates for the discussed upgrade. **Closure reason does not automatically veto an otherwise eligible upgrade.** The admin sees the closure context and confirms whether to proceed.
- Original package remains a same-complaint, same-category 10-visit purchase. It must be terminal: naturally Completed, or Closed under the discussed on/after-own-review-date exception. Closing before its own review date is not newly admitted by removing the reason filter. Purchase remains within the original inclusive Day-30 window, with no prior upgrade or intervening replacement.
- Extension is always **16 additional visits**. Unused original allowances are not restored. Original visit, closure and invoice history remain unchanged. The long-review milestone retains the original coverage-start anchor; a new review cycle is not granted.
- Entry is through the eligible original package's Upgrade action and a narrow confirmation dialog. Show original package/complaint/category, usage and closure context where applicable, added 16 visits, calculated charge and applicable dates. Admin selects Proceed; complaint, category and custom dates are not selectable. No separate new mandatory upgrade-justification field is introduced by this decision.
- Backend eligibility, uniqueness, ownership and transactional safeguards still apply. Admin discretion over the closure reason is not a bypass of those gates and cannot reassign an incorrect parent to another complaint.
- Preserve the agreed lifecycle meanings: Completed for actual allowance exhaustion; Closed for deliberate termination with unused allowance forfeited and a mandatory closure reason; review timing is determined separately from the package's own review date. The admin can approve the discussed upgrade from an eligible Closed parent without reopening it.

### Future reason-policy review

**[USER-SETTLED FUTURE REVIEW]** Observe actual closure/upgrade patterns before deciding whether future reason-based restrictions are warranted. Custom written closure reasons provide useful context for that later review. This is a request to preserve the future consideration, not approval for new analytics, tracking tables, notifications, automatic classification, or a reason blacklist today.

Reception's future package-invoice debt restriction remains as settled in Section 10, distinct from the deferred question of whether closure reasons should limit reception upgrades. Queueing and its separate activation/cancellation design remain deferred. The user did not choose new reception reason restrictions in this reply.

**Scope of lock:** The date/structural-gate-plus-admin-confirmation policy and absence of a reason-based veto are settled. Earlier Section 19's Completed-only recommendation and Section 20's proposed reason blacklist are not current policy. Exact database/API/UI implementation, concurrency verification and the zero-charge upgrade integration still require the engineering work already identified. The compact "Review closure" preset label remains a wording proposal rather than a separately confirmed selection.

**Documentation action:** Appended this settled checkpoint to the active discussion log. No operative topic-page/tracker consolidation, application change, schema migration, stage, commit or push was performed.

## 23. First reading of Claude's upgrade audit and scratch prototype

**Date/source:** 5 November 2026. User requested an assistant first reading of everything in `upcoming/claude/`, identification of what the files cover, gaps, and matters needing attention/approval/discussion. Both files were read in full: [response.md](claude/response.md) (161 lines) and [scratch prototype and test results](claude/package_upgrade_prototype_and_test_results_2026-11-04.md) (912 lines). Their contents are external assistant analysis, not authority to change business policy, run SQL or implement a feature.

**Reading/verification boundary:** This is source review against the latest discussion, not an independent execution of Claude's tests. The supplied report claims ten test groups on scratch PostgreSQL 16.15. Its SQL, Bash driver and captured results were inspected. RLS, frontend, full refunds and deployed clinic state are explicitly outside Claude's verification. This first reading did not run the scripts or inspect a live database. It did not edit Claude's files or adopt their proposed schema/guards.

### What the files contain and the overall assessment

- `response.md` is an upgrade-design audit with twelve numbered findings, nine requested decisions, technical recommendations and a stated test boundary.
- The larger file contains a throwaway relational model, fixtures, illustrative purchase/visit/closure functions, invariant queries, six guards, captured outputs and a test driver. Tests demonstrate identity constraints, over-consumption races, queue handover/closure races, date boundaries, price arithmetic and zero-price cases in that model.
- **[ASSISTANT ASSESSMENT]** Useful mechanism demonstrations and test ideas; not an implementation ready for the clinic. The audit follows the November 4 upgrade log and tracked main `9670880`, not the later staged decisions in Sections 16–22. Most queue-related findings are real for that older model but are no longer current MVP decision requests. Reported weaknesses in an intentionally incomplete scratch model are not demonstrated defects in a deployed clinic application.

### Disposition of Claude's requested decisions

| Claude request | Current reading / recommended handling |
| --- | --- |
| Early upgrade purchase / Queued child | Already deferred by user. Do not reopen it for MVP. Stranded queue, automatic handover ordering, queued-parent closure cascade and chain review while the parent is still Active belong to future queue design. |
| Zero-charge floor and settlement | Floor already confirmed. Narrow invoice/settlement integration remains to be finalized; preserve separate issuance/return followed by settlement. Do not apply scratch auto-PAID at issuance as if approved. |
| Pricing basis | Already user-settled: current long-offering price minus original short-package purchase-line charge, floored at zero. Read the package's own line, not a combined invoice total. |
| Reject all Closed parents | Conflicts with Section 22. Eligible at/after-review Closed parents can upgrade; reason itself is not a veto. A new refund-related restriction would be a separate financial policy question, not grounds to undo the lifecycle decision wholesale. |
| Recreate an upgrade after its mistaken closure | Current one-upgrade-per-parent guardrail disallows this. Claude proposes a new zero-used-child exception. Do not silently loosen uniqueness; disclose the irreversible consequence and ask only if user wants a correction feature beyond the current rule. |
| Upgrade daily limit | Not explicitly selected for the extension. Recommend copying the parent's current `max_visits_per_day` when the upgrade is bought, retaining existing admin adjustment authority; user approval of that default remains appropriate. Do not silently convert every package's limit into a new universal complaint-wide rule. |
| Original Day-30 review anchor | Already settled. It stays the original coverage start + 29 calendar dates, independently of extension eligibility/start. No Active parent remains to raise the old early-purchase review-clock conflict. |
| Cascade closure to Queued child | Deferred with queueing. Not MVP work or an immediate question. |
| Include upgrade in MVP | Still assess engineering effort against approved scope. This prototype is not a current-policy implementation or proof that all production safeguards are ready. Do not treat scratch timing/performance claims as an implementation estimate. |

### Financial points worth user attention

**Refunded original purchase:** Claude's price table usefully shows that subtracting an unchanged original ₹1,800 line after its collected ₹1,800 was fully refunded still grants a ₹1,800 discount. This needs explicit treatment. **[ASSISTANT RECOMMENDATION]** Do not grant the old-price credit automatically after a full reimbursement that undid the original package purchase; establish that eligibility restriction before implementation. Partial refunds require a defined interpretation rather than switching the whole formula back to cash paid.

Claude recommends blocking if the original invoice has *any* refund. That is too broad to adopt without discussion: the invoice can contain unrelated charges, and the user's hypothetical long-price-drop scenario specifically contemplates a separate refund of the price difference. A refund's attribution/purpose matters. A blanket block would also need its interaction with that authorized refund scenario explained. This is a new proposed financial gate, not already approved by backend guardrail acceptance.

**Waived original charge:** With original charge ₹1,800 waived and current long price ₹5,000, the settled billed-price formula charges ₹3,200 for the upgrade. The original concession persists; total cash collected may therefore be less than ₹5,000. This is a consequence of the chosen formula, not an arithmetic bug or a reason to ask again whether billed or collected is the basis. Bring it to the user's attention; do not add back waived amounts or infer a clawback without a new decision.

**Zero-charge integration:** Claude correctly notes that current zero-total rules explicitly cover only package-covered treatment, while the user also confirmed a zero upgrade price floor. Recommend a narrowly validated computed-zero upgrade exception, potentially combined with valid covered zero lines, using the already familiar issue/return then Mark as paid flow with no receipt row. A mixed invoice with any positive charge follows ordinary settlement. This proposed amendment remains distinct from ordinary free/discounted package sales, which retain full-price issuance plus waiver. Claude's prototype instead sets PAID within `purchase_upgrade`; that conflicts with the currently separate issuance/settlement boundary and does not establish the correct UI/metadata flow.

### Code-reading findings and engineering work to carry forward

1. **Credit locking is not idempotency.** In captured T4b (prototype lines 584–588), the lock caps the parent at ten redemptions, but session 2 returns UNCOVERED and the function still inserts a second real visit (lines 279–287). If those requests represent the same treatment retry, duplicate clinical entries and a potential ordinary charge still result. Keep stable request/replay protection separate from allowance locking; do not claim the scratch lock solves double-submit recovery.
2. **Prototype line/source ownership is incomplete.** `invoice_line_items` has individual invoice/source FKs (lines 157–166) but does not tie invoice patient/clinic to the source patient/clinic. Composite parent/visit FKs are useful; the finance-source context still needs the agreed ownership enforcement.
3. **Six guards are partial.** Purchase-line completeness checks run only after package INSERT (lines 492–501); later line deletion is not covered. Visit allowance guarding runs only before visit INSERT (lines 503–524), leaving UPDATE/DELETE and immutable-history protection outside that guard. `packages_immutable` omits status/start-date changes, and Completed correctness is only queried in the invariant view, not enforced by the shown guards. These are reasons not to install the prototype as production code, not new business questions.
4. **Zero amount must be calculated, not merely typed.** `line_zero_guard` allows any UPGRADE zero line (lines 479–490); it does not prove the approved price difference actually equals zero. A caller who controls line amounts could bypass positive charging. Server calculation, price integrity and restricted writes are still required.
5. **Server time/context required.** Claude explicitly demonstrates accepting a forged `purchased_at` even after its guards. Prototype functions also expose lab-only time/lock controls. Production must not expose those as discretionary client inputs; evaluate dates in clinic-local time and preserve the no-backdating decision.
6. **Dynamic cap versus historical queries:** Its invariant view compares all historical daily usage with current cap values (lines 413–416). Because admin may edit a cap later, this is not by itself proof that past visits violated the rule then in force. Retain the already accepted MVP cap-change audit limitation rather than claiming complete historical validation.
7. **Tests do not cover the policy just locked.** The guards reject Closed parents and support Active parents/Queued children. RLS, authenticated role enforcement, real invoice/payment/refund integration, end-to-end retry handling and the exact terminal-only/review-Closed variant were not independently verified. Capture the useful tests for a later implementation suite after updating the model to current policy.

### Attention filter for this first reading

Bring forward refund-related eligibility, the narrow zero-upgrade settlement exception, and the extension's daily-cap default as the useful policy/default choices. Also explain the carried-forward waiver concession and one-upgrade mistake consequence. Do not deliver Claude's nine-item list as nine unanswered questions: several are settled, deferred, or contrary to current choices. Mechanical FK/locking/permission/test defects are engineering work to prepare for review, not new clinic-owner policy questions.

**Source fingerprints at first reading:** `response.md` SHA-256 `23411aef565debf3648b09a534d36e549809dc0954f6cc8e27ed8ded04fe5ac1`; prototype/test report SHA-256 `9d5cbe36b661a3e7fa1fe52e5c807ed836d1afb93b5ab753ac9c6f1bbc7e7398`. Only this staged discussion log is appended. No new policy approval, source alteration, topic/tracker consolidation, SQL execution, code change or Git mutation is implied.

## 24. User responses to the Claude review — refunds, limits and upgrade finality

**Source:** Seven user response annotations on 5 November, addressing Section 23's first-reading summary. Preserve the distinction between new confirmations, clarification requests and proposed policy.

### Clarification of earlier refund example and terminology

- The user asks what "separate price-difference refund example" meant. It refers to their earlier hypothetical original short charge of ₹1,800 and current long price of ₹1,500: no negative upgrade invoice, and any actual ₹300 cash return would be handled separately through the refund workflow (Section 16). It is not a second invented scenario or an automatic catalog-price-drop refund.
- The user objects to "old-price credit" as unclear jargon. **[ASSISTANT CLARIFICATION]** It meant only the **₹1,800 original package-line amount subtracted from the current long price**. Use that concrete description instead. No new wallet, credit balance or financial entity was intended or approved.
- **[USER OBSERVATION / RECONFIRMED MODEL LIMITATION]** A refund recorded against a combined package/exam invoice cannot reliably be assigned to the package by matching its amount to the package price. A ₹1,800 invoice refund does not prove that precisely the package's ₹1,800 charge was refunded. No line-level allocation may be inferred from equal numbers or free-text reasons.
- **[ASSISTANT CLARIFICATION]** A refund returns money actually received. Forgiving an amount still owed is a waiver. The current question concerns attribution of returned money within a mixed invoice, not treating a refund as an automatic reduction or cancellation of a particular charge.

### Refund information in upgrade confirmation — direction under discussion

**[USER DIRECTION / REQUEST FOR OPINION]** Package closure and refund are independent actions/workflows. Closure does not always cause a refund. The user questions whether any refund-based upgrade block can be reliable, and proposes retaining package date/structural eligibility while showing the amount refunded against the associated invoice in the admin confirmation. The user expects many refunds to relate to pre-review termination but does not establish a rule that refunds can only happen before review.

**[ASSISTANT RECOMMENDATION]** Agree with nonblocking invoice-refund context rather than a refund-amount/reason-based veto under the current unallocated invoice model. Show the original package charge, associated invoice reference, invoice-level recorded refund total and calculated upgrade charge; identify refunds as against the **invoice**, not necessarily the package, when the invoice has multiple charges. Leave admin to decide after reviewing the invoice. This applies the already chosen admin-discretion approach without silently changing the upgrade-price formula.

The assistant's Section 23 recommendation to block after a fully reimbursed original purchase cannot be automatically implemented from invoice-only refund amounts; attributable package reimbursement is not reliably established by those values alone. Do not add Claude's blanket any-refund block or claim this user proposal is a selected line-allocation design. Package closure timing can be enforced, but must not be used to infer that a refund did/did not happen. Exact refund display/API integration remains engineering work; final locking of the refund-specific presentation direction has not been explicitly requested in this reply.

### Zero upgrade versus ordinary complimentary package — explanation requested

The user asks what the assistant meant when contrasting scratch auto-PAID behavior with current settlement. **[ASSISTANT CLARIFICATION]** The difference is between (a) the existing issue/return invoice first, then a separate Mark as paid action for a permitted zero-total case, with no payment row, and (b) Claude's scratch `purchase_upgrade` setting PAID immediately during purchase creation. This is a policy/prototype comparison, not a verified claim that a live clinic feature is missing.

An ordinary gifted package is a separate example: invoice the normal positive catalog price and subsequently waive it. It must not be confused with a calculated-zero upgrade or with an upgrade whose *original* short invoice was waived. The extension of the permitted zero-total types and exact zero-upgrade settlement remains the previously identified implementation/policy integration item; no automatic settlement change is approved by this clarification request.

### Daily limit — explicitly accepted

**[USER-SETTLED]** Copy the original package's **current `max_visits_per_day` at upgrade purchase time** into the extension. Retain admin's established ability to adjust the extension's limit afterward. This is a one-time copy, not a permanent live inheritance link: later parent edits do not silently change the extension. This closes Section 23's daily-limit default question; exact schema remains implementation work.

### Original waiver — consequence confirmed, no new question

**[USER REAFFIRMED EXPECTATION]** The original waiver remains a concession. An original ₹1,800 charge that was waived is still the ₹1,800 subtracted from a current ₹5,000 long price; upgrade charge stays ₹3,200. This follows the already settled original-invoiced-charge basis. The user asks why this was presented as needing attention. **[ASSISTANT CORRECTION]** It was an explanatory consequence, not an unanswered business decision requiring another approval. Do not add back waived amounts or re-ask the pricing-basis question.

### One upgrade per original — terminal finality explicitly settled

**[USER-SETTLED / REAFFIRMED]** One upgrade opportunity per original package. The narrow UI contains no selectable complaint/category/date and does not create a separate routine "wrong upgrade" recovery path. If the upgrade is later deliberately Closed for any reason, it stays terminal and the original's one upgrade opportunity has been used. No second upgrade, reopening or zero-used-child exception is permitted, even if no extension visits were consumed. Reject Claude's proposed retry-purchase exception for the current design. The possibility of a UI misclick does not create an approved correction feature.

**Checkpoint:** Daily-limit copying and finality are settled; waiver arithmetic is reaffirmed. Refund-aware confirmation is the user's proposed direction supported by the assistant, not an approved automatic block or refund attribution scheme. Zero-upgrade flow was explained rather than silently changed. Only this staged log was appended; original Claude files, topic pages, tracker, code, schema and Git history were not changed.

## 25. Final two policy approvals and compact engineering checklist

**Source:** User's subsequent 5 November response annotations: explicit "yep" approving both outstanding recommendations, clarification that their completeness question concerns policy, and request for a compact implementation checklist.

**[USER-SETTLED] Refund-aware confirmation:** Display the related invoice's recorded refund information when confirming an upgrade. Do not automatically block the upgrade because that invoice has refunds. Admin decides whether to proceed after reviewing the context. Label invoice-level refunds accurately rather than attributing them to a package without evidence. All other agreed eligibility gates remain enforced.

**[USER-SETTLED] Calculated-zero upgrade:** Permit the narrow computed-zero upgrade case. Issue and return the invoice first, then offer Mark as paid in a separate settlement action with no payment row and zero collection. Extend the permitted zero-total cases accordingly; do not authorize arbitrary zero-priced standard package sales. Mixed invoices with positive charges follow ordinary settlement. This supersedes the pending status in Sections 23–24.

**Policy-review checkpoint:** The assistant has no known unresolved business choices within the package/upgrade scenarios reviewed in this discussion. This is a scoped completeness assessment, not proof that every conceivable clinic policy is specified. Previously deferred topics (queueing, reception implementation/future restrictions, old-debt management and other future work) remain explicitly deferred. New contradictory evidence or genuinely unspecified business behavior discovered during implementation should be brought to the user as a concrete scenario; mechanical design work is not grounds to reopen settled policy.

**Engineering checklist — to implement and verify, not claimed complete:**

- [ ] Reconcile the staged settled decisions into operative topic pages/register/tracker when that incorporation is requested, retaining superseded-source attribution.
- [ ] Finalize `package_catalog`, purchased packages, parent extension, visits and typed invoice sources; enforce patient/clinic/complaint/category agreement and line completeness.
- [ ] Enforce lifecycle and upgrade eligibility: terminal original short purchase, qualifying Closed timing, inclusive Day-30 deadline, fixed +16, one upgrade per parent, no intervening replacement, no reopening and preserved original review anchor.
- [ ] Implement server-derived, clinic-local dates, correct short-review Sunday counting, coverage start versus review date, and no client date/price/context overrides.
- [ ] Calculate upgrade from the original package's purchase-line charge; preserve issued prices and separate payments/refunds/waivers; implement confirmed computed-zero issuance/settlement and combined-invoice handling.
- [ ] Derive used/remaining visits from real committed linked visits; copy the parent's current daily limit at upgrade purchase; enforce start date, daily limits and automatic completion without altering historical records.
- [ ] Coordinate purchase, closure, credit use and applicable cap changes transactionally; protect the last credit, one-active-package rule and simultaneous upgrade purchases.
- [ ] Implement stable submission/replay protection for lost responses; prevent duplicate visits, purchases, lines or settlement, including avoiding a retry silently creating an uncovered charged visit.
- [ ] Enforce authenticated admin-only MVP authorization, controlled write paths, immutable history and invoice/source ownership beyond UI restrictions.
- [ ] Build the parent-only confirmation UI, accurate invoice-refund context, zero-total settlement action and required precommit checks/error recovery.
- [ ] Reconcile the actual deployed/schema starting point and prepare migrations/rollback without importing legacy paper packages or debt contrary to the clean-launch decision.
- [ ] Execute targeted integration/concurrency/boundary tests, including final-credit races, duplicate submits, same-day transitions, Day 30/midnight, Closed/Completed eligibility, refunds shown without blocking, waiver arithmetic and zero settlement without receipts.

Only this staged log was appended. No implementation, tests, live database verification, operative-record consolidation or Git mutation was performed by this checkpoint.
