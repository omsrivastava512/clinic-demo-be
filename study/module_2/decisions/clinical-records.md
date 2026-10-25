# Clinical records

Current discussion view incorporating the supplied October 25 records and scope continuation. Design decisions do not establish deployed behavior or authorize implementation.

## Current position

- **Settled:** one treated complaint on an occasion produces one `visit`. Treating two complaints produces two complaint-visits, even during one physical attendance.
- **Settled:** `visit_services` records each procedure actually performed within a complaint-visit, including routine procedures bundled in flat therapy. These are clinical occurrences, distinct from catalog definitions and issued charges.
- **Settled in the proposed model:** ordinary complaint therapy is flat-fee. Its performed procedures have no separate financial amount or charge-status fields; one therapy invoice line comes from the visit. See [checkout](checkout-and-invoice-lines.md).
- **Confirmed real case:** someone may receive individually purchased traction without ordinary complaint-course care. Cupping can also be separately charged, including alongside ordinary therapy.
- **Settled conceptual cardinality:** each independently performed standalone treatment has its own occurrence and invoice line. Two actual performances remain distinct, even for the same procedure/body region. Individual standalone sessions are not presold; prepaid packages remain separate.
- **Settled MVP scope, October 25:** no retroactive clinical correction/voiding workflow. The earlier open→optional voiding scope was subsequently deferred outside MVP; do not pre-add unused void columns. A financial waiver does not correct a false clinical record.

## Open questions

- **O11:** final standalone-occurrence table/name, required clinical fields, optional complaint association, and precise V1 boundary. Do not fabricate a complaint or visit to satisfy an FK.
- Minimum registration and clinical documentation for standalone care; clinical authorization, referral, treatment site and safety information remain to define from actual practice.
- Combined versus separate entry flows and invoices remain open. Distinct forms do not imply distinct transactions.
- No generic physical-session table is approved. Clinical entry is usually by reception, but clinicians may enter records; full checkout permissions are not specified.

## Future correction discussion and historical schema review

If voiding is developed later, admin-only authority, independent child-procedure void, effective parent-visit cascade, no Paid-linked void and a collected-money floor were conditionally agreed. The four discussed void fields, cascade mechanism, metadata constraints and waiver interaction are not MVP schema instructions. The audit identifies indirect invoice links and false records trapped behind Paid lock as unresolved risks.

The historical 16-table review records vitals deferred with drop-versus-dormant open, alerts removal intent pending dependency/data checks, and timeline-events purpose to investigate. These are documented dispositions, not tables removed or a verified deployed inventory. Complaint-course/note correction remains future design. Complaint deselection/procedure-cleanup validation was parked, not approved as a new interface requirement.

## Reasoning and scenarios

Routine ultrasound during knee therapy remains a clinical procedure beneath a flat-billed knee visit. Independently purchased traction is a separate performed event; the catalog identity alone does not determine billing context. Two complaint-visits and an extra cupping occurrence can occur in one attendance without proving a shared session entity is needed.

## Changes in position and sources

- [Parent register](history/clinic_billing_decision_log_2026-10-23.md), §§1–2 and 13: clinical meanings and earlier flat/per-procedure policy.
- [Successor addendum](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md), §§2–5: vocabulary, standalone cases S1–S7, candidate fields and workflows. These alternatives are retained, not all approved.
- Successor §§11.1–11.2, 11.5 and 12 supersede the ordinary billing-mode fork and chargeable routine-procedure source. Final occurrence schema remains open.
- [October 25 A1](history/clinic_module2_checkout_payments_voiding_addendum_2026-10-25_164648_IST.md), §§4–6: field discussion, historical table audit and validity/reporting dependencies. [A2](history/clinic_module2_successor_addendum_2026-10-25_171327_IST.md), §§1–2, 6: optional scope and conditional safeguards.
- [A3 audit](history/clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md), G03, G06–G15: source, financial and clinical-history conflicts. [Scope continuation](history/clinic_module2_scope_decision_addendum_2026-10-25.md), §§1–2, 7, supersedes optional MVP voiding with future deferral while retaining the conditional reasoning.

Related: [catalog](service-catalog-and-branch-pricing.md), [examinations](examination-fees.md), [packages](packages.md).
