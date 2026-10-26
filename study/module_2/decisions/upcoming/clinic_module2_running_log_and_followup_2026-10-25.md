# Clinic Module 2 — Running Decision Log + Follow-up

**Date:** 25 October 2026 (IST)  
**Baseline:** The running code block supplied in chat after `clinic_module2_waiver_catalog_addendum_2026-10-25.md`.  
**Scope:** Full running-log content, followed by newly settled matters and status changes from subsequent discussion. An independent file; no predecessors overwritten.

**Legend:** LOCKED = approved MVP behavior; OPEN = undecided; IMPLEMENTATION OPEN = behavior approved, schema/security mechanics not settled; FUTURE SCOPE = not MVP; SUPERSEDED = previous status changed by later explicit decisions.

---

## Part I — Running log baseline (as of the earlier code block)

### 1. Examination-only visits — LOCKED

Patients can attend solely for a formal examination, with no therapy on that day, and be billed for the examination. Example: examination and payment on Monday, therapy beginning Tuesday. No dummy therapy record should be created to record an examination. **Implementation open:** examination occurrence identity, relationship to the patient/clinical visits/complaints and invoice line source.

### 2. Examination and therapy in one checkout — LOCKED

Formal examination and therapy can occur together in one checkout with both appropriate charges, or on different days with independent clinical records and invoices.

### 3. Examination charge per appointment — LOCKED

One formal examination fee per clinical attendance, irrespective of how many complaints are examined in that sitting. Ordinary therapy pricing is separate and may be complaint-visit based. ₹350 has been used for the formal examination, subject to the later fee clarification in Part II.

### 4. Thirty-day formal examination rule — LOCKED

The patient-wide rule uses time since the **most recent clinical visit**, not time since the previous formal examination. During continuing therapy the clinician can perform informal check-ins without generating a new formal exam or fee. After a gap of **30 days or more**, a formal examination is mandatory before therapy resumes; both eligibility and billing must be enforced by the backend. Example: formal exam August 1, therapy August 20 → gap clock starts August 20. The previously OPEN choice of calendar days vs exact timestamps is settled in Part II.

### 5. Examination-only visits reset gap clock — LOCKED

A completed examination-only clinical visit restarts the inactivity clock, just like a qualifying therapy visit; examination followed by next-day therapy is a continuing course. The last qualifying clinical visit may involve an examination only, therapy only, or both.

### 6. New complaint during ongoing therapy — LOCKED

Creating a new complaint while a therapy course is ongoing may include an informal clinical assessment and complaint record, but **does not** automatically register or bill another formal examination. Ordinary informal check-ins are not standalone formal examination records. The 30-day rule remains applicable.

### 7. Formal examination on new complaint — FUTURE SCOPE ONLY

If the clinic's clinical requirements change, creating a new complaint **may eventually** trigger a mandatory formal examination and corresponding billing. This is **not approved for MVP**. Revisit the trigger, exemptions, relationship to 30-day gap, same-attendance duplicate exams and charges only when the policy is requested.

### 8. Examination data-model consequences — IMPLEMENTATION OPEN

The eventual design must support exam-only visits; exams coinciding with several complaint visits; a single exam charge per attendance; patient-wide examination history and inactivity checking; distinct formal vs informal assessment; and valid typed billing sources. Do not assume examination belongs in `visit_services` or fabricate a complaint-specific therapy visit to satisfy a foreign key. Exact table layout, source FKs and consistency requirements remain open for engineering review.

### 9. Open questions as listed in the earlier running log (historical checkpoint)

- Q1: Calendar-date vs timestamp threshold — **CLOSED in Part II**.
- Q2: Storage/source entity for formal examination occurrences — **IMPLEMENTATION OPEN**.
- Q3: Qualifying completed clinical visit definition — routine completed clinical entries are the accepted trigger; **do not introduce an appointment module**. Mechanics remain implementation review only if necessary.
- Q4: Prevent duplicate examination charges across multiple complaints in a checkout — **IMPLEMENTATION OPEN**.
- Q5: ₹350 examination fee versus old ₹150 gap-return fee — **CLOSED in Part II**.

### 10. Earlier scope and supersession

Packages are in MVP but detailed in another chat. All retroactive invoice and clinical correction workflows remain deferred beyond MVP. Payment/waiver behavior, issued-charge immutability, and single-clinic catalog pricing decisions are unaffected. The agreed examination recurrence is based on the last qualifying clinical visit, superseding any last-exam-date-only interpretation. No source code or database migration was approved by the running log.

---

## Part II — Decisions and agenda corrections AFTER the running code block

### A. Calendar-date calculation — LOCKED

- **Use calendar dates**, not an exact elapsed-hour/timestamp clock, to calculate the 10-day and 30-day inactivity boundaries.
- Timestamps can still be stored for records/audit; they are not the eligibility cutoff mechanism. Implement using appropriate clinic-local dates.
- Examination-only and therapy visits both reset the clock, as already locked.
- **Scope correction:** The assistant's proposed question about appointment bookings/cancelations was irrelevant: there is **no appointment module in MVP**. No appointment scheduling workflow, booking-based resets, or cancellation policy should be introduced from this conversation.

### B. ₹150 / ₹350 return policy — LOCKED; original O04 ambiguity CLOSED

The user explicitly clarified that these fees represent **different gap intervals**, not competing price guesses or charges to stack:

| Calendar-day inactivity since most recent clinical visit | Return/examination fee | Rule |
| --- | ---: | --- |
| **0–9 days** | **No gap-based fee** | Ordinary therapy charges, if any, remain separate. |
| **10–29 days** | **₹150** | Apply the return-after-gap fee; no automatic ₹350 formal re-exam solely for this interval. |
| **30+ days** | **₹350** | Require and bill the formal examination before therapy resumes; do not add ₹150 on top. |

This closes the previous apparent conflict between an older ₹150 return-gap charge and a ₹350 examination. A backend fee calculator must evaluate recency from the latest qualifying clinical visit. Invoice-line classification/technical storage for the ₹150 fee remains a design task, not a fee-policy disagreement.

### C. O05 billing mode — REMOVE FROM THIS CHAT'S AGENDA

The user expressly said O05 is **settled elsewhere**, is **not this assistant's job**, and must be **ignored/omitted** from further questions. The exact settled design was **not shared here**, so this document deliberately does not manufacture an invoice/visit storage choice. This is an **agenda boundary**, not evidence for any particular schema choice.

### D. O06 catalog/procedure eligibility — PARTIALLY SETTLED, STILL OPEN

Earlier Module 2 discussions had already addressed significant behavior:

- Routine performed procedures can be recorded clinically without an extra independent charge when covered by the agreed flat-rate ordinary therapy rules.
- Separately billable specialty treatments are distinguished from routine included procedures.
- The treatment procedure picker is intended to record procedures performed, rather than administrative charges.
- Single-clinic MVP: service prices and enabled/disabled status live in `services`; defer `clinic_service_prices` and multi-clinic pricing/availability overlays. A later deliberate migration must integrate/rewire to `clinic_services` **depending on its eventually chosen role**, which is **not fixed here**.

**Genuinely open within O06:** final physical catalog architecture (unified `services` definition vs more specialized definitions) and precise classification/filter eligibility in the current catalog, where not already decided elsewhere. Its relationship to O11's standalone specialty clinical occurrences is also unresolved. **Do not mark O06 fully closed**, and do not restart already settled business behavior. The assistant's earlier jump to O07 as though O06 were finished was corrected.

### E. O07 package-covered invoice-line question — NO NEW DECISION

After the running block, the assistant proposed choosing between a ₹0 package-covered invoice line and omission, but the user gave **no approval for either option**. Preserve any pre-existing package discussion decisions from the separate package chat; no new O07 rule is authorized by this exchange.

### F. Other unresolved items and non-goals — unchanged

- O02 occurrence/source schema and duplicate fee prevention: engineering modeling remains.
- O11 independent specialty-treatment occurrence, scope and invoice linkage: still open unless subsequently resolved in its own discussion.
- O12 therapy price tier/negotiated-rate ownership: open unless already resolved elsewhere.
- Exact invoice schema, RPC, transaction boundaries, authorization/RLS, retry/idempotency, concurrency safeguards, legacy migrations and tests: technical implementation/audit tasks, **not evidence of missing clinic-policy approval**.
- Retail, multi-clinic pricing, and retroactive corrections stay deferred; packages stay MVP and are handled in a dedicated discussion.

## Current checkpoint

**Closed since the earlier running block:** calendar-day threshold; distinct ₹150 (10–29-day gap) and ₹350 (30+-day mandatory exam) fee policy; removal of O05 from this chat's agenda.

**Clarified, not closed:** O06 is partially decided, but final MVP catalog structure/classification needs deliberate review if not already answered elsewhere.

**Not newly decided:** the O07 ₹0 vs excluded package-charge proposal; standalone specialty occurrence and other engineering architecture.

**Discussion procedure:** Cross-check the historical agenda and later Module 2 decisions before proposing new business questions. Ask one real unresolved question at a time; do not reintroduce scheduling or deferred features. The user decides when to update written running logs.

---

*Documentation only. No previously created document, repository, or database was altered by this addendum.*
