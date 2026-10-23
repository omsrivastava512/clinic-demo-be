# Clinic billing decision addendum

Date: 2026-10-21 (Asia/Kolkata)
Source: the supplied conversation. This addendum does not replace or reconstruct the earlier full decision log, which was not located.

## Settled: unified billable-item catalog for MVP

- Use one small catalog for chargeable offerings, with stable identifiers/codes, names, and explicit kinds (for example, fee or procedure).
- Store clinic-specific prices in a related pricing table rather than separate fee amount fields on the clinic.
- Keep billing policy, including flat versus per-procedure mode, in clinic settings.
- Backend billing logic evaluates policy, treatment, and relevant patient history to choose applicable charges. Configurable parameters belong in database settings; the exact rule representation remains to be designed.
- Invoice lines preserve actual historical billed amounts. Catalog prices describe current rates.
- Procedure selection must expose clinical procedures, not automatic fee entries.
- Packages and equipment can use the shared catalog when their modules are implemented. This decision does not bring those deferred modules into MVP scope. Package coverage and equipment inventory need their own operational records.

Rationale: consistent offering identities and clinic pricing across fees, procedures, and later packages/equipment. Accepted trade-off: explicit classification, workflow filtering, and validation are necessary. New supported offerings may be data changes, but new billing behavior can still require code.

## Other explicit settled decision from this conversation

- Remove is_charged from the proposed visit_services design; reliable invoice-line links identify separately billed procedures. A missing separate charge does not by itself establish inclusion, waiver, or package coverage.

## Confirmed offering context and tentative prices

- Routine pain therapy: INR 200.
- Rehabilitation therapy, including accident recovery and post-op: INR 300.
- Four package offerings: routine 10-day, routine 30-day, rehab 10-day, rehab 30-day.
- Tentative 30-day prices: routine INR 4,500; rehab INR 6,000. Both 10-day prices unknown; do not infer by proration.
- Separately priced procedures and specialty treatments; initial examination fee; gap-return fee discussed with applicability unresolved.
- Equipment such as belts is actually sold by the clinic, but is outside MVP scope.

## Remains open

- Whether an applied billing-mode historical snapshot is needed, and whether it belongs on visits or invoices.
- Exact invoice-to-visit and invoice-line-to-performed-procedure relationships and integrity constraints.
- Exact flat-fee procedure inclusions and the simplest configuration needed to express them; no inclusion mapping schema is settled.
- Package days: calendar validity versus attended treatment days; coverage and expiry rules.
- Whether package-covered treatment appears on invoices, and any package valuation or allocation model.
- Attribution of checkout-level charges such as examination to individual visits.
- Exact catalog kinds, stable codes, price-selection precedence, and rule configuration schema.

## Later audit requested

After the remaining discussions, audit the combined design for consistency, duplicated financial authority, historical price preservation, source links, referential integrity, duplicate billing prevention, rule applicability, and MVP scope. Record contradictions and unresolved questions explicitly. This audit has not yet been performed.

Implementation status: design decisions only; no application code, migration, or database changes made.
