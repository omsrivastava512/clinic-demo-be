# Clinic demonstrable demo — scope and delivery plan

**Prepared:** 16 November 2026. **Status:** inspection and planning only; implementation has not started under this plan.

This plan turns the existing clinic component catalogue into one coherent, authenticated, database-backed demonstration. It also preserves the larger clinic product scope. Finishing this plan does not mean the application is finished, any engineering proposal is approved, or the existing technical audit register is closed.

Om accepted the approach of substantial AI assistance with deliberate human ownership of a representative frontend workflow. The exact scope boundaries and engineering recommendations below are prepared for review before implementation. No code, SQL, deployment, Git history, PR status, or existing decision records are changed by this document set.

## Read this set in order

For delivery sessions, invoke `$delivery-learning-cycle` and resume [WORKFLOW_STATE.md](WORKFLOW_STATE.md). That record tracks the current slice, state, pending human contribution and actual evidence. The skill requires a human checkpoint between states; creating the workflow does not approve the implementation decisions below.

| Document | Purpose |
| --- | --- |
| [README](README.md) | Scope, success criteria, work sequence, ownership, and compact deferred backlog |
| [EVIDENCE](EVIDENCE.md) | Repository checkpoint, open-PR inspection, reusable work, old/new reconciliation, and inspection limits |
| [DECISIONS](DECISIONS.md) | Engineering choices, alternatives, recommendations, and decisions needed before dependent work |
| [TASKS](TASKS.md) | Granular implementation and learning checklist, with ownership, dependencies, and completion evidence |
| [ACCEPTANCE](ACCEPTANCE.md) | Concrete scenarios to verify the implemented result; all runtime checks initially unexecuted |

The checklist is detailed because each dependency, failure path, and handoff matters. Its size is not an estimate of feature count or evidence that everything fits in seven days. New findings must be recorded against the affected task and decision rather than silently expanding the project.

## 1. The result we are aiming for

An authorized demo admin can sign in, find or register a synthetic patient, select or create a catalog-linked complaint, record ordinary therapy, review the clinical entry, issue an invoice, record a full payment or choose Pay Later, and retrieve the same saved records after refresh or a later login.

The same workflow supports multiple treated complaints on one checkout. Each gets its own flat therapy line; an applicable patient-level examination/return fee appears once. Payment happens after invoice issuance. The application shows actual saved history and balances rather than fabricated metrics.

The engineering story should be demonstrable: frontend state transitions, API contracts, database ownership, atomic writes, immutable invoice history, and safe recovery from an uncertain response. Om should be able to trace, modify, and debug the representative frontend path he owns. This is not a requirement to memorize the entire application or independently recreate Supabase.

## 2. Proposed demo boundary

| Area | Included in the demonstration | Boundary |
| --- | --- | --- |
| Environment | One isolated synthetic-data environment; one operating clinic | No real patient records or clinic operational launch |
| Identity | Real sign-in/sign-out, session restoration, authorized admin membership, protected data access | No fake role switch; no public self-enrollment as admin |
| Patient | Search by supported name/phone inputs; register minimum agreed demographics; stable patient ID and MRN | No patient edit/delete/merge or old-record import in this slice |
| Complaint | Select existing active course; create a course from a seeded catalog identity; assign Routine/Rehab once | No uncataloged free-text course, automatic closure, or category conversion |
| Treatment | One or several complaint-visits; record enabled bundled services under the correct complaint | No separate charge for bundled procedures; standalone care deferred |
| Review | Explicit clinical review before commit; edit draft and cancel | No financial totals or automatic fees in pre-issuance preview, per current Q05 |
| Checkout | Backend-calculated flat prices and return fees; atomic care + fee + invoice + lines | No client price override; no care-save-first/bill-later path |
| Settlement | Full actual payment with one tender, Cash or UPI; Pay Later; later full settlement of that same invoice | Partial/split collections, waivers, and refunds deferred from demo only |
| History | Today's saved checkout activity, patient complaint/visit history, invoices, original billed amount, receipts and outstanding balance | No clinical/financial histories generated from mocks |
| Reliability | Loading/error/empty states, backend authorization, source integrity, transaction rollback, retry recovery, stale-response handling | No offline mutation queue or optimistic financial success |
| Delivery | Reproducible setup, controlled demo access, verified deployed flow, short walkthrough and factual project description | Public deployment/account changes require explicit execution authority |

Additional scope recommendations requiring confirmation in D01: seed catalogs instead of building catalog management UI; hide clinical-note entry in this demo rather than silently dropping entered notes; collect a real optional DOB instead of converting age to an invented date; use manual MRN unless existing evidence supports a different agreed mechanism; keep fee-only entry outside the demo UI while preserving independent fee-event modeling. These are demonstration cuts, not claims that those clinic requirements disappeared.

No package-bearing records should enter the demo flow. Do not silently charge ordinary therapy for a real patient whose valid package is unsupported. The isolated dataset and supported workflow must make that exclusion explicit.

## 3. Rules that the smaller demo must still respect

Current policy comes from the six Module 2 topic pages and their reconciliation register. See EVIDENCE for precedence and known conflicts.

1. One clinic and admin-only operation. A clinician in a clinical narrative does not create another login role.
2. Complaint classification belongs to the patient's course and remains immutable. Routine and Regular are synonyms.
3. Each treated complaint creates a visit and flat therapy line. Bundled procedures are clinical records, not separately charged items.
4. Clinical review precedes Create Invoice. Automatic fees and prices first appear on the returned issued invoice.
5. Checkout commits its care, applicable fee events, invoice, and lines together, or none of them.
6. Invoice issuance finishes before the separate payment operation begins. No database transaction waits while a person chooses payment.
7. First qualifying attendance costs ₹350; a return after 0–9 calendar days adds no return fee; 10–29 adds ₹150; 30+ adds ₹350. Use Asia/Kolkata calendar dates and patient-wide qualifying attendance, not last payment.
8. Issued line descriptions and prices are immutable snapshots. Billed totals derive from those lines, not today's service prices.
9. Payments are positive actual receipts, in integer paise, linked to one invoice. Cash and UPI are the supported tenders. Pay Later creates no zero-valued payment.
10. The full clinic model has UNPAID, PARTIALLY_PAID, PAID, and WAIVED. The proposed demo exposes only issuance and full-payment transitions; it does not rename or redefine the other states.
11. PAID and WAIVED are terminal. Saved payments cannot be edited/deleted. No generic invoice/visit deletion or correction workflow is included.
12. Debt does not prevent recording genuine subsequent therapy. No one-invoice-per-patient-per-day or one-visit-per-complaint-per-day shortcut is acceptable.
13. Server identity, prices, timestamps, ownership, and allowed transitions are authoritative. Client validation and disabled buttons are useful UX, not database enforcement.
14. Atomicity, replay protection, and preventing duplicate billing of the same occurrence are separate requirements.

## 4. Ownership: who does what

| Label used on every task | Responsibility | Required human involvement |
| --- | --- | --- |
| **AI** | AI drafts/implements within an approved scope, runs appropriate checks, and supplies a bounded diff and evidence | Om reviews the behavior and material changes at the phase gate; not line-by-line transcription |
| **OM** | Om performs the exercise, makes the product choice, or grants the specified authority | AI can explain, give hints, review, and troubleshoot; it does not silently complete an independent exercise |
| **PAIR** | AI proposes and explains; Om reasons through the rule or change; they inspect the evidence together | Om's specific decision or demonstrated understanding is recorded; passive agreement is insufficient for a learning checkpoint |

AI may write most implementation lines. Do not turn that into an invented percentage or a claim of Om's independent authorship. Work provenance is recorded per task: AI-drafted, Om-written, adapted from a PR, paired, or unreviewed. The engineer's completion evidence is the running result and tests; the learner's evidence is a trace, prediction, change, or diagnosis.

Om owns one bounded path: selecting an existing patient, carrying the saved ID and patient context into complaint selection, handling async loading/failure, and preventing stale data from replacing a newer selection. He then makes one small requirement change and diagnoses one introduced failure. AI carries setup, repetitive adapters, most database implementation, and additional integrations after the first pattern is understood.

## 5. Execution order and review gates

| Gate | Task phases | Evidence required to move on |
| --- | --- | --- |
| G0 — scope and contracts | P00–P02 | Current baseline; PR dispositions; demo exclusions; field/API contracts; blocking decisions resolved |
| G1 — trustworthy persistence | P03–P06 | Reproducible schema; correct admin authorization; catalogs; patient registration survives refresh; unauthorized requests denied |
| G2 — human-owned frontend path | P07–P09 | Om's patient-to-complaint path works; draft treatment review is coherent; no care saved early |
| G3 — complete persisted journey | P10–P13 | Atomic invoice issuance; separate payment/Pay Later; real history and balances retrieved |
| G4 — failure behavior | P14–P15 | Required acceptance scenarios pass, including lost responses, races, and direct API attempts |
| G5 — demonstrable and explainable | P16–P18 | Deployed synthetic demo verified; clear limitations; walkthrough; Om's explanation/change/debug exercises |

Dependencies are phase-specific in TASKS. Read-only research can proceed while a decision is open; dependent code must wait for that decision. A failure in a database invariant is not offset by a passing UI build.

### One-week experiment

Plan around the stated three focused hours per day across all work. Four or five hours are possible capacity, not yet demonstrated. A useful initial allocation is about 2½ hours on the demo and 30 minutes on job research/interview maintenance; an independent coding block sits inside demo time. This gives roughly 17½ delivery hours over seven days. It is a feasibility experiment, not a credible fixed estimate for every item in this checklist.

Target G0 on day 1, patient persistence by day 2, a draft workflow by day 3, invoice issuance by day 4, full saved journey by day 5, failure checks on day 6, and demo/explanation on day 7. Re-estimate after the first two days using actual time and unresolved decisions. If behind, present remaining work and alternatives; do not silently drop authorization, financial correctness, or recovery to meet a date. Om decides whether to extend the sprint or show an accurately labeled intermediate result. Job research continues; demo completion is not a prerequisite for every application.

## 6. Definition of demo completion

All applicable P00–P18 tasks have completion evidence, all required acceptance scenarios have recorded actual results against the same frontend/backend versions, and there is no unresolved failure in a supported path. Optional decisions have an explicit disposition. Om has seen the deployed result and can perform the learning exercises. The live walkthrough demonstrates a new record and later retrieval rather than screenshots or seeded-only interactions.

Keep three statuses distinct: **plan complete**, **implementation complete**, and **release verified**. None implies the next. Current application checks are NOT RUN under this plan.

## 7. Deferred work — compact whole-project backlog

“After demo” is not synonymous with “post-MVP.” Packages, full settlement, and several operational needs remain necessary for the real clinic product.

| ID | Workstream and necessary scope | Delivery classification / dependency |
| --- | --- | --- |
| F01 | Package catalog: four Routine/Rehab × 10/26 offerings; configured prices; block unpriced sales; purchase-time snapshots | Required clinic MVP, after demo; current package sources govern |
| F02 | Package sale/redemption: one combined checkout invoice; complaint-bound allowance; full entitlement regardless of payment/waiver; atomic redemption; last-unit races; no retroactive coverage | Required clinic MVP; A11 and checkout extension |
| F03 | Package lifecycle: start-date rules, daily cap default 1/admin-adjustable, short/long review rules, soft review, per-visit post-review confirmation, immutable notes, permanent closure with reasons, active-package complaint-closure block | Required clinic MVP; use packages.md for exact conditions; no freeze/reopen |
| F04 | 10→26 upgrade: terminal eligible parent, Day-30 window, +16 allowance, original anchor, current long price minus original billed short price floored at zero, one upgrade, preserved debt/waivers, zero-total settlement | Business behavior settled; MVP delivery conditional on bounded engineering effort; A11 |
| F05 | Partial and repeated collections; split Cash/UPI; terminal whole-residual waiver with reason; full four-state reporting and concurrency | Required clinic MVP, after demo; A07/A09 |
| F06 | Refund module and mistaken-package financial workflows, admin-only with mandatory reason; retain separate refund/new-payment records | Planned clinic scope under S31/S33; exact refund accounting design must be reconciled before building, not invented here |
| F07 | Standalone performed treatments with service identity and mandatory anatomy, no fake complaint, distinct charge on combined invoice | Required clinic workflow, after demo; A03 |
| F08 | First/recurrent fee-only checkout without dummy care; qualifying-attendance queries also count these events | Required clinic workflow, after demo UI; A01/A04 |
| F09 | Admin catalog additions, complaint-catalog append-only who/when/what trail, service prices and availability; immutable service names after launch; flat rate settings | Operational MVP setup/management; seed-only demo is not a replacement |
| F10 | Clinical notes contract and persistence; patient warning notes in relevant contexts; preserve notes/alerts distinction | Resolve exact operational fields; do not infer alerts/vitals from notes |
| F11 | Needs Attention: separate overdue-review package list and all positive invoice dues sorted by balance; includes small balances; no invented debt threshold | Required agreed operational behavior; after package/payment completion |
| F12 | Useful clinic reports: actual receipt-date collections, dues, patient/visit counts with defined units and timezone, invoice vs receipt distinctions | After reliable underlying records; metric list and release priority need owner review |
| F13 | Reception role, permissions, own allowed collections and package-debt restriction; admin-only waiver/refund/catalog authority | First planned post-MVP addition, not an extra demo role |
| F14 | Multi-clinic/branch pricing and ownership model; migration of explicit prices; multiple-admin model if requested | Deferred; no speculative branch matrix now |
| F15 | Historical paper transcription/import | Separately scoped/priced future module; no launch-day active-package or old-debt imports, no fictitious sales |
| F16 | Bulk settlement/concessions across many invoices, early-upgrade/renewal interpretation from second owner call | Unreconciled discussion, not approved implementation; clarify against current records first |
| F17 | Complaint inactivity/closure/reopening and draft deselection details beyond demo contract | Parked A05/A13; no automatic clinical behavior inferred from old issues |
| F18 | Generic clinical voids, invoice corrections/repricing, reversals and reopening | Deferred/restricted; would require new policy and integrity design, not a routine CRUD addition |
| F19 | Appointment booking/calendar, reminders, upcoming/waiting queue, no-show metrics | Future product work; old GitHub issues do not make it demo/MVP scope |
| F20 | Dashboard widget customization, charts, referral analytics, universal search, recent-patient shortcuts, expanded history search/tab and contextual entry actions | Selective reuse of PR #96 after accurate data semantics and route contracts; profile issues #91–#92 remain future UX |
| F21 | Print/export artifacts, patient-facing communications, regional languages | Future requirements; no automatic email/SMS/WhatsApp sending |
| F22 | Offline writes, optimistic mutations, realtime/SSE, multi-user conflict UX | Deferred; online failure recovery remains in demo |
| F23 | Legacy vitals/alerts/generic timeline tables and mock profile fields | N09–N11 dispositions after dependency/data inspection; do not silently drop tables |
| F24 | Real-clinic operational readiness: owner acceptance of complete workflows, access provisioning, backups/restore, monitoring, data handling, rollout/support process | Separate operational launch gate; synthetic demo acceptance does not establish readiness |
| F25 | Retail/inventory, wallets, cross-invoice transfers, patient-specific rate engine | Excluded from current MVP; require explicit future scope change, not assumed eventual commitments |

## 8. Maintaining this plan

Keep task IDs stable. Record the implementing commit(s), selected decision version, test evidence, and human review alongside a completed task. A source change invalidates affected task/check results until reviewed. Preserve the original record and add the new disposition. Do not auto-close GitHub issues, existing audits, or workboard cards based on this checklist.

Continue from the first incomplete dependency, not from an old chat summary. Refresh repository evidence before implementation. This document set is the scoped delivery plan; current Module 2 records remain the authority for clinic policy.

## 9. Planning-package verification

This completed planning package contains 268 unchecked execution tasks across 19 phases: 202 AI-owned tasks, 24 Om-owned tasks and 42 paired tasks. Those counts describe responsibility, not equal-sized work units, estimates of hours, or percentages of code authorship. There are 25 proposed engineering decisions, 130 acceptance scenarios, and 25 compact deferred workstreams.

The inspection covers all four returned open frontend PRs, the four returned merged PRs, and a disposition map for all 73 returned open issues. EVIDENCE records older/current conflicts and the limits of the inspection. Newer source dates, merged visual work, and deployment preview notices are not treated as proof of approved policy or working integration.

At the original scoping-task checkpoint, document checks verified unique/contiguous task and scenario IDs, phase/decision coverage, relative file links, and whitespace. A content-hash comparison confirmed all 187 pre-existing tracked/untracked files captured across the backend and portfolio repositories were unchanged; that task added this folder's five planning documents. Frontend working-tree cleanliness remained unverified because its Git scan failed, although the relevant source files were readable. No application code, existing decision records, migrations, deployment, PR status, or Git history was changed. No application/runtime acceptance scenario was run by that documentation task. The subsequent workflow setup adds WORKFLOW_STATE.md as a separate progress record; it does not mark implementation or learning tasks complete.
