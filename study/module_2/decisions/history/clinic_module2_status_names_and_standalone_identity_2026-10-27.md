# Invoice status names and standalone clinical identity — 27 October 2026

Source: the user's latest answer in the consolidation chat after Q07 was settled.

## Q02 — Explicitly approved terminology change

The user selects the suggested set: UNPAID, PARTIALLY_PAID, PAID, WAIVED. The spoken waiver/waived is a correction to WAIVED, not an additional fifth state. This is an explicit approved rename of the earlier PENDING and DUE terminology, superseding the previous decision to retain those names.

UNPAID is an issued invoice with no receipt; PARTIALLY_PAID has a positive receipt and remaining collectible balance. PAID and WAIVED retain their settled meanings and terminal behavior. Package-covered-only zero invoices still offer Mark as paid after issuance, with no payment row and zero collections. Business behavior is unchanged by the rename. No database enum or application code is migrated in this documentation task.

## Q06 — Service identity through the catalog

In the context of the service_id-only recommendation, the user rejects hypothetical catalog renaming or repurposing as grounds for copying a clinical service name. Record the standalone clinical occurrence using the mandatory service_id and display the catalog name through a join; do not add a separate treatment-name snapshot to that clinical table. Immutable invoice-line descriptions and amounts remain required and are separate from clinical storage. Existing server recorder/timestamp conventions remain settled.

The user mentions aliases for alternative wording, preferring an aliases field rather than a separate alias table. This is catalog metadata direction; exact representation, datatype and implementation are not selected here. It does not require a clinical name snapshot or reopen the hypothetical repurposing discussion.

## Current checkpoint

All seven review questions Q01–Q07 are settled. The earlier versions remain historical evidence and cannot override current rules. Dedicated package design, exact DDL, reference ownership/anatomy, examination representation, authorization implementation, idempotency, migration and validation remain unfinished as already recorded in A01–A13. Closing these review questions does not claim implementation completion.

Existing archived sources and research remain unchanged. Only current documentation and the tracker are updated; no Git mutation, application code or database schema change is performed.
