# Current decision authority — 27 October 2026

Source: the user's follow-up in the consolidation chat after the Q01–Q04 corrections.

## User instructions

- Historical logs may contain different versions. Current decision files must give a consistent current rule, without competing approved alternatives.
- Check this across the decisions folder, including automatic fees and atomic clinical/billing issuance.
- Confirm that the covered-only zero-total invoice decision is recorded in files, rather than remaining only in chat.
- Retain the original four invoice statuses PENDING, DUE, PAID and WAIVED. The user is open to considering a recommendation, but has not approved another set.
- Q05–Q07 will be answered afterward; do not infer answers or close those items.

## Current meaning

The six topic pages and current reconciliation register state the operative decisions. Pending or parked questions are explicitly marked and do not approve either alternative. Source inventories, comparison tables, tracker historical columns and the historical schema inventory are evidence only; they cannot override current decisions. Research is background rather than clinic policy.

Settled rules remain: covered-only zero invoices are allowed and offer only Mark as paid after issuance, with no payment row or collections; automatic fees are calculated by the backend on Create Invoice; clinical records, applicable fees, invoice and lines commit atomically, with payment/waiver afterward. Original PENDING/DUE/PAID/WAIVED names are retained.

## Consistency corrections in this pass

- Qualify no-receipt Pay Later behavior as applying to positive-total invoices; preserve the covered-only zero exception everywhere.
- State current rules directly in topic pages, with chronology confined to clearly labeled evidence references.
- Replace the root reconciliation's repeated historical issue-status table with current dispositions. Keep the source logs unchanged.
- Replace tracker current-column pointers to old wording, including an obsolete card-payment allowance, with explicit current interpretations. A–F historical decision attribution and historical schema observations remain preserved and clearly marked as evidence.
- Record explicit retention of the original status names. No alternative naming proposal is accepted or applied.

No application code, database schema or Git history is changed. Existing historical sources and research content remain unchanged. This consistency review does not complete the parked technical audits.
