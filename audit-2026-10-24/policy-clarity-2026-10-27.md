# Decision-policy clarity audit — completed 29 October 2026

Status: COMPLETE FOR CURRENT-POLICY CLARITY THROUGH S24. This report audits the meaning of the operative decision files. It is a companion to the earlier architecture audit, not a replacement for the operative decision pages. No reviewer finding approves a new business rule; subsequent owner clarifications are explicitly attributed.

## Reviewed state and preflight

- Target: `omsrivastava512/clinic-demo-be`, `study/module_2/decisions/`.
- Live upstream default branch: `main` at `f19793f683e06a289f1f50003d8588027aff6425`, checked through the connected GitHub API on 27 October.
- Current local backend `main`: `92647cfacdb330f19ae39a27444c821954a8e87b`, exactly 2 commits ahead and 0 behind live upstream. This includes `e424e85` reconciliation and the subsequent `92647cf` final review-answer commit. That was the clean reviewed base before the authorized S23 fee clarification was promoted below. The earlier checkpoint had reviewed these bytes while they were local edits; their later commit does not alter the findings. Existing changes elsewhere are outside this policy audit.
- Frontend local/live `main`: `e2c2f4db9fb565ee3562029d642be4c98220abae`, 0 ahead and 0 behind. Its pre-existing ClinicalTimeline change and other local material are outside this policy pass.
- All-state backend PR listing returned no PRs. Searches across both repositories found the relevant backend decision commits `d5ed39d`, `4a66bfa`, `f19793f`; no equivalent policy-clarity audit was identified. Existing frontend PRs 95/96 concern implementation work, not approval of these later clinic policies. No ticket ID was supplied.
- No branch update is needed. No Git mutation, application change or migration is part of this audit. The later explicit owner fee clarification is promoted only into the current decision layer and a new attributed S23 record.

## Current review scope and disposition

The owner clarified that the real decision files are the audit target. History is retained evidence and may contain gaps, competing statements and superseded rules. A historical inconsistency alone is not a finding. Historical files will not be repaired or harmonized, and exhaustive historical reading is not a completion gate for this current-policy audit.

The operative review covers all six topic pages, README, the 82-row current reconciliation register and the workbook's current decision/issue layer. The source coverage CSV records broader evidence consultation without turning every consulted historical paragraph into current policy.

- **Owner clarification received:** PC01 is resolved by the latest examination/lapse-fee explanation. PC17 records the completed billing-only and visit-association consistency work and current-layer promotion.
- **Owner clarification resolved:** PC08, through the 29 October S24 decision: future reception may take initial and later receipts, while only admin may waive. The current phase is admin-only.
- **Current-wording corrections completed:** PC02, PC04, PC05, PC07, PC12 and PC16. PC09 is incorporated by S23.
- **Current presentation corrections completed:** PC15.
- **Cleared or explicitly bounded work:** PC03, PC06, PC10, PC11, PC13 and PC14. These are not additional policy defects.

The S23 fee clarification and evidence-backed wording corrections are incorporated in the current policy pages/register/tracker. These corrections carry forward accepted rules and clarify parked boundaries. S24 now resolves PC08 and explicitly expands future reception authority. All historical source logs remain unchanged.

## Method and scope

For each current rule, check its actor, triggering event, affected records, allowed outcome, exceptions, finality and interactions with other rules. A concrete scenario must expose different outcomes before ambiguous wording is called a policy gap. Separate three results:

1. An accepted rule is missing or weakly stated: propose tighter wording using its existing evidence.
2. Sources do not determine the required clinic behavior: preserve the gap and obtain the owner's decision.
3. Behavior is settled but its implementation is unfinished: keep it in the engineering audit; do not ask the owner to reapprove an already settled rule.

The six topic pages, README and all 82 reconciliation decision rows have been read. All 53 current issue dispositions and 32 illustrative workbook acceptance cases have been inspected. The coverage register records which historical sources were read fully or selectively, and which still require reading. The current-file review is complete at the recorded revision; historical consultation is evidence only and is not claimed to be exhaustive.

## Provisional model used to test the policies

A complaint belongs to the patient and has a permanent, manually assigned ROUTINE/REHAB classification. A treated complaint produces its own visit and one flat therapy line, with bundled procedures recorded clinically. An independently performed standalone service is a separate patient-linked, anatomical occurrence with no fabricated complaint visit. Service identity is catalog-backed; its clinical display name is joined, while invoice descriptions and amounts are frozen.

Checkout issues confirmed performed-care records when present, applicable automatic fee events, invoice and lines atomically. A fee-only checkout does not require a fabricated clinical examination or complaint-visit. The complete invoice is returned before a separate payment/waiver transaction. Positive Cash/UPI tenders are immutable payment rows. PAID and WAIVED are terminal; receipts and forgiven balances remain distinct. Packages are required for MVP but their eligibility/activation/redemption policy is incomplete. A verified package-covered-only zero invoice has its own manual Mark as paid closure and creates no receipt.

This model describes intended policy. No deployed-schema or enforcement claim follows from it.

## Owner clarification added to the audit scope — examination/lapse fees

The owner's latest message extends the active audit with an explicit meaning and a targeted consistency check. It is attributed in S23 and incorporated into the current topic/register/tracker layer. Earlier historical files remain unchanged.

1. The application records the applicable **fee**, not proof of a performed or thorough clinical examination. “Soft clinical record” here does not mean transient, optional or freely editable financial data; existing issued-fee/invoice immutability remains in force.
2. First-time and recurrent examination fees follow the same fee-event and fee-only-checkout rules. Actual routine clinical assessment is ongoing and does not itself create these fee events.
3. The fee event must be independent of a complaint-specific visit, just as independently recordable standalone care does not need a fabricated complaint visit. This does not merge billing fees into the actual standalone-treatment clinical entity or select new DDL.
4. Check all operative examination wording for accidental claims that it documents a clinical procedure, and check whether a fee is embedded in visits or requires visit_id/visit_services.
5. Preserve the existing patient-wide calendar bands, one fee per attendance, automatic calculation during issuance, separate settlement, and the fee-only-attendance recency rule.
6. Historical logs may retain old models. They are not to be repaired; older schema/planning material is consulted only to answer the owner's explicit visit-association check.

The fee-model check was added to the existing clarity goal. That checkpoint did not create a separate goal, application change or migration.

## Findings

Finding evidence and line references below describe the reviewed wording before correction; each disposition records its current outcome. Applied scope and final verification appear below.

### PC01 — Resolved: examination is a fee term, not a five-day clinical-exam workflow

**Disposition:** RESOLVED by the owner's latest clarification in this chat. Stop asking the earlier five-day question.

The five-day example meant five calendar days since the last qualifying completed attendance. The owner rejects treating that as a new system “examination”: within 0–9 days there is no examination/lapse fee event. Ongoing physiotherapy assessment is not the clinical fact represented by this fee model.

“Examination fee” and “late/lapse fee” are billing terms. A fee record states what the clinic charged under the first-attendance/return-gap rule; it does not assert that a clinical examination occurred or how thorough one was. The already selected extra_fees charge-event direction remains separate from actual performed-treatment occurrences.

An applicable fee can be checked out without therapy. This supports a first ₹350 fee-only checkout followed by therapy on another day, and the same fee-only workflow after a qualifying 30+ day return. No complaint-specific visit or visit_services record is to be fabricated or required for it. Fee-only attendance continues to count for the existing recency rule; payment alone does not reset recency.

The settled bands remain: first qualifying attendance ₹350; return gap 0–9 days no fee; 10–29 days ₹150; 30+ days ₹350, never stacked. This clarification does not create another zero-total invoice exception, a discretionary fee override or a clinical-save-first path.

PC17 below records the current-document consistency work and checked repository evidence resulting from this clarification.

### PC02 — Older UNPAID receipts explicitly covered

**Disposition:** CORRECTED. Current R39 explicitly covers positive receipts on UNPAID and PARTIALLY_PAID invoices with collectible balance. S24 subsequently permits the planned reception role, as well as admin, to record those later receipts; the older manual admin-handoff requirement is superseded.

A ₹500 UNPAID invoice receiving ₹200 becomes PARTIALLY_PAID with ₹300 collectible. Further positive receipts remain allowed until full collection closes PAID or admin waives the residual. Only admin may waive; no reception waiver or invoice-line mutation is approved. Current evidence: payments/R38–R40/N17 and S24.

### PC03 — Historical zero-receipt Due behavior is not a current policy defect

**Disposition:** CLEARED under the owner's current-file-only clarification.

The current payment page and R37 explicitly define PARTIALLY_PAID as positive receipts with a remaining collectible balance. Older logs allowed a zero-receipt Due interpretation; that difference is acceptable historical evolution. It does not authorize a current zero-receipt PARTIALLY_PAID transition.

No historical correction or extra reviewed-debt workflow is requested. The current four state meanings govern. Broad compatibility wording in R75/R81 must be read within that current authority; it is not permission to import an obsolete historical transition.

### PC04 — Package direction sits under a “Settled” heading

**Disposition:** CORRECTED: the package heading distinguishes scope and boundaries; activation policy remains parked.

Evidence: packages lines 5–9 and 19–25; R62/A11 explicitly leave detailed activation and redemption to the dedicated package discussion.

The heading “Settled MVP position” contains both settled scope and tentative activation/pricing direction. Readers should not mistake that heading for approval of partial-payment or waiver activation. Historical activation wording is context, not authority to settle the current parked choice.

**Applied wording:** rename that heading to “MVP scope and current boundaries”. Use the combined line-7 replacement in PC05 so the activation-status and invoice-grouping clarifications are applied together.

This clarifies status without choosing an activation policy or reopening package design here.

### PC05 — “Separately billed” can be mistaken for a separate invoice

**Disposition:** CORRECTED: a distinct purchase charge does not imply a separate invoice. Combined invoice grouping and same-checkout coverage remain parked.

Evidence: packages lines 7, 22, 24 and 30; R62/A11.

Scenario: package purchase and treatment occur during one checkout. “A purchase is separately billed” can mean either a distinct billed fact or a compulsory second invoice. The same page leaves combined purchase/treatment grouping parked.

**Applied combined replacement for packages line 7 (PC04 + PC05):** “Offering definition, purchase/entitlement acquisition, and redeemed treatment are different facts. A package purchase is a distinct billed event from treatment redemption. Activation after full payment is recorded direction; detailed activation and redemption conditions, including partial-payment and waiver effects, remain parked. Whether purchase and treatment share an invoice, and whether the purchase can cover treatment in that same checkout, also remain parked in the dedicated package discussion.”

Do not require a second invoice or assume immediate coverage.

### PC06 — Historical ₹200/₹300 values are not a current-policy defect

**Disposition:** CLEARED as a missing-current-tariff finding.

The current policy requires one configured ROUTINE flat rate and one configured REHAB flat rate, with immutable issued amounts. R09 does not adopt a new numerical tariff; rate storage and configuration are explicitly unfinished. Historical ₹200/₹300 values cannot, by themselves, be inserted as current binding prices.

No historical-price restoration or hardcoded fallback is proposed. The eventual implementation must obtain a valid configured rate; current catalog edits must preserve issued snapshots. Exact configuration mechanics remain an engineering audit rather than a new clinic-price vote.

### PC07 — Enabled/disabled availability needs an operative selection rule

**Disposition:** CORRECTED from accepted source evidence: new selection is enabled-only and backend availability validation is explicit; concurrent effective-time work remains audited.

Evidence: catalog lines 8–10 and R02/R10 require current availability and admin-only mutation but do not directly state how unavailable IDs affect new recording. Targeted source verification at S02 §1 and the O06 brief §4 A1 records active-only selection and independent backend eligibility validation; the single-clinic change does not revoke that requirement.

Scenario: a client submits a disabled service ID directly. Merely storing an availability flag and preserving old history does not explicitly say whether the new entry is rejected.

**Applied sentence after catalog line 9:** “For new service entry, the picker exposes enabled services and the backend independently validates service availability; direct submission of a disabled service ID must not bypass that restriction. Disabling a service preserves existing clinical references and issued invoices.”

Concurrent disablement and already-performed-but-not-yet-issued work remain a separate effective-time/recovery audit. No retrospective override or deletion is approved.

### PC08 — Initial and later collection authority — resolved

**Disposition:** RESOLVED by the owner's 29 October reply, recorded as S24.

Reception may resume interrupted initial collection after checking the stored outcome. Pay Later ends that initial interaction, but future reception may independently record later positive receipts on UNPAID/PARTIALLY_PAID invoices, today or on another day. No admin handoff or initial-checkout-only cutoff limits those receipts. Only admin may waive.

The current operating/delivery phase has one application role, admin. A separate reception role and its authorization/RLS are planned for a later/final MVP phase. This is intended phasing, not verified deployment state. R38–R40, A07/N17/N47, the current payment page, README and tracker reflect it.

Existing no-overpayment, terminal-state, immutable-line/payment, server timestamp/recorder, separate-settlement and duplicate-prevention requirements remain. Exact role/RLS and recovery mechanisms remain technical audits. No further owner decision is required for this clarity pass.

### PC09 — Clarify vocabulary without inventing encounter mappings

**Disposition:** INCORPORATED by S23. Current R15–R17 and the fee page now state patient-wide completed attendance, including fee-only checkout and therapy-only attendance, independent of payment. No additional glossary edit is required; exact record mapping remains A04.

Evidence: examination lines 14–16 and 35–37; R15–R17/A04.

The current policy distinguishes complaint-visits, physical attendance, completed clinical encounters, checkout and invoice, but uses these terms across pages. Its patient-wide recency rule is settled. Actual stored-event/completion mapping, same-day handling and local-date mechanics remain explicitly audited.

**Suggested clarification:** “A complaint-visit is care for one treated complaint on an occasion. Several complaint-visits may occur during one physical attendance. Recency is based on the patient's qualifying completed attendance, including fee-only checkout and therapy-only attendance, independently of invoice payment or waiver status; it is not measured from invoice settlement. The examination-fee event is not proof of an actual clinical examination. Map actual stored record types during A04.”

No generic attendance table, extra qualifying restriction, new appointment module or changed fee band is approved.

### PC10 — Package design is explicitly parked, not ambiguously approved

**Disposition:** KNOWN PARKED WORK; not a defect merely because it is unfinished.

Packages remain required for MVP. Current files explicitly leave duration/session meaning, confirmed offerings/prices, activation, coverage, expiry/exhaustion, imports, same-checkout use and unapproved lifecycle details to A11.

This clarity audit does not need to finish that separate design discussion. It must preserve a clear boundary: none of the unselected alternatives is an operative package policy. PC04–PC05 address confusing current phrasing around that boundary. The parked work prevents a claim of completed MVP design, not completion of reviewing its documented status.

### PC11 — Complaint closure is explicitly parked

**Disposition:** KNOWN PARKED WORK; no new policy finding or question here.

Clinical lines 9–10 and 28, R08/R70 and A05 clearly preserve permanent classification while leaving closure/inactivity and replacement-complaint consequences open. That does not approve automatic closure, mandatory closure or indefinite-open behavior.

The current framing already distinguishes the settled rule from the parked lifecycle choice. No historical cleanup or closure decision is required by this audit.

### PC12 — Bound “clinical records” to the submitted checkout

**Disposition:** CORRECTED: atomic issuance expressly covers that checkout's confirmed performed-care records when present; fee-only checkout requires no dummy care.

Evidence: clinical line 19; checkout line 7; R22–R24/R77. The stated requirement concerns the confirmed clinical work submitted in the checkout.

“Clinical records” is broad enough to be mistaken for patient registration, catalog setup, complaint registration or every longitudinal note. The current files separately describe registration/catalog prerequisites.

**Applied bounded scope for checkout line 7:** “MVP checkout atomically saves that checkout's confirmed performed-care records when present, applicable extra-fee records, invoice and invoice lines, or none of them. Completed care submitted through this checkout cannot be saved first and billed later. Settlement follows after the complete issued invoice is returned.”

Apply the same bounded noun phrase in clinical line 19 and R22/R23/R77. This does not authorize a clinical-save-first path or choose persistence rules for unrelated registration work.

### PC13 — Checked: the provisional typed-source status is supported

**Classification:** CLEARED after further source review; no policy correction requested.

The earlier S02 typed-FK direction and the current audited default initially appeared to differ in strength. Full reading of S15 §3.7 then located the recorded user qualification: retain the approach as the default until the user explicitly changes it. The current “working default / audit required” wording is supported by that evidence. The earlier portion of S15 is a historical reconstruction, not a newly verified raw quotation.

Continue preserving mandatory source integrity, ownership and once-per-occurrence outcomes while exact physical details remain audited. No competing architecture is approved. Do not treat this cleared observation as an additional business-policy question.

### PC14 — Recovery and metadata are explicitly unfinished engineering

**Disposition:** KNOWN IMPLEMENTATION AUDIT; not a current policy contradiction.

Current pages already require issuance before settlement, manual Mark as paid for the permitted covered-only zero invoice, no fake payments, backend-confirmed success and safe handling of unknown outcomes. Exact recovery, nonterminal metadata, API/error states, locking and idempotency remain A07–A09.

The clarity audit does not choose a new state, automatically close zero invoices during issuance, create a receipt, waive zero balances or select a replay mechanism. Those would alter settled requirements or exceed this review.

### PC15 — Small current presentation corrections

**Disposition:** CORRECTED in the current reading layer: current-register reference, audit-and-parked workstream label, and settled original-question status framing.

- Workbook Decision Register H37 (C34): replace “through S18” with “through S22” or a direct reference to the current register.
- Workbook Overview A23: replace “Thirteen technical audit workstreams” with “Thirteen audit and parked workstreams”; A05/A11 include business choices.
- Reconciliation line 110: qualify USER REVIEW/APPROVAL EVIDENCE as historical categories, or remove the stale sentence now that Q01–Q07 are closed.

Preserve historical source files, workbook A–F attribution and the historical schema inventory. Historical text describing a now-corrected clinical-first exception does not require cleanup.

### PC16 — “May share” weakens the settled shared-invoice requirement

**Disposition:** CORRECTED: same-checkout therapy, standalone care and applicable fees share the invoice as distinct lines. One-invoice-per-day remains unapproved.

Evidence: clinical line 16 says standalone care delivered with ordinary therapy shares the checkout invoice; R25 says it uses that same invoice. Checkout line 11 says the sources “may share” an invoice, and the clinical scenario at line 40 says cupping “can share” it.

Scenario: therapy and separately performed cupping are delivered together. One reader issues one invoice; another treats separate invoices as equally approved. The current settled requirement supports the shared invoice.

**Applied replacement for checkout line 11:** “Complaint therapy, standalone care delivered with it, and an applicable examination/return fee appear as distinct lines on the same checkout invoice. One standalone line represents one actual performed occurrence; individual standalone sessions are not presold.”

In the clinical scenario use “shares their invoice” rather than “can share their invoice”. This does not impose one invoice per day or decide the parked package-purchase grouping.

### PC17 — Align current examination wording with the clarified billing-only meaning

**Disposition:** PROMOTED from the owner's explicit clarification as S23 into the current topic/register/tracker layer. The documented repository association check is complete; exact new schema and implementation remain unaudited.

**Starting decision wording before S23 promotion:** examination-fees lines 22–25 already use extra_fees and say a charge is not proof of a clinical examination. However, its lines 3/31/35 still leave an “actual clinical examination” representation open. Clinical-records line 25, checkout lines 12/28, reconciliation R18/R69/A01/N41 and the README topic description repeat that clinical-model question. Those passages have been replaced with independent billing-fee events and fee-only checkout. They no longer ask this fee policy to select a clinical examination table or put a fee under a complaint-visit.

**Repository check at backend 92647cf and frontend e2c2f4d:**

- [Documented migration](../docs/schema/supabase_migration.md), lines 635–664: visits requires complaint_course_id and stores consultation_fee_in_paise on that complaint-specific visit. This is an older documented model, not live database proof.
- The same document, lines 697–725: visit_services requires visit_id; reusing that entity for the new fee would impose the unwanted complaint-visit dependency.
- Its invoice visit_id at line 805 is nullable. That historical invoice field does not prove an independent extra_fees record has a visit FK.
- [Schema status map](../docs/schema/schema_status_map.md), dated August 30 and explicitly a one-time snapshot, lines 11–18 and 205–218, proposes exam_fee_in_paise and lapse_penalty_in_paise on visits. This is an older planning proposal conflicting with the current fee-event direction, not evidence it was deployed.
- The frontend Visit type at clinic-demo/src/types/index.tsx lines 129–144 also places consultationFee on a complaint-linked visit. Its patientUtils.ts lines 196–202 use the older FIRST/SUBSEQUENT 300/200 calculation. These inspected source paths do not implement the new 10/30-day fee contract.
- No extra_fees table definition or implemented new examination-occurrence table was found in the inspected schema documents. This is a bounded repository finding, not a claim about deployed Supabase.

**Accepted meaning carried into the current layer:**

> “Examination fee” and “late/lapse fee” name automated billing charges, not performed clinical procedures. An extra_fees event records the applicable first-attendance or return-gap charge and its calculation evidence. It belongs to the patient and checkout; it is not a child of a complaint-specific visit and does not require a visit_services row, a fabricated visit or proof of an actual examination. First-time and recurrent fee-only checkouts are supported without therapy and follow the same invoice/settlement rules. Confirmed performed-care records, when present, applicable fee events, invoice and lines are issued atomically. No fee event is created for a 0–9-day return merely because routine clinical assessment took place.

**Completed current-layer alignment, limited to this clarification:**

- examination-fees: make billing-only meaning explicit at the start, use fee-only attendance terminology, retain the precise fee bands, and replace the actual-clinical-exam-model question with fee-event independence/recency/source enforcement work.
- clinical-records and checkout: distinguish actual performed-care records from fee events; allow fee-only issuance with no fabricated clinical occurrence or complaint-visit. Keep extra_fees separate from actual standalone treatments.
- README and reconciliation: carry this meaning into R13–R20/R22–R23/R34/R69/R76–R77, Q03, A01/A04 and affected current issue descriptions. Preserve old source descriptions as history.
- Tracker current columns/issues/acceptance cases: mirror the revised rules and demonstrate first fee-only checkout followed by next-day therapy, recurrent 30+-day fee-only checkout, no new fee at five days, and no required complaint visit. Preserve A–F historical attribution and unexecuted-test labels.
- Exact fee-event columns, patient/clinic/invoice constraints, indexes, recency lookup and concurrency controls remain engineering audit. No proposed SQL in old documentation is executed or silently adopted.

**Consistency scenarios:** a first fee-only ₹350 checkout on Monday followed by therapy Tuesday has no new gap fee Tuesday; a 30+-day return may likewise have a ₹350 fee-only checkout with therapy later; a five-day return creates no examination/lapse-fee event. Fee-only checkout counts as completed attendance for the already settled recency rule, regardless of when its invoice is paid. No payment event or arbitrary fee insertion may advance the attendance clock.

The promotion changes examination-fees.md, clinical-records.md, checkout-and-invoice-lines.md, README.md, reconciliation-2026-10-27.md and 61 workbook cells, and adds history/clinic_module2_examination_fee_semantics_2026-10-27.md as S23. The original historical files, application code and schema documents remain unchanged. Fee-only support supplies the performed-care-when-present qualification discussed in PC12; the later carry-forward correction now also bounds the records to that checkout's confirmed care.

## Coverage of current policy rules

Every current R-ID is included once in the following review groups. This register accounts for the rule review; matching counts and hashes alone do not establish semantic correctness.

| Current rules | Reviewed meaning and boundary | Result |
| --- | --- | --- |
| R01–R03 | Single-clinic catalog, future branch independence and deferred migration | Clear current/future boundary; no branch architecture added |
| R04–R09 | Complaint-wise flat billing, bundled procedures, permanent manual classification and two rate classes | Clear; historical numerical tariffs are not adopted through PC06 |
| R10–R12 | Catalog-backed identities, admin service mutation authority and complaint-addition trail | PC07 makes active-service entry restrictions explicit; no new roles |
| R13–R21 | No scheduling, new-complaint scope, exam-only workflow, patient-wide day bands and automatic fees | PC01 resolved by owner; PC17 billing-only/current-wording alignment pending; settled fee bands/visibility retained |
| R22–R24 | Atomic clinical/billing issuance followed by separate settlement | PC12 bounds the clinical-record phrase; no earlier clinical persistence |
| R25–R35 | Shared invoice, standalone identity/anatomy, frozen lines, zero-covered exception, real sources and uniqueness | PC16 corrects optional wording; source-model/audit boundary is clear |
| R36–R41 | State meanings, initial/later collection, waiver authority, finality and no overcollection | PC02 clarifies older UNPAID collection; PC08 resolved by S24; future reception initial/later receipts and admin-only waiver are explicit |
| R42–R50 | Cash/UPI split tender, one invoice per receipt, server timestamps/recorder, positive paise and immutability | Clear policy; schema, validation and locking remain technical work |
| R51–R61 | Invoice-level waiver, reasons/final metadata, derived balances, collection reporting and retries | Clear intent; PC14 keeps recovery/metadata engineering unselected |
| R62 | Package scope and parked activation/coverage design | Explicitly parked; PC04–PC05 clarify wording rather than settle design |
| R63–R73 | Excluded corrections, conditional future safeguards, schema audits, lifecycle parking and evidence limits | Clear exclusions; PC10–PC11 are bounded parked work |
| R74–R82 | Seven final review answers, current authority, final status names and joined clinical service names | Settled current answers retained; historical differences do not reopen them |

The six topic pages were also checked for cross-page contradictions, permissive wording that weakens a requirement, exceptions and authority. PC16 is a cross-page example that simple register/workbook equality does not catch.

## Mechanical verification completed

- All 82 current decision rows and 53 current issue dispositions agree with the tracker. Targeted source citations now include S24; the historical A–F attribution remains intact.
- Seven tabs, 136 decision entries, 16 historical table rows and 32 unexecuted acceptance cases are retained. Summary formulas independently produce 0 owner-review rows and 24 audit-required rows after N47 returns to technical audit.
- S24 updates 31 targeted cell values, adds one source row using adjacent formatting and raises Acceptance Checks row 18 to 62 points so the recovery case is readable. All original formulas, cell styles, panes, merges, existing column widths, validations and native-object counts are preserved; unrelated row heights remain unchanged. Affected views were rendered and visually checked.
- Every current Markdown link/anchor and tracker history-source path resolves. Linked hash-bearing source/manifest rows were checked against the final overlay; all 35 pre-existing source fingerprints matched before this update.
- The current-policy permission sweep finds no operative initial-checkout-only reception restriction or unanswered PC08 marker. Retained historical source text is evidence only.
- No application test, API call, migration, deployment verification or native Excel interaction was performed. The workbook cases remain explicitly unexecuted.

## Source map and coverage

[policy-clarity-coverage-2026-10-27.csv](policy-clarity-coverage-2026-10-27.csv) records all 36 decision-folder files, including added S23/S24 with reviewed-byte hashes, read status and remaining content work. The accompanying repository inventory accounts for 93 backend paths and 137 frontend paths before this audit's additions; the new S23/S24 paths are included separately. Application/schema content is outside the general policy-only review except for the owner-requested fee-association check: four sources were selectively read and one cross-reference file was keyword-searched, as recorded with hashes in the repository inventory. Personal planning and unrelated sources remain excluded; older materials do not establish current policy.

At this checkpoint: 31 decision-folder files fully read, 4 selectively consulted, no files merely inventoried, and 1 research document excluded from policy authority. The workbook current layer is inspected; its old attribution columns and historical schema cells are not described as an exhaustive current-policy review. Raw transcript consultation stays targeted to ambiguity; displaced short approvals are not given invented adjacency.

## Applied carry-forward wording corrections

Prepared and checked against post-S23 current files, then applied to the operative policy pages/register/tracker under the requested clarity pass. The latest checked live default remains main at f19793f683e06a289f1f50003d8588027aff6425. All 35 source fingerprints matched before the update. No original source history, application code or schema was changed.

| Findings | Exact target | Applied correction |
| --- | --- | --- |
| PC02 | payments line 10; R39; tracker H96 and Acceptance Checks B4 | Name older UNPAID and PARTIALLY_PAID invoices explicitly; partial receipts leave PARTIALLY_PAID and full balance collection ends PAID. S24 now permits future reception later receipts as well; only waiver remains admin-only. |
| PC04–PC05 | packages heading/line 7; R62; tracker H119 | Use MVP scope and current boundaries; distinguish a purchase charge from redemption while keeping activation, combined invoice grouping and same-checkout coverage parked. |
| PC07 | catalog after line 9; R10; tracker H67/I67 | State enabled-only new selection and independent backend availability validation; preserve historical entries and leave concurrent effective-time mechanics audited. |
| PC12 | checkout line 7; clinical line 19; R22/R23/R77; tracker H79/H80/H134 | Add that checkout's confirmed to the performed-care scope. Retain S23 fee-only support and the atomic issuance boundary. |
| PC16 | checkout line 11; clinical scenario; R25; tracker H82 | Replace may/can share with the existing same-checkout shared-invoice requirement. No one-invoice-per-day rule is added. |
| PC15 | register introductory status explanation; tracker H37 and Overview A23 | Remove stale current-status framing and through-S18 wording; call A01–A13 audit and parked workstreams. |

The earlier carry-forward correction changed six Markdown files and 18 workbook cells. S24 subsequently resolves PC08, updates 31 targeted cells and returns N47 from USER REVIEW to AUDIT REQUIRED because its business authority is settled. The final summary has no owner-review row and 24 technical-audit rows. Decision and acceptance IDs remain unchanged; no test is marked executed.

### Applied register wording

- **R10 (PC07):** Routine and standalone service identities come from services; no free-text service identity. Only admin may add services,edit existing master catalog prices or enable/disable services. Reception cannot change master prices or availability. Backend enforcement must accompany UI restrictions. For new entries, select only enabled services; backend independently rejects unavailable service IDs. Concurrent status changes remain audited.
- **R22 (PC12):** MVP issuance atomically saves that checkout's confirmed performed-care records when present, applicable extra-fee events, invoice and lines: all commit or all roll back. Fee-only checkout requires no fabricated clinical record. No clinical-save-first/bill-later path is approved. Settlement is separate after the invoice is returned.
- **R23 (PC12):** No clinical-save-first/bill-later MVP path is permitted. That checkout's confirmed performed-care records when present, applicable fees, invoice and lines commit atomically; fee-only issuance remains valid without dummy care. Possible future invoice-line work does not authorize partial clinical persistence.
- **R25 (PC16):** Complaint therapy, standalone care delivered with it, and an applicable examination/return fee appear as distinct lines on the same checkout invoice. One standalone line represents one actual performed occurrence; individual standalone sessions are not presold.
- **R39 (PC02, updated by S24):** Admin now, and the planned reception role when introduced, may record repeated positive receipts against UNPAID or PARTIALLY_PAID invoices with collectible balance, including after Pay Later and on later days. There is no initial-checkout-only, invoice-age or arbitrary repayment-count limit. Partial collection leaves PARTIALLY_PAID; full balance collection closes PAID. Reception need not hand those receipts to admin. Wallets and automatic cross-invoice allocation remain excluded.
- **R62 (PC04/PC05):** Packages remain required for MVP; detailed offering,purchase,activation,coverage,validity and redemption remain in another chat. Covered-only zero-total invoices are now explicitly allowed; detailed eligibility and activation are still unfinished. Package purchase is a distinct billed event from redemption; purchase/treatment invoice grouping and same-checkout coverage remain parked.
- **R77 (PC12):** No clinical-save-first/bill-later path in MVP: that checkout's confirmed performed-care records when present, applicable fees, invoice and lines commit atomically. Fee-only issuance needs no dummy clinical record. S12 Q3's earlier exception is a source-summary error. Payment/waiver settlement follows separately after the invoice is returned.

The full topic replacements appear in the corresponding findings above. These corrections restate already accepted behavior and clearly retain parked alternatives. They are applied to the generated current reading layer; none edits an original historical decision log or grants authority to implement application/schema changes.

### Owner decision and completion audit

No owner question remains in this clarity pass. The latest decision is attributed in [S24](../study/module_2/decisions/history/clinic_module2_reception_payment_authority_2026-10-29.md). Earlier source history remains unchanged.

| Objective requirement | Final evidence | Result |
| --- | --- | --- |
| Audit the operative decision layer | Six topic pages, README, all 82 R-rules, 53 N-issues and current tracker fields reviewed; grouped R-ID coverage retained above | Complete |
| Reflect latest accepted decisions | S17–S24 linked; examination fee semantics and 29 October collection authority incorporated in current pages/register/tracker | Complete |
| Remove materially ambiguous current wording | PC01/PC08/PC17 owner clarifications resolved; PC02/PC04/PC05/PC07/PC12/PC15/PC16 corrected; PC09 incorporated; other findings cleared or explicitly bounded | Complete |
| Keep one current rule per settled subject | Exact 82-rule and 53-issue tracker agreement; current permission restriction sweep; concrete fee/collection/waiver scenarios reviewed | Complete |
| Preserve history as evidence | All prior historical/research source hashes retained; only new attributed S23/S24 records added | Complete |
| Preserve and validate deliverables | Seven tabs and original IDs/formulas/styles retained, two intentional layout changes documented, 36 final coverage fingerprints, links/anchors and source paths verified | Complete |
| Keep deferred/technical work distinct | A01–A13 and package/lifecycle choices remain explicitly unfinished; no unselected alternative becomes operative and no runtime validation is claimed | Complete |

The current-policy clarity objective is complete. Full MVP design, application implementation, migrations, reception-role/RLS work and deployment validation remain separate unfinished work. Closing this clarity pass does not claim those outcomes.

## Source changes detected during this pass

The closing fingerprint check detected edits to reconciliation and tracker at 21:41 IST. They were reread; this audit did not write either source. The reconciliation changed N46 from “original four status names” to the explicit approved names. Reversing only that text replacement recreates the exact earlier file hash, confirming the Markdown change is limited to that wording. The workbook extraction found the corresponding Open Issues D49 wording update; all inspected current decision rows remain unchanged. The finding set is unaffected. Coverage hashes below now identify the reread versions; the other 32 source-file hashes remain unchanged.

| Source | Starting SHA-256 | Reread SHA-256 |
| --- | --- | --- |
| clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.xlsx | aaa9992eadcf2731c29731ae59940b5164e7211943b6b5065689254187a28814 | 777206cb8b49c3aca70a4c8fc62f48422c071c3e64bb0058111e159185a2384f |
| reconciliation-2026-10-27.md | e797546c481e0bb2ed7d895eb35ab54ea210d1873b71cf3bf2d5f890a24c68c2 | 57bed850939fb6d3361fd7a0a0eb3b5ee4e6a4a5e65d6eca87cc435ad5122c88 |

The table above records earlier concurrent drift, before this audit's own S23 promotion. The coverage CSV now pins the final S24-aligned files. Historical full-reading is not a completion gate. PC01/PC08/PC09/PC17 and the carry-forward wording corrections are complete; no owner-review issue remains in this clarity pass.

## Final checkpoint — 29 October 2026

S24's decision source, current payment/role-phasing text, register and tracker are aligned. Current live backend default main remains f19793f683e06a289f1f50003d8588027aff6425; reviewed local main is 92647cfacdb330f19ae39a27444c821954a8e87b, exactly 2 ahead and 0 behind. Targeted all-state PR and commit searches found no matching reception-policy work. This task performed no Git mutation.

The final source coverage fingerprints and agreement checks identify the completed documentation state. Native Excel and deployed enforcement were not tested; all acceptance cases remain proposed/unexecuted.
