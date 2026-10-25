# Examination fees

Current discussion view reviewed against the supplied October 25 scope/payment records. **O02–O04 remain open in detail; no new examination policy was settled there.** No examination schema or deployed behavior is established by this page.

## Current position

- **Established owner direction:** examination recurrence after 30 days without a completed visit, across the clinic chain using the same patient identity. Anchor on the most recent completed visit, not last paid invoice or last examination.
- Payment status must not erase clinical attendance. Return examination is not patient re-registration.
- **Superseded:** a once-per-patient-lifetime examination model and lifetime `UNIQUE(patient_id)` assessment restriction.
- **Provisional:** charging on day 31 rather than day 30; patient-wide across any complaint is the recorded interpretation requiring owner confirmation of details.

## Open questions

- **O02:** examination occurrence under a visit procedure versus a dedicated repeatable event; patient-level examination spanning complaints or without ordinary treatment. Do not arbitrarily attach it to one complaint just to fit a required visit FK.
- **O03:** calendar days versus exact elapsed timestamps, time zone, definition of completed visit, cancellations/no-shows, same-day repeats, package attendance, retroactive entries and historical imports.
- **O04:** first examination, exemptions/overrides, fee amount and whether the old separate gap fee survives.
- ₹350 was a previously discussed examination price, not reconfirmed under the revised recurrence rule. The older ₹150 gap-return fee must not be equated with or added to the recurring examination without clarification.
- Eligibility under concurrent branch checkouts, one charge per qualifying attendance, patient/branch/invoice consistency and source shape require technical audit.

## Reasoning and scenarios

Multiple complaints can produce multiple visits in one attendance; that does not establish multiple examination charges. A completed unpaid or covered visit may matter for recency. The treatment-free examination case is hypothetical, not confirmed operational practice.

## Changes in position and sources

- [October 22 earlier log](history/clinic-billing-decision-log-2026-10-22.md), D09, preserves earlier examination/gap-fee discussion.
- [Parent register](history/clinic_billing_decision_log_2026-10-23.md), §4, supersedes the lifetime-only design with recurrence; §§8–9 and 13 preserve concurrency scenarios, O02–O04 and the standalone occurrence gap.
- [Successor](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md), §12, does not settle examination details. Standalone treatment design must not silently choose the examination model.
- [October 25 scope continuation](history/clinic_module2_scope_decision_addendum_2026-10-25.md), §4, explicitly leaves examination modeling in its independent issue register. Its payment same-day rule does not decide examination eligibility, clinical recency or historical-import semantics. The blanket no-retroactive-corrections MVP boundary does not resolve these occurrence/eligibility questions.

Related: [clinical occurrences](clinical-records.md), [invoice source audit](checkout-and-invoice-lines.md).
