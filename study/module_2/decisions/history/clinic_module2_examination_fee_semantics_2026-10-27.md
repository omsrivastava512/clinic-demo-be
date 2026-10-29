# Examination and lapse fee meaning — owner clarification

Recorded 27 October 2026 from the owner's subsequent clarification in this conversation. This is a concise attributed record, not a verbatim transcript. Existing historical sources remain unchanged. The current topic pages and reconciliation register govern operative policy.

## Accepted meaning

- “Examination fee” and “late/lapse fee” describe billing charges. A fee event is a soft attendance/billing record, not proof that a clinical examination happened or how thorough it was. Ongoing physiotherapy assessment does not itself trigger such a fee.
- The applicable fee is independently recordable at patient/checkout level. It does not require a complaint-specific visit_id, a visit_services row or a fabricated clinical examination record. The existing extra_fees direction remains separate from actual standalone-treatment occurrences; no new physical schema is selected.
- First qualifying attendance attracts ₹350. Return gaps are calendar days since the most recent qualifying completed attendance: 0–9 days means no examination/lapse fee event, 10–29 means ₹150, and 30+ means ₹350 without an additional ₹150. Five days is within the no-fee band.
- An applicable examination fee can be checked out without therapy. This supports an initial consultation/fee-only attendance followed by therapy on another day, and the same fee-only workflow on a qualifying recurrent return. Both follow the same rules.
- Therapy-only and fee-only completed attendance reset recency. Payment timing does not. A fee-only attendance followed by therapy the next day does not produce another return-gap fee.
- One physical attendance must not multiply the fee across treated complaints. Backend calculation remains automatic during atomic issuance. Performed-care records, when present, save with fees, invoice and lines; fee-only checkout needs no dummy care. Settlement follows after the issued invoice is returned.
- Fees first appear on the returned issued invoice under Q05. This clarification does not add a zero-total exception, override UI, clinical-first persistence path or permission to alter issued records.

## Requested consistency check

Check current decision wording and documented repository models for fees embedded in complaint visits or requiring visit_id/visit_services. Older history may preserve gaps or competing models. Preserve those originals; do not treat their wording as current policy or as evidence of deployed enforcement.

Exact fee-event schema, ownership/source constraints, qualifying-record mapping, calendar handling, concurrency and migration remain implementation audit work.
