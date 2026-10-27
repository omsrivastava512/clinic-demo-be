# Clinical preview and automatic-fee visibility — 27 October 2026

Source: the user's answer to Q05 and request for discussion of Q06 in the consolidation chat.

## Q05 — Resolved

Before Create Invoice, reception focuses on clinical accuracy and what was done today. The preview is clinical; do not show automatic fee amounts or an invoice-price/total preview there. Existing last-visit context can remain visible, without adding a fee preview or approval step.

On Create Invoice, the backend checks fee eligibility and calculates applicable fees inside the atomic clinical/billing issuance transaction. Once issuance succeeds and the complete invoice is returned, reception sees the itemized treatment charges, automatic fees and total, then proceeds to the separately committed settlement stage. This display decision does not move fee calculation after issuance or permit partial clinical persistence.

This resolves the pre-issuance monetary-preview ambiguity in favor of clinical review followed by the issued invoice. Any earlier requirement to show automatically applied fees, invoice-line prices or the invoice total in that clinical preview is superseded. Scope: documented UI behavior only; no implementation change occurs here.

## Q06 — Still under discussion

The user asks why a standalone clinical occurrence needs its own copied service name rather than storing service_id and displaying the name through a join. They request the assistant's recommendation. No clinical name-snapshot decision is approved by this request. The existing immutable invoice-line description remains settled and separate.

## Q07 — Still unanswered

The user will answer catalog price/availability editing authority after discussion of Q06. Admin-only additions remain settled; do not infer editing authority.

Current checkpoint: Q01–Q05 settled; Q06–Q07 open. Existing source records remain unchanged. No application code, database schema or Git history is changed.
