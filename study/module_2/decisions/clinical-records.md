# Clinical records

Current clinical decisions as of 27 October 2026. Settled rules below are authoritative for the proposed MVP; pending audits remain explicitly marked. These decisions do not establish deployed schema. Evidence and current issue dispositions are in the [reconciliation register](reconciliation-2026-10-27.md).

## Current position

- **Locked:** one treated complaint on an occasion produces one complaint-specific visit. Two treated complaints during one physical attendance produce two visits and two therapy lines on the same checkout invoice (R04).
- **Locked:** ordinary therapy has a flat charge per complaint-visit. visit_services records the actual bundled procedures; those procedures do not have separate ordinary financial amounts, charge-status fields, or extra zero-value procedure lines. The ordinary billing_mode fork is dropped (R04–R05).
- **Locked:** the clinician manually assigns ROUTINE or REHAB when creating the patient's complaint. Store that classification on the complaint and use it for subsequent visits; do not infer it automatically from complaint_catalog or reselect it per attendance (R07).
- **Locked:** classification is permanent. A different classification requires a new complaint while preserving the original history. Closure of the original is **parked for audit**; no mandatory or automatic closure is approved (R08, R70).
- **Locked direction:** every patient complaint references complaint_catalog; no uncataloged free-text complaint identity. Derive complaint anatomy through that catalog in the discussed model. Narrative clinical notes remain distinct. Shared body-region storage and migration remain audited (R11, R68).
- **Locked:** admin can add a missing complaint-catalog entry before registering the patient's complaint, with an append-only who/when/what trail for the addition. General catalog versioning or a system-wide audit schema is not selected (R12).
- **Locked:** routine and standalone service identities come from services. An individually purchased traction/cupping treatment is a separate performed occurrence; the same service can be bundled in ordinary care in another context (R10, R26).
- **Locked:** a standalone occurrence does not require a fabricated complaint-visit and has no complaint_course_id link (R27).
- **Locked requirement:** every standalone treatment identifies its body region, such as cervical versus lumbar traction. A shared body_regions table versus another representation remains an architectural audit question (R28).
- **Locked:** one actual standalone performance produces one immutable timestamped occurrence and its own invoice line. Two actual performances remain distinct. Standalone care delivered with ordinary therapy shares the checkout invoice with separate lines. Individual standalone sessions are not presold; prepaid packages remain a separate subject (R25–R26).
- **Locked MVP scope:** no physical deletion or retroactive clinical correction/voiding workflow is approved. A financial waiver does not correct a false clinical record (R30, R63).

- **Locked:** that checkout's confirmed performed-care records when present, applicable fees, invoice and lines save in one atomic MVP issuance transaction. No clinical-save-first/bill-later path is permitted. Fee-only checkout requires no fabricated clinical record; settlement follows separately after the complete invoice is returned (R22–R24,R77; S23).

- **Locked / Q06 settled:** standalone clinical occurrences store the mandatory service_id and display the service name through a catalog join. No separate clinical treatment-name snapshot is required. Invoice lines retain their own frozen descriptions and amounts; common server recorder/timestamp conventions remain settled (R29,R82; S21).

## Implementation audits

- **A01–A03:** independent fee-event representation, cross-schema reference ownership/normalization, and anatomical catalog consistency remain open for audit. Examination/lapse fees are billing events, not clinical examination records, and require no complaint-specific visit_id or visit_services row. Keep actual standalone treatment occurrences separate; do not assert live schema from assistant reports (R19,R69; S23).
- Minimum standalone registration/documentation and exact field names, source links, authorization, and ownership checks remain implementation/audit work. A user-mentioned clinician is a workflow actor, not approval of new database roles.
- No generic physical-session table is approved. Distinct entry forms do not force distinct invoices or transactions.
- Complaint closure/inactivity (**A05**) and complaint deselection/procedure-cleanup validation (**A13**) remain explicitly parked.

See [all review questions and audit work](reconciliation-2026-10-27.md#questions-for-user-review).

## Future correction discussion and historical schema review

If voiding is developed later, the archived conditional discussion retains admin-only authority, individual-child invalidation, effective parent/child consistency, Paid-link blocking, and a collected-money floor. Triggers, metadata constraints, undo, and waiver-aware financial interactions remain unselected. These are future conditions, not MVP schema instructions (R64).

The earlier 16-table review records vitals deferred with drop-versus-dormant open, alerts removal intent pending dependency/data checks, and timeline_events purpose to investigate. No table has been removed or verified against deployment through this reconciliation (R65).

## Scenarios

Routine ultrasound during knee therapy is a clinical procedure beneath one flat-billed knee visit. Independently purchased traction is a separate performed event with anatomy. Treating knee and shoulder produces two complaint-visits; a separate cupping occurrence shares their invoice.

## Evidence

Current rules and source references: [reconciliation register](reconciliation-2026-10-27.md#decision-register). Original versions are in [history/](history/); their wording does not override the current rules above.

Related: [catalog and pricing](service-catalog-and-branch-pricing.md), [checkout](checkout-and-invoice-lines.md), [examination fees](examination-fees.md).

Current fee clarification: [owner's billing-only and fee-only-checkout clarification](history/clinic_module2_examination_fee_semantics_2026-10-27.md). The [seven-answer checkpoint](history/clinic_module2_seven_review_answers_2026-10-27.md) remains evidence for Q01–Q07; implementation and parked audits remain unfinished.
