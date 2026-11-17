# Demonstrable clinic demo — execution checklist

**Status: planning only. Every implementation task below starts unchecked.** The repository inspection in EVIDENCE is completed research, not evidence that these future implementation tasks pass. This document does not authorize code changes, dependency changes, database mutations, deployment, or Git operations.

Read [scope and ownership](README.md), [source/PR reconciliation](EVIDENCE.md), and [decisions](DECISIONS.md) first. The checklist assumes the recommendations in DECISIONS are selected. Where an alternative is chosen, revise the dependent tasks and scenarios before implementation; do not implement competing paths.

## How to execute and record progress

- **AI:** AI performs the bounded implementation/check and supplies evidence. Om reviews the result and material diff at the phase gate.
- **OM:** Om makes the specified decision or performs the exercise. AI can explain, hint, review, and troubleshoot; independent work must not be silently completed for him.
- **PAIR:** AI and Om inspect a critical rule, contract, trade-off, or result together. Record Om's decision or demonstrated understanding.
- Each task's second sentence describes its completion evidence. A checked box without that evidence is not complete.
- Record `status; implementer/provenance; selected decision; affected files; version/commit; actual check result; evidence location; human review; remaining limitation` beside the task or in a linked execution log. Git commits are recorded only if separately authorized and created.
- Use `not started`, `in progress`, `blocked with reason`, `ready for review`, `done`, or `deferred by explicit scope decision`. Deferral is not a passing result. A supported requirement cannot be waived merely to finish the week.
- Check scenarios in [ACCEPTANCE](ACCEPTANCE.md) against the same known frontend, schema, fixture, and API versions. Re-run affected checks after a relevant change, not the entire suite after every prose edit.
- No automatic merges, cherry-picks, issue closure, workboard changes, or audit closure. Reviewable patch proposals and local verification are distinct from those actions.

## P00 — Establish the implementation baseline and reuse plan

**Depends on:** implementation being requested. **Decisions:** D02, D22, D25. **Gate:** G0. **Evidence:** a refreshed version/source/reuse record; this is necessary again when implementation starts.

- [ ] **P00.01 · AI** — Read applicable repository instructions and the relevant current decision records. Record the files used and distinguish current policy, implementation evidence, proposals, and old material.
- [ ] **P00.02 · AI** — Inspect branch, HEAD, recent commits, staged/unstaged changes, and untracked contents in backend and frontend; refresh portfolio evidence when career claims are involved. Preserve a baseline of pre-existing work and explicitly report any unavailable scan.
- [ ] **P00.03 · AI** — Verify each repository's authoritative origin and live default-branch SHA read-only. Calculate exact divergence only when the necessary objects are available; report missing objects without an unapproved fetch.
- [ ] **P00.04 · AI** — Recheck open/closed PRs, relevant direct commits, and equivalent integration work since this inspection. Record newly completed, superseded, duplicated, or changed work before planning a patch.
- [ ] **P00.05 · AI** — Refresh PR #95's scrollbar candidate and obsolete billing changes separately. List exact candidate files/hunks and dependencies; do not adopt its dynamic historical pricing behavior.
- [ ] **P00.06 · AI** — Refresh PR #96's draft prerequisite and main-branch divergence. Identify reusable widget shells separately from fabricated metrics, dead routes, obsolete types, and unrelated shared-component changes.
- [ ] **P00.07 · AI** — Refresh PR #97's validation/error UI candidates. Record the old demographic/payment contracts, discarded transformed output, and early form reset that must not survive an adaptation.
- [ ] **P00.08 · AI** — Refresh PR #98's visual baseline and small interaction changes. Compare its current head with main and report whether the preview is accepted; an open PR or optimistic PR description is not acceptance.
- [ ] **P00.09 · PAIR** — Choose the approved visual baseline and selective reuse disposition. Record what is reused, adapted, deferred, or rejected, and why; pending work remains pending rather than silently merged.
- [ ] **P00.10 · AI** — Map overlapping PR files and semantic conflicts against the intended patch. Produce an order of adaptation that avoids importing multiple competing patient, visit, invoice, and payment models.
- [ ] **P00.11 · AI** — Inspect installed packages, lockfile, scripts, environment-variable names, and existing tests. Record actual versions and usable native features before proposing dependencies or custom workarounds; omit secret values.
- [ ] **P00.12 · AI** — Establish the current typecheck/build/test baseline with the existing supported commands. Separate pre-existing failures, unavailable tooling, and failures introduced later.
- [ ] **P00.13 · PAIR** — Agree the first bounded implementation slice and file scope. Record the behavior, accepted components, proposed departures, and required approvals before any nontrivial code change.
- [ ] **P00.14 · OM** — Choose a short daily work block and start a real time log. Record focused time and blockers for the first two days; do not assume six productive hours are available.

## P01 — Freeze the demonstration and source authority

**Depends on:** P00. **Decisions:** D01, D06–D09, D17, D23–D25. **Gate:** G0. **Evidence:** approved demo boundary and reconciled blocking questions.

- [ ] **P01.01 · PAIR** — Walk through the exact demonstration from sign-in to later retrieval. Record supported actors, entry points, saved records, and the final observable result.
- [ ] **P01.02 · OM** — Confirm synthetic data, one clinic, and provisioned admin access for this slice. Decide whether the first demonstration is guided or separately accessible to reviewers.
- [ ] **P01.03 · OM** — Confirm the proposed cuts: seeded catalogs, no package-bearing cases, no note-entry UI, no standalone/fee-only UI, and full single-tender settlement only. Preserve all deferred clinic requirements under F01–F25.
- [ ] **P01.04 · PAIR** — Review the six topic pages and reconciliation register against EVIDENCE. Resolve only conflicts that block a supported demo path; do not reopen settled business rules without a specific reason.
- [ ] **P01.05 · PAIR** — Classify the second owner call's bulk settlement, early upgrade, retrospective pricing, and waiver statements as unreconciled where appropriate. Record their impact on deferred work without silently promoting them to policy.
- [ ] **P01.06 · PAIR** — Confirm clinical-only review before issuance and separate payment after issuance. Demonstrate on paper exactly when care, fees, invoice lines, and receipts become saved.
- [ ] **P01.07 · PAIR** — Confirm patient-wide calendar recency and the 0–9/10–29/30+ fee boundaries. Identify all qualifying event types, including fee-only records even though their creation UI is deferred.
- [ ] **P01.08 · OM** — Choose the patient field and MRN rules from D06/D07. Record required/optional fields, valid unknowns, and identity/display distinctions rather than inheriting mock fields.
- [ ] **P01.09 · OM** — Approve or replace the proposed complaint-course save timing in D09. State what remains saved after canceling an unissued visit.
- [ ] **P01.10 · PAIR** — Agree the clinical draft's deselection/back/cancel behavior within this demo. Record the treatment-data consequences without declaring the wider parked A13 question settled.
- [ ] **P01.11 · PAIR** — Establish a small fixture catalog and explicit demonstration rates. Label them synthetic examples; do not present those values as newly approved clinic pricing.
- [ ] **P01.12 · AI** — Produce a screen/feature disposition list for existing components. Every visible control must work, be clearly unavailable with a reason, or be hidden within approved scope; no success-shaped placeholder actions remain.
- [ ] **P01.13 · AI** — Map supported requirements to the relevant audit items and deferred backlog. Show which items receive implementation evidence and which remain open beyond the demo.
- [ ] **P01.14 · PAIR** — Review the first two days' actual progress and update the delivery estimate. Record any extension or reduced demonstration claim explicitly; job research does not wait for an unbounded polish phase.

## P02 — Agree data, API, state, and permission contracts

**Depends on:** P01. **Decisions:** D03, D10–D22. **Gate:** G0. **Evidence:** reviewable contracts, minimal ERD, privilege matrix, and resolved decisions for the next slice.

- [ ] **P02.01 · AI** — Inspect actual database objects/configuration if access is authorized and available. Separate deployed facts from Markdown DDL, local migrations, seed data, and planned tables.
- [ ] **P02.02 · PAIR** — Review the minimum ERD for membership, patient, catalog, course, visit, bundled service, independent fee, invoice, invoice line, receipt, and operation identity. Record keys, cardinalities, and ownership; do not introduce a generic encounter table by assumption.
- [ ] **P02.03 · PAIR** — Choose the authoritative relationship that binds clinical sources to their invoice. Specify exactly-one-source and source uniqueness constraints without contradictory duplicate membership links.
- [ ] **P02.04 · AI** — Define patient registration/search/detail contracts. Include nullability, date-only DOB, MRN, referral data, normalization, bounded results, and backend field-error codes.
- [ ] **P02.05 · AI** — Define catalog and course contracts. Separate catalog ID, persisted course ID, temporary draft identity, immutable category, and catalog-derived anatomy; do not duplicate anatomy as an independently editable course field.
- [ ] **P02.06 · AI** — Define the treatment draft and checkout request. Include patient/course identities, bundled service selections, operation key, and contract version; exclude client-authoritative prices, owner, actor, status, and timestamps.
- [ ] **P02.07 · AI** — Define the issuance response and saved-invoice read model. Include stable internal/display references and their allocation rule, immutable lines, billed amount, receipts, current due, current state, and lost-response recovery data.
- [ ] **P02.08 · AI** — Define the full-payment request/response and Pay Later behavior. Separate intended tender/amount from authoritative amount validation, server actor, receipt time, and resulting state.
- [ ] **P02.09 · PAIR** — Choose exact paise transport and formatting rules. Test the design on zero, ordinary amounts, an invalid fractional value, and an integer beyond JavaScript's safe range before selecting implementation types.
- [ ] **P02.10 · PAIR** — Specify stable operation identity, payload canonicalization, key scope, result lookup, same-key conflict, and retention. Define replay independently for patient registration, course batch, checkout, and payment.
- [ ] **P02.11 · PAIR** — Review transaction boundaries and a consistent lock order. Cover same-key races, concurrent attendance, source reuse, concurrent settlement, rollback, and retryable database errors.
- [ ] **P02.12 · AI** — Create a privilege matrix for signed-out, authenticated nonmember, authorized admin, and operator/test setup. Include direct tables, views, RPC execution, and operation-result lookup.
- [ ] **P02.13 · PAIR** — Choose invoker/definer behavior and the enforcement point for each mutation. Record why callers cannot bypass atomic writes through exposed table permissions or a less protected endpoint.
- [ ] **P02.14 · PAIR** — Approve the smallest frontend state owner and route behavior. Draw allowed transitions from patient selection through issued invoice; distinguish draft, saved record, loading, failed, and unknown-outcome states.
- [ ] **P02.15 · AI** — Define error categories and user recovery for validation, conflict, authorization, unavailable configuration, transient network failure, uncertain write, and incompatible client version. Keep machine codes stable and messages actionable.
- [ ] **P02.16 · PAIR** — Choose draft/recovery persistence, sign-out clearing, and later outcome lookup behavior. Explain exactly what a refresh loses, preserves, or retrieves from the server.
- [ ] **P02.17 · AI** — Map approved contracts onto existing component props/types and repository files. Identify necessary adaptations from #95–#98 without silently reorganizing the project or installing libraries.
- [ ] **P02.18 · PAIR** — Approve required dependencies, migration/seed layout, API compatibility strategy, and test tools. Close G0 only when dependent implementation decisions have an explicit answer; remaining unrelated decisions may stay deferred.

## P03 — Reproducible environment, schema, and fixtures

**Depends on:** G0, authorized environment access. **Decisions:** D03, D05, D12–D16, D22. **Gate:** G1. **Checks:** AC-001–AC-004, AC-021, AC-023–AC-024, AC-040–AC-048, AC-106–AC-110 as applicable.

- [ ] **P03.01 · OM** — Identify the actual development/demo target and authorize the specific setup actions. Record whether data must be preserved; an available credential is not authority to reset a database.
- [ ] **P03.02 · AI** — Inventory existing tables, constraints, indexes, triggers, functions, grants, policies, views, and migration history. Record conflicts with the approved contracts before generating a migration.
- [ ] **P03.03 · AI** — Prepare versioned standalone migrations in the approved location. Document prerequisites and order; do not copy obsolete Markdown SQL wholesale or rewrite applied migration history.
- [ ] **P03.04 · AI** — Implement clinic membership and patient identity constraints from the approved model. Verify clinic-scoped MRN uniqueness and backend-derived ownership without assuming every authenticated account is an admin.
- [ ] **P03.05 · AI** — Implement catalog/course keys, immutable category, and required relationships. Validate anatomical and availability rules at the correct boundary; avoid an unapproved lifetime patient/catalog uniqueness constraint.
- [ ] **P03.06 · AI** — Implement visit and bundled-service relationships. Prevent services being attached to the wrong visit/course/clinic and preserve one visit per treated complaint without a blanket date uniqueness rule.
- [ ] **P03.07 · AI** — Implement independent extra-fee events and typed invoice-line source relationships. Enforce positive applicable charges, exactly one source, and one billing of each real occurrence under the approved design.
- [ ] **P03.08 · AI** — Implement invoice and actual-receipt storage with exact money types and terminal-state protection. Do not introduce authoritative stored totals that conflict with line/receipt-derived balances.
- [ ] **P03.09 · AI** — Implement the approved operation identity/result storage and uniqueness. Ensure the mechanism can participate in the same transaction as effects and cannot expose another caller's results.
- [ ] **P03.10 · AI** — Add only indexes justified by supported search, ownership, recency, history, and uniqueness queries. Record query purpose and inspect plans on representative data; avoid speculative indexing work.
- [ ] **P03.11 · AI** — Prepare versioned synthetic fixtures for empty/new/returning patients, multiple complaints, and issued invoices. Keep real patient data, credentials, and package-bearing cases out of normal demo fixtures.
- [ ] **P03.12 · AI** — Seed stable catalog/rate identities and define safe repeat behavior. Re-running setup must not duplicate catalogs, silently reprice issued lines, or overwrite a deliberate change without an explicit operator action.
- [ ] **P03.13 · AI** — Run clean setup in the authorized isolated environment. Record actual schema/fixture versions and failures; migration text alone is not proof it executes.
- [ ] **P03.14 · AI** — Test upgrade from the inspected existing baseline when one exists. Verify retained records and dependencies; if no baseline exists, record that fact rather than inventing an upgrade test.
- [ ] **P03.15 · PAIR** — Review the resulting ERD, constraints, migration diff, and remaining legacy objects. Confirm that no valuable data or unrelated schema was altered and explain the rollback/recovery limits.
- [ ] **P03.16 · AI** — Establish the approved frontend environment/configuration loading and API client boundary. Fail clearly on missing configuration; keep secret/service credentials out of browser code, logs, and committed files.

## P04 — Real sign-in and enforced authorization

**Depends on:** P03 and approved identity/privilege contracts. **Decisions:** D04, D16, D20, D23. **Gate:** G1. **Checks:** AC-005–AC-014, AC-100–AC-102.

- [ ] **P04.01 · AI** — Provision or document controlled demo accounts through the approved method. Public registration must not create clinic/admin membership; never commit passwords or reusable session tokens.
- [ ] **P04.02 · AI** — Implement the approved sign-in form using existing UI conventions. Show pending, invalid-credential, and service-error states without clearing useful input or claiming success prematurely.
- [ ] **P04.03 · AI** — Restore an existing session before resolving protected navigation. Show a bounded loading state rather than briefly exposing previous patient data or redirecting in a loop.
- [ ] **P04.04 · AI** — Resolve authorized clinic/admin membership after authentication. Distinguish signed-out from authenticated-but-not-authorized and prevent role selection in the browser from granting access.
- [ ] **P04.05 · AI** — Protect supported routes and API calls consistently. A manually entered patient/invoice URL must receive the same ownership checks as navigation from the app.
- [ ] **P04.06 · AI** — Apply read policies to all exposed tables and views. Verify cross-clinic IDs and broad-list queries with real nonprivileged accounts, not only an operator connection.
- [ ] **P04.07 · AI** — Apply mutation/execute privileges for the supported RPC boundary. Deny direct partial clinical/financial writes and revoke unintended default function exposure.
- [ ] **P04.08 · AI** — For each definer function, enforce the reviewed search path, qualified object references, caller identity, membership, and input ownership checks. Record concrete bypass attempts and results.
- [ ] **P04.09 · AI** — Implement sign-out and session-expiry handling. Clear patient/session state and bounded local recovery data according to D11, while preserving server records and a safe later lookup path.
- [ ] **P04.10 · AI** — Handle session changes during a read or write. Discard obsolete reads; treat uncertain mutations through their operation key rather than assuming an expired response proves rollback.
- [ ] **P04.11 · AI** — Prevent recovery lookups, errors, logs, or browser caches from leaking another patient's data after account change. Verify the allowed account can resume only its authorized operations.
- [ ] **P04.12 · PAIR** — Review sign-in versus authorization with Om using a direct API attempt. Om predicts why a nonmember cannot obtain clinic data even after bypassing the React route guard.

## P05 — Catalog reads and patient search

**Depends on:** P03/P04; approved field/catalog contracts. **Decisions:** D06, D08, D18, D20. **Gate:** G1. **Checks:** AC-015–AC-020, AC-027–AC-029, AC-093–AC-095.

- [ ] **P05.01 · AI** — Implement authorized complaint/service/rate catalog reads. Return stable identities and necessary availability/category metadata; do not expose price controls or fabricate missing settings.
- [ ] **P05.02 · AI** — Distinguish loading, empty catalog, configuration failure, and network failure. An unavailable catalog must block dependent selection with a useful explanation rather than silently allowing free text.
- [ ] **P05.03 · AI** — Implement bounded patient search by the agreed name/phone rules. Parameterize queries, handle literal wildcard characters deliberately, and avoid fetching the whole patient table into React.
- [ ] **P05.04 · AI** — Agree and implement minimum input/debounce behavior based on actual lookup needs. Preserve supported short names/phone inputs and provide a visible instruction instead of an unexplained empty list.
- [ ] **P05.05 · AI** — Return enough distinguishing fields for safe selection: saved patient ID, MRN, name, and agreed secondary detail. Shared names or phones remain separate candidates rather than an automatic merge.
- [ ] **P05.06 · AI** — Adapt the existing search result/keyboard interaction. Support up/down/Enter, accessible focus, explicit selection, and a clear registration entry point without alert-based placeholder actions.
- [ ] **P05.07 · AI** — Implement request identity and stale-result handling for search. Slow results for an old query must not replace the current query's results, including after input clearing or sign-out.
- [ ] **P05.08 · AI** — Distinguish no matches from failed search. Offer retry for failure and registration only as a deliberate choice, not as a fabricated conclusion that the patient does not exist.
- [ ] **P05.09 · AI** — Define result limits and load-more behavior only as needed for the demo dataset. Check that duplicates and truncation are visible enough to avoid selecting the wrong patient.
- [ ] **P05.10 · PAIR** — Review actual catalog/search responses against UI types. Confirm that labels are display data and that persistence/integration use stable IDs throughout.

## P06 — Patient registration that really saves

**Depends on:** P03–P05 and D06/D07/D14/D18 selections. **Gate:** G1. **Checks:** AC-021–AC-026, AC-071–AC-076, AC-096.

- [ ] **P06.01 · AI** — Adapt the intake form to the approved demographic contract. Remove or hide unsupported note entry and age-as-DOB conversion only within approved scope; no entered supported field may be silently discarded.
- [ ] **P06.02 · AI** — Implement optional DOB and explicit unknown presentation. Validate a real calendar date, reject future dates, and preserve the date across browser/timezone serialization without inventing a birthday.
- [ ] **P06.03 · AI** — Implement MRN input/allocation exactly as selected in D07. Show uniqueness conflicts without silently assigning a different identity or overwriting an existing patient.
- [ ] **P06.04 · AI** — Implement referral/doctor conditional fields and null/empty normalization. If using #97's parser pattern, submit its parsed/transformed output rather than the original object.
- [ ] **P06.05 · AI** — Validate legitimate names, agreed phone format, and required fields on the client. Remove obsolete arbitrary limits only when replaced by an approved field rule; associate each error with its control.
- [ ] **P06.06 · AI** — Implement independent backend validation and authorization. Reject unexpected authority fields and invalid referrals even when the client is bypassed.
- [ ] **P06.07 · AI** — Implement atomic registration with a stable operation key and returned saved patient identity. Same-key retry returns the same patient; conflicting payload reuse gives a conflict instead of a second registration.
- [ ] **P06.08 · AI** — Distinguish a uniqueness conflict with a different operation from a replay of the original operation. Return sufficient safe guidance to select the existing record or correct the MRN.
- [ ] **P06.09 · AI** — Change the submit contract to await actual persistence. Keep values intact while pending and after failure; reset/advance only after confirmed success or explicit user cancellation.
- [ ] **P06.10 · AI** — Disable duplicate submission while preserving backend replay protection. Enter, button click, double click, and a slow response must all use the same in-flight identity.
- [ ] **P06.11 · AI** — Handle an uncertain registration result through lookup/same-key retry. Do not replace a timed-out key and accidentally register a second patient.
- [ ] **P06.12 · AI** — Pass the returned patient into the selected-patient context. Refresh relevant search/history data without requiring the user to rediscover a newly registered patient.
- [ ] **P06.13 · AI** — Verify registration, refresh, sign-out/login, and retrieval from a new session. Demonstrate that canceling later treatment leaves the intentionally registered patient intact.
- [ ] **P06.14 · PAIR** — Review patient persistence and validation with Om. Om traces form value → request → server checks → saved row → returned identity and explains why a shared phone number is not automatically a duplicate person.

## P07 — Om owns the patient-to-complaint integration

**Depends on:** G1 and the approved state/route contracts. **Decisions:** D10, D19, D20, D24. **Gate:** G2. **Checks:** AC-030–AC-033, AC-097–AC-101, AC-111–AC-113.

This is a bounded learning slice, spread across the agreed independent blocks. AI prepares the stable API contract and explains nearby code; Om writes the central connection. If the exercise becomes paired, record that honestly and repeat a smaller independent change later.

- [ ] **P07.01 · PAIR** — Trace the current search callback, workflow owner, complaint selector props, and API client. Draw which component owns selected patient, loading state, courses, and error before changing code.
- [ ] **P07.02 · OM** — State the behavior in plain language before coding. Include selecting patient A, loading A's courses, switching to B, handling failure, and preventing A's delayed response from appearing under B.
- [ ] **P07.03 · AI** — Supply a documented, independently verified course-read API and a small example response. Give file references and hints rather than silently writing the exercise's integration patch.
- [ ] **P07.04 · OM** — Connect the actual selected patient callback to the approved state owner. Carry the persisted patient ID/MRN/name into the next step instead of a mock patient or display-name lookup.
- [ ] **P07.05 · OM** — Load that patient's saved courses using the approved read contract. Implement loading, success, genuine empty, and retryable failure states separately.
- [ ] **P07.06 · OM** — Implement the selected stale-response protection and prior-patient cleanup. Demonstrate rapid A→B selection with A returning last and confirm that only B's courses remain visible.
- [ ] **P07.07 · OM** — Preserve usable navigation when the request fails or selection changes. Retry must target the current patient and must not resurrect previously deselected patient data.
- [ ] **P07.08 · AI** — Review Om's bounded diff for actual behavior, type/contract correctness, and hidden state dependencies. Explain necessary changes with reasons; do not silently broaden it into a general state-management rewrite.
- [ ] **P07.09 · PAIR** — Verify selection by click and keyboard, a newly registered patient, no courses, server failure, and expired session. Record observed behavior and the exact portion independently written.
- [ ] **P07.10 · OM** — Trace one successful and one failed request through browser state and the server response. Identify which values are persisted identities and why a local UUID or patient name cannot replace them.
- [ ] **P07.11 · AI** — Extend only the accepted integration pattern to surrounding repetitive wiring. Preserve the ownership decisions and record any new abstraction proposal before implementing it.
- [ ] **P07.12 · PAIR** — Close the human-owned-path gate with a runnable result and a short explanation from Om. If he cannot explain the current behavior, pair on the smallest missing concept rather than adding another course playlist.

## P08 — Catalog-linked complaint courses

**Depends on:** P05/P07 and the approved D08/D09 contract. **Gate:** G2. **Checks:** AC-027–AC-039, AC-071–AC-076, AC-097–AC-099.

- [ ] **P08.01 · AI** — Adapt the selector's existing-course model to returned saved courses. Show stable clinical labels/category and use course ID for selection, without changing established layout beyond approved needs.
- [ ] **P08.02 · AI** — Preserve complaint catalog identity when a catalog item is selected. Replace the current title-only handoff; distinguish temporary draft identity from the eventual persisted course ID.
- [ ] **P08.03 · AI** — Add the approved one-time Routine/Rehab choice for a new course. Require an explicit valid value; do not infer category from package, service, title, or price.
- [ ] **P08.04 · AI** — Display/use the selected catalog's anatomical identity and capture only approved course fields. Reject an inconsistent required catalog mapping visibly and on the backend; do not invent anatomy or add an independently editable duplicate field.
- [ ] **P08.05 · AI** — Prevent duplicate accidental selection of the same existing course or same draft entry. Preserve the ability to represent separate legitimate historical courses under the selected uniqueness rule.
- [ ] **P08.06 · AI** — Implement the selected atomic course-batch save boundary. Validate patient/catalog/clinic ownership and return a complete draft-to-persisted-ID mapping; either the batch succeeds or none of it does.
- [ ] **P08.07 · AI** — Apply stable replay identity to course creation. Repeated Continue, lost responses, and concurrent same-key requests must not create extra courses.
- [ ] **P08.08 · AI** — Keep category immutable after persistence at the backend boundary. A forged request or alternate UI must not change classification on an existing course.
- [ ] **P08.09 · AI** — Handle disabled/missing catalog items and unsupported additions. Explain the seeded-demo limit and block invalid continuation; do not silently create uncataloged records.
- [ ] **P08.10 · AI** — Apply the approved deselection rule before and after new-course persistence. Clear the correct draft procedures and explain which saved patient/course records remain after canceling.
- [ ] **P08.11 · AI** — Advance only after confirmed course persistence and complete ID mapping. Preserve inputs on validation/failure and lock uncertain batches until their result is recovered.
- [ ] **P08.12 · PAIR** — Verify existing-only, new-only, and mixed course selection for two patients. Confirm no visits, fees, invoices, or payments are written by this step.
- [ ] **P08.13 · OM** — Explain catalog ID versus course ID and why classification is immutable. Inspect the saved result of one new course and its reuse in a later visit.

## P09 — Treatment draft and clinical review

**Depends on:** P07/P08. **Decisions:** D01, D10, D11, D19. **Gate:** G2. **Checks:** AC-049–AC-058, AC-096–AC-099, AC-125–AC-128.

- [ ] **P09.01 · AI** — Connect the existing procedure logger to the selected persisted courses and enabled service catalog. Remove mock context from the supported path without unrelated component cleanup.
- [ ] **P09.02 · AI** — Key each procedure selection by the correct course and service identity. The same service on two complaints must remain two distinct clinical selections, not overwrite one another.
- [ ] **P09.03 · AI** — Preserve grouped selections while navigating within the draft. Back/forward or opening a control must not reset unrelated complaint selections.
- [ ] **P09.04 · AI** — Enforce the approved minimum clinical content per treated complaint. If a supported rule permits no bundled service, represent that explicitly; do not invent a procedure to satisfy UI assumptions.
- [ ] **P09.05 · AI** — Remove cost-summing behavior from the clinical draft's authoritative meaning. Bundled selections describe performed care and must not create per-procedure charge lines.
- [ ] **P09.06 · AI** — Build the approved clinical review from patient, course/category, and selected services. Show no prices, payable total, or automatic return fee before issuance.
- [ ] **P09.07 · AI** — Provide edit/back controls that return to the correct draft. Preserve patient/course identities and selections while changing a procedure.
- [ ] **P09.08 · AI** — Implement explicit cancellation and patient-change behavior. Apply D11's draft rule and avoid implying that saved registration/course records are deleted.
- [ ] **P09.09 · AI** — Ensure no visits, fee events, invoice lines, or payments are persisted during draft editing or review. Prove this with before/after database counts for a canceled draft.
- [ ] **P09.10 · AI** — Validate the final draft before enabling Create Invoice and implement the agreed nonblocking same-day repeat notice. Distinguish a genuine additional treatment from resubmitting the same operation; client checks never replace server validation/replay protection.
- [ ] **P09.11 · AI** — Make Create Invoice the single supported issuance action. Replace console/alert/no-op callbacks and prevent accidental submission from unrelated form controls.
- [ ] **P09.12 · PAIR** — Walk through one-complaint and two-complaint drafts, deselection, back, cancel, and missing catalog/configuration. Confirm the UI describes only supported saved behavior.
- [ ] **P09.13 · OM** — Predict what is and is not in the database at review time. Explain why the clinical screen cannot claim care was saved merely because selections are visible.

## P10 — Atomic invoice issuance and authoritative fees

**Depends on:** approved contracts/schema/auth, G2. **Decisions:** D12–D16, D22. **Gate:** G3. **Checks:** AC-040–AC-076, AC-117–AC-121.

- [ ] **P10.01 · AI** — Implement the approved checkout RPC boundary with explicit contract validation. Reject malformed, unsupported, or inconsistent patient/course/service identities before any effect commits.
- [ ] **P10.02 · AI** — Derive caller, clinic, recorder, and authoritative timestamps on the backend. Do not trust client-supplied ownership, admin status, invoice status, fee, or price fields.
- [ ] **P10.03 · AI** — Enforce authorization and ownership for every referenced record, including nested service selections. A valid patient ID must not legitimize another patient's course or another clinic's source.
- [ ] **P10.04 · AI** — Implement the selected operation-key acquisition and same-payload replay/conflict behavior. Store the operation result and committed effects atomically; a partial receipt must not masquerade as a successful checkout.
- [ ] **P10.05 · AI** — Acquire the approved patient/source locks in the documented order before evaluating attendance or consuming billable occurrences. Record why simultaneous first visits cannot both read an empty history.
- [ ] **P10.06 · AI** — Query the last qualifying patient-wide attendance before inserting this checkout. Include ordinary therapy and independent qualifying fee events; exclude payments and unsupported invented events.
- [ ] **P10.07 · AI** — Calculate date differences in Asia/Kolkata calendar dates. Implement first attendance ₹350, return days 0–9 ₹0, 10–29 ₹150, and 30+ ₹350 without browser clock or elapsed-hour shortcuts.
- [ ] **P10.08 · AI** — Resolve the current configured flat therapy rate per selected course category. Missing/invalid prices must fail the whole checkout instead of generating a free visit or a guessed rate.
- [ ] **P10.09 · AI** — Create one clinical visit for each treated complaint and its bundled service rows. Preserve distinct same-day genuine visits while rejecting duplicate course entries within one submitted checkout.
- [ ] **P10.10 · AI** — Create at most the one applicable patient-level fee event for this attendance. Do not attach it arbitrarily to the first complaint or multiply it by complaint/procedure count.
- [ ] **P10.11 · AI** — Issue the invoice with a stable identity and the approved human-readable numbering rule. Enforce uniqueness under concurrency; do not derive numbers from client row counts.
- [ ] **P10.12 · AI** — Create one flat therapy line per visit and the applicable fee line. Snapshot descriptions, amounts, and source/category context required to interpret history after catalog changes.
- [ ] **P10.13 · AI** — Enforce exactly-one-source, source uniqueness, and consistent patient/clinic/invoice membership. Test the actual constraints and transaction boundary rather than relying on frontend selection limits.
- [ ] **P10.14 · AI** — Calculate the returned billed amount from persisted immutable lines. The supported ordinary-care demo must reject accidental zero/negative totals; future legitimate package-zero behavior remains separately scoped.
- [ ] **P10.15 · AI** — Commit care, fee events, invoice, lines, and operation result in one transaction. Inject a failure after intermediate work and prove that no orphan clinical or billing rows remain.
- [ ] **P10.16 · AI** — Return stable references and the saved invoice read model only after successful commit. Do not keep a transaction open while the user views the invoice or chooses a tender.
- [ ] **P10.17 · AI** — Return typed failures for invalid data, missing config, conflict, authorization, and retryable transaction failure. Ensure database details/secrets are not exposed and uncertain transport outcomes remain distinguishable.
- [ ] **P10.18 · AI** — Verify same-key concurrency, different-key genuine attendance, duplicate-source rejection, and first-attendance serialization in the real database. Record row counts and final invoice amounts for each race.
- [ ] **P10.19 · PAIR** — Review the full issuance boundary with Om. He predicts rollback, duplicate click, lost-response, changed-price, and two-complaint outcomes before checking the evidence.
- [ ] **P10.20 · OM** — Explain why atomicity, idempotency, source uniqueness, and locking solve different problems. Use the actual checkout implementation and one failed scenario rather than memorized definitions.

## P11 — Issued invoice and uncertain-submit recovery

**Depends on:** P09/P10. **Decisions:** D10, D11, D14, D15, D19, D20. **Gate:** G3. **Checks:** AC-049, AC-055–AC-058, AC-071–AC-076, AC-096–AC-105.

- [ ] **P11.01 · AI** — Replace the hard-coded invoice/mock patient with the saved issuance response. Display actual invoice identity, patient, line descriptions, amounts, state, and date from the agreed read model.
- [ ] **P11.02 · AI** — Create and freeze the operation key/payload when the user submits the reviewed draft. Duplicate clicks and transport retries must reuse that operation rather than rebuilding a changed payload.
- [ ] **P11.03 · AI** — Show an issuance-pending state and prevent conflicting edits/navigation according to D11. Preserve enough recovery information before sending the request to survive the approved interruption cases.
- [ ] **P11.04 · AI** — On confirmed failure with no committed effects, restore the editable draft and map actionable errors. Do not reset patient/procedure data or show a success notification.
- [ ] **P11.05 · AI** — On unknown outcome, show an explicit recovery state and use authorized lookup/same-key retry. Absence from one immediate lookup is not permission to create a new checkout key.
- [ ] **P11.06 · AI** — On confirmed success, transition once to the issued invoice. Clear only resolved draft/recovery state, retain saved references, and refresh affected activity/history reads.
- [ ] **P11.07 · AI** — Re-read the current invoice after recovering an older issuance result. A subsequently paid invoice must not reappear as newly UNPAID because an old response was cached.
- [ ] **P11.08 · AI** — Format paise through the approved exact money boundary. Verify labels and totals against the server result; no procedure-cost sum or current catalog price may replace saved line values.
- [ ] **P11.09 · AI** — Make issued invoices retrievable through the approved route/identifier. Refresh, later login, and a direct authorized link must load saved state without relying on the previous React draft.
- [ ] **P11.10 · AI** — Remove or clearly defer unsupported print/export actions. Closing an invoice view must not record a payment, reverse issuance, or silently create another invoice on return.
- [ ] **P11.11 · PAIR** — Verify visible state through success, rejection, timeout after commit, refresh during recovery, and later-paid replay. Confirm button labels describe real effects.

## P12 — Full payment, Pay Later, and later settlement

**Depends on:** P10/P11 and approved settlement contract. **Decisions:** D14–D17, D20. **Gate:** G3. **Checks:** AC-071–AC-089, AC-103–AC-105, AC-122–AC-124.

- [ ] **P12.01 · AI** — Implement the separate settlement operation for an already issued invoice. The request identifies invoice, stable key, Cash/UPI, and intended full amount, without granting client control of PAID or receipt metadata.
- [ ] **P12.02 · AI** — Authorize the invoice and derive recorder/receipt time from the backend. Deny another clinic's invoice and spoofed backdating/actor fields through direct requests.
- [ ] **P12.03 · AI** — Acquire the approved invoice/operation locks and recompute billed, collected, and current due from authoritative records. Use the consistent lock order and avoid line/receipt join multiplication.
- [ ] **P12.04 · AI** — Validate positive exact integer paise and the current full due. Reject underpayment, overpayment, invalid tender, stale intended amount, and unsupported split/waiver/refund in this demo with distinct useful errors.
- [ ] **P12.05 · AI** — Save one actual positive receipt and the valid invoice-state transition atomically. A failure between receipt and status handling must leave neither a misleading PAID invoice nor a partially committed receipt.
- [ ] **P12.06 · AI** — Apply same-key replay/conflict behavior independently from checkout. A repeated receipt request returns the original receipt/result; a different key cannot collect money twice once due is zero.
- [ ] **P12.07 · AI** — Protect PAID/WAIVED terminal states and immutable receipt data at the backend boundary. Unsupported demo actions must not provide an alternate mutation route through direct table access.
- [ ] **P12.08 · AI** — Implement Cash/UPI selection and deliberate confirmation of receipt in the invoice view. UPI is recording an actual externally received payment, not a fabricated gateway verification/integration.
- [ ] **P12.09 · AI** — Implement Pay Later as leaving the issued invoice UNPAID with no receipt. It may close/navigate the view according to the approved flow; it must not issue a second invoice or save zero money.
- [ ] **P12.10 · AI** — Allow an authorized user to reopen an outstanding invoice and settle it later. Show original billed amount, actual received amount, current due, and payment state distinctly.
- [ ] **P12.11 · AI** — Show pending, confirmed failure, unknown outcome, conflict, and success states for payment. Disable conflicting input while an uncertain receipt is being resolved; do not create a fresh payment key to dismiss a timeout.
- [ ] **P12.12 · AI** — Refresh invoice/history/activity after confirmed settlement or recovered result. A stale screen must re-read current due before allowing a second collection.
- [ ] **P12.13 · AI** — Verify two different payment keys racing for the same full due. Exactly one succeeds with a receipt; the other receives a settled/conflict result without overcollection.
- [ ] **P12.14 · AI** — Verify response loss after receipt commit and a delayed retry after another session reads the invoice. Confirm one receipt, the original recorder/time, and the current PAID state.
- [ ] **P12.15 · PAIR** — Review issuance versus collection with Om. He explains why Pay Later has no payment row, why closing a payment dialog leaves the invoice, and why browser-disabled buttons cannot enforce financial correctness.
- [ ] **P12.16 · OM** — Predict the persisted amounts/states for full Cash, full UPI, Pay Later, later payment, and a duplicate receipt attempt. Compare predictions to actual saved rows/read models.

## P13 — Saved activity, patient history, and balances

**Depends on:** P08/P10/P12. **Decisions:** D15, D19, D20. **Gate:** G3. **Checks:** AC-090–AC-095, AC-100–AC-105, AC-129–AC-130.

- [ ] **P13.01 · AI** — Implement authorized patient detail/course/visit/invoice reads from saved relationships. Remove mock lookup and random simulated failures from the supported profile path.
- [ ] **P13.02 · AI** — Implement the agreed daily activity query using Asia/Kolkata date boundaries. Every navigable row carries a required saved patient ID; label rows/counts by their real unit rather than reusing misleading queue badges.
- [ ] **P13.03 · AI** — Implement immutable billed totals from invoice lines and actual collected totals from receipts. Aggregate each independently before combining; validate against an invoice with multiple lines and multiple test receipts.
- [ ] **P13.04 · AI** — Display original invoice total, payments, outstanding balance, and status separately. A later receipt must change balance/state without rewriting original line descriptions/prices.
- [ ] **P13.05 · AI** — Display clinical history under the correct patient and complaint course. A multi-complaint checkout must show the right visit/service attribution and link to the shared invoice without double-counting its total.
- [ ] **P13.06 · AI** — Provide retrieval/navigation for an unpaid invoice and later settlement. Positive debt must not block starting a genuine new therapy draft for the same patient.
- [ ] **P13.07 · AI** — Hide deferred package/vitals/alert/dashboard sections and unsupported actions in the approved demo surface. Do not populate them with plausible but fabricated clinical or financial data.
- [ ] **P13.08 · AI** — Add loading, actual empty, permission/not-found, and failed-read states; validate retained URL complaint filters against this patient's saved courses. Apply the approved invalid-filter fallback without a false empty history or prior-patient content.
- [ ] **P13.09 · AI** — Refresh affected saved reads after registration, course creation, issuance, and payment. Confirm route return or explicit refresh sees the new state without duplicating local and server records.
- [ ] **P13.10 · AI** — Bound/sort history queries and define any necessary pagination. Use stable order and identifiers so reload/load-more does not hide or duplicate entries.
- [ ] **P13.11 · AI** — Verify persisted records after page reload, sign-out/login, and a fresh browser session. Confirm the flow works without any earlier in-memory patient/invoice context.
- [ ] **P13.12 · PAIR** — Review labels and aggregation with Om using actual saved examples. Explain invoice date versus receipt date and why PR #96's Paid-invoice sum is not a valid collections metric.

## P14 — Recovery and coherent workflow states

**Depends on:** G3. **Decisions:** D10, D11, D14, D20, D22. **Gate:** G4. **Checks:** AC-071–AC-076, AC-096–AC-105, AC-117–AC-124.

- [ ] **P14.01 · AI** — Inventory every async read/write in the supported flow and its state transitions. Confirm each has a defined loading/pending, success, failure, cancellation, and stale/unknown behavior where applicable.
- [ ] **P14.02 · AI** — Handle offline/unreachable service during search, registration, course save, checkout, and payment. Explain whether retry is a read, a same-key mutation, or a fresh action after a proven rollback.
- [ ] **P14.03 · AI** — Resolve lost responses after each durable write boundary. Compare operation lookup, server records, and same-key retry so the UI cannot mistake a committed result for a failed action.
- [ ] **P14.04 · AI** — Preserve frozen payload/key association while an operation is uncertain. Prevent editing the payload under the old key or issuing a replacement key for the same unresolved action.
- [ ] **P14.05 · AI** — Implement the approved refresh/tab-close recovery behavior. Record ordinary draft loss separately from durable-operation recovery; no warning may imply a saved invoice was discarded.
- [ ] **P14.06 · AI** — Handle sign-out/session expiry during an unresolved operation. Clear locally sensitive context according to D11 and allow an authorized later login to inspect the saved outcome safely.
- [ ] **P14.07 · AI** — Handle browser back/forward and direct navigation without valid draft context. Return to a safe entry point or fetch saved identifiers; do not reconstruct a fake patient or resubmit on mount.
- [ ] **P14.08 · AI** — Prevent stale search/profile/course responses from overriding a newer patient/session. Include component unmount/remount and repeated retry in the check, not only a simple request abort.
- [ ] **P14.09 · AI** — Handle a catalog/service/rate change between draft and issuance. Server validation must reject disabled selections or apply current approved issuance pricing consistently; saved invoices remain unchanged afterward.
- [ ] **P14.10 · AI** — Handle a second tab paying or issuing while the first is stale. Re-read authoritative state and present a useful conflict/result instead of optimistic duplicate success.
- [ ] **P14.11 · AI** — Map all backend error categories to actionable UI recovery. Preserve correct input and avoid revealing SQL internals, unrelated patient information, or credentials.
- [ ] **P14.12 · AI** — Remove success-looking placeholder alerts, silent catches, hard-coded IDs, and random failures from reachable demo paths. Inventory intentionally retained mocks outside the demo surface without unrelated deletion.
- [ ] **P14.13 · AI** — Ensure cancel/close controls perform only their labeled action at each stage. Canceling a draft, closing an issued invoice, and deferring payment must have distinct persisted consequences.
- [ ] **P14.14 · AI** — Handle incompatible frontend/API/schema versions explicitly. An old open tab must not create malformed records after deployment; record the chosen reload/compatibility response.
- [ ] **P14.15 · PAIR** — Review unknown-outcome UX with Om using a deliberately dropped post-commit response. He identifies the safe next action and explains why repeatedly pressing a newly generated submit is unsafe.
- [ ] **P14.16 · AI** — Record supported recovery limits and unresolved failures. No failure in a supported financial/permission path can be relabeled as a harmless demo limitation.

## P15 — Verify the supported system, not just individual screens

**Depends on:** P14 and approved test tooling. **Decisions:** D16, D21, D22. **Gate:** G4. **Checks:** all applicable AC-001–AC-130; learning/release checks finish in their later phases.

- [ ] **P15.01 · AI** — Establish test environments/fixtures and record exact frontend, schema, API, and fixture versions. Isolate destructive/fault-injection fixtures from the normal demonstration target.
- [ ] **P15.02 · AI** — Map each applicable acceptance scenario to an automated check or a repeatable manual procedure. Avoid tests that merely restate implementation; prioritize externally visible behavior and enforced invariants.
- [ ] **P15.03 · AI** — Run typecheck/build/lint and existing relevant tests using the repository's actual scripts. Report unavailable scripts and baseline failures accurately rather than claiming a generic green suite.
- [ ] **P15.04 · AI** — Test meaningful frontend state/validation behavior with the existing test runner. Cover stale patient responses, parsed form output, retained input on failure, frozen retry identity, and exact money formatting.
- [ ] **P15.05 · AI** — Test migrations and real database constraints with the actual installed engine. Validate relationships, source uniqueness, terminal immutability, and invalid direct-write rejection.
- [ ] **P15.06 · AI** — Test RLS and function privileges using signed-out, authenticated nonmember, authorized, and wrong-owner request contexts. Operator/service-role tests alone cannot satisfy this task.
- [ ] **P15.07 · AI** — Test fee boundaries with deterministic server-side fixture dates, including local midnight and 9/10/29/30 days. Do not weaken production timestamp rules by adding a client-controlled clock.
- [ ] **P15.08 · AI** — Test checkout rollback at meaningful intermediate failures. Assert exact absence/presence of patient/course versus visit/fee/invoice/line/operation records, respecting their distinct save boundaries.
- [ ] **P15.09 · AI** — Run two-session concurrency checks for same-key checkout, separate first attendances, source reuse, and settlement. Record successful/failed requests, final row counts, fees, receipts, and balances.
- [ ] **P15.10 · AI** — Test post-commit response loss for registration, course creation, checkout, and payment. Prove same-key recovery and payload-conflict behavior through the real request boundary.
- [ ] **P15.11 · AI** — Test balances against multi-line/multi-receipt fixtures and catalog price changes. Verify that aggregation cannot multiply amounts and historical billed values do not drift.
- [ ] **P15.12 · AI** — Run complete browser journeys: new patient/full Cash, existing patient/full UPI, multi-complaint/Pay Later, later settlement, and subsequent visit with outstanding debt. Verify database persistence after each.
- [ ] **P15.13 · AI** — Test keyboard-only operation, field-error association, loading/error announcements, focus return, and visible focus through the supported flow. Record concrete controls and outcomes rather than an unsupported accessibility certification.
- [ ] **P15.14 · AI** — Inspect actual rendered screens at the agreed desktop and narrow/mobile widths, including any relevant breakpoint. Verify navigation, overflowing names/amounts, dialogs, invoice readability, and the approved visual baseline.
- [ ] **P15.15 · AI** — Inspect reachable routes, network failures, console errors, build output, and configuration for accidental mocks/secrets/broken actions. Keep sensitive values out of screenshots and evidence logs.
- [ ] **P15.16 · PAIR** — Review failures and classify them as implementation defect, test/tool blocker, unresolved decision, or out-of-scope requirement. Fix/retest only authorized scope and update the affected evidence without hiding failed attempts.
- [ ] **P15.17 · OM** — Perform an unscripted supported journey while AI observes. Record actual usability/confusion points; passing automated tests do not substitute for being able to use the resulting demo.
- [ ] **P15.18 · AI** — Produce the G4 result matrix with actual outcomes, evidence paths, versions, and remaining release/learning checks. Do not mark NOT RUN or blocked scenarios as passed.

## P16 — Controlled deployment and release verification

**Depends on:** G4; explicit execution authority for external mutations. **Decisions:** D05, D22, D23. **Gate:** G5. **Checks:** AC-001–AC-014, AC-106–AC-110, AC-125–AC-130 on the deployed versions.

- [ ] **P16.01 · AI** — Prepare a concrete deployment/change list for the selected hosting/database targets. Include migrations, configuration names, auth redirect changes, build version, access arrangement, and recovery steps.
- [ ] **P16.02 · OM** — Authorize the specific deployment/account/configuration actions when the reviewable result is ready. Do not treat this planning document as blanket authority for publishing or database mutation.
- [ ] **P16.03 · AI** — Verify the resolved target/environment before applying an authorized action. Preserve valuable data and never run a reset against a computed or ambiguous project identifier.
- [ ] **P16.04 · AI** — Apply migrations/seeds in the approved order and record the actual deployed versions. Verify constraints/policies after deployment rather than assuming a successful build updated the database.
- [ ] **P16.05 · AI** — Configure frontend public values and protected server/operator secrets in their proper locations. Inspect the built browser bundle for unintended secret credentials without exposing them in logs.
- [ ] **P16.06 · AI** — Configure the approved sign-in redirect origins and session behavior. Verify direct visits to protected routes, callback handling, refresh, and sign-out on the actual deployed domain.
- [ ] **P16.07 · AI** — Verify SPA/deep-link hosting for patient and invoice retrieval. A pasted URL or browser refresh must not become a hosting 404 or redirect into the wrong patient context.
- [ ] **P16.08 · AI** — Run a deployed smoke journey with a newly created synthetic patient and invoice. Reopen from a fresh session and verify persistence; seed-only screenshots are insufficient.
- [ ] **P16.09 · AI** — Recheck critical authorization and payment/checkout boundaries against the deployed nonprivileged client. Confirm no environment-specific grant or policy exposes an alternate write path.
- [ ] **P16.10 · AI** — Verify old-tab/API compatibility and the approved recovery/rollback procedure. Record when rollback means frontend redeploy versus a separately designed data/schema change; do not promise destructive rollback as safe.
- [ ] **P16.11 · AI** — Verify the controlled reviewer-access arrangement and synthetic-data notice. Do not publish reusable admin credentials or expose an unauthenticated reset/admin endpoint.
- [ ] **P16.12 · AI** — Document operator setup, fixture restoration, and known demo limitations. Any data-reset procedure identifies the exact allowed target and requires the appropriate explicit authority.
- [ ] **P16.13 · AI** — Record release URL, frontend/schema/API versions, actual smoke-test evidence, and observed limits. A preview marked Ready by a deployment bot is not this verification.
- [ ] **P16.14 · OM** — Run the deployed demonstration personally and confirm the scope is represented accurately. Report any confusing or broken supported interaction before calling it ready to show.
- [ ] **P16.15 · PAIR** — Decide whether the result is a controlled demonstration or ready for wider reviewer access. Real-clinic launch remains F24 and is not implied by this gate.

## P17 — Understanding, truthful claims, and the walkthrough

**Depends on:** working implementation and evidence from earlier phases; deployed examples when available. **Decisions:** D24. **Gate:** G5. **Checks:** AC-111–AC-116, AC-125–AC-130.

- [ ] **P17.01 · AI** — Prepare a compact code map tied to actual files and request paths. Cover frontend state, RPC boundary, authorization, schema, checkout, payment, and saved reads without making Om read the whole repository at once.
- [ ] **P17.02 · OM** — Explain the patient-to-complaint path using his own implementation. Identify state ownership, async loading, persisted IDs, stale-response handling, and the recovery action after failure.
- [ ] **P17.03 · OM** — Make one small agreed change to that path without receiving a complete AI patch. Verify the result; record hints/pairing if needed rather than pretending the change was independent.
- [ ] **P17.04 · AI** — Introduce or describe one bounded, reversible failure in the isolated learning setup. Keep it out of the deployed release and provide the observable symptom without immediately supplying the fix.
- [ ] **P17.05 · OM** — Diagnose the failure using state, network response, and the relevant code. Explain the root cause and verify a correction; guessing until the UI looks right does not satisfy this exercise.
- [ ] **P17.06 · PAIR** — Trace one invoice from draft through committed rows to later receipt. Om explains why clinical review is not persistence, why payment is separate, and how historical line values remain stable.
- [ ] **P17.07 · PAIR** — Trace authorization through a direct request and identify where it is enforced. Om explains the limits of route guards, client validation, and public versus privileged credentials.
- [ ] **P17.08 · PAIR** — Review one actual rollback, one replay, and one race result. Om predicts the records that should exist and distinguishes atomicity, idempotency, uniqueness, and locking.
- [ ] **P17.09 · AI** — Compile a provenance record by task/feature: Om-written, AI-drafted/reviewed, paired, PR-adapted, or still not understood. Do not invent a percentage of independent authorship or hide unreviewed work.
- [ ] **P17.10 · PAIR** — Create a small learning backlog from remaining noncritical gaps. Separate topics needed to maintain the supported flow from deeper internals that can be learned later; do not reopen an endless full curriculum before applications.
- [ ] **P17.11 · AI** — Draft a short walkthrough grounded in a real saved journey and known limitations. Show registration/selection, care, issued invoice, payment/Pay Later, refresh, and history; use synthetic identities.
- [ ] **P17.12 · OM** — Deliver the walkthrough and answer follow-up questions using actual evidence. Clearly distinguish this demonstration, the wider unfinished clinic product, and any production outcomes that have not occurred.
- [ ] **P17.13 · PAIR** — Prepare factual project/resume talking points for later review. Claim demonstrated behavior and explain AI assistance if asked; publication or external outreach remains a separate instructed action.

## P18 — Handoff, remaining scope, and completion record

**Depends on:** G4 and completed G5 deployment/learning checks, or explicitly documented blockers that prevent a completion claim. **Gate:** G5. **Checks:** AC-106–AC-116 and final coverage review.

- [ ] **P18.01 · AI** — Reconcile the task log with the actual implementation and acceptance results. Every checked task has evidence; deferred or blocked work is visible and not counted as complete.
- [ ] **P18.02 · AI** — Record the selected versions of D01–D25 and any approved departures. Remove contradictory execution guidance while preserving historical decision provenance.
- [ ] **P18.03 · AI** — Update setup/run/verification instructions for the final implementation. Verify them in the authorized environment and label any manual provisioning or unavailable dependency honestly.
- [ ] **P18.04 · AI** — Summarize exact files changed by this implementation separately from pre-existing changes/untracked files. Report actual checks, failures, assumptions, and limits without attributing unrelated work to this task.
- [ ] **P18.05 · AI** — Update the compact F01–F25 backlog with newly discovered dependencies. Distinguish required clinic MVP, conditional MVP, post-MVP, unreconciled policy, and excluded product ideas.
- [ ] **P18.06 · AI** — Record dispositions of #95–#98 and relevant issues against the final baseline. Explain what was reused and what remains useful/deferred without automatically closing or merging anything.
- [ ] **P18.07 · AI** — Map implementation evidence back to A01/A02/A04/A06–A10 and other affected audit items. Propose narrowly justified status changes only through the audit's required process and explicit authorization.
- [ ] **P18.08 · PAIR** — Check the demonstration claim against the actual supported/deferred boundary. Confirm no mock metric, unimplemented button, undocumented unsupported package case, or missing financial recovery is being presented as complete.
- [ ] **P18.09 · OM** — Review the delivered demo, learning evidence, and remaining work. Record any final supported-path failure; approval of presentation cannot waive a known integrity defect.
- [ ] **P18.10 · AI** — Produce a concise handoff with entry URL/setup, versions, walkthrough, evidence, limitations, and the next incomplete dependency. Keep secrets and private career context out of public-facing artifacts.
- [ ] **P18.11 · PAIR** — Decide the next bounded workstream using actual progress and job-search needs. Do not let the detailed backlog become an automatic commitment to finish the entire clinic before applying.

## Gate closeout record

Complete this table during implementation, using links to actual evidence. It is intentionally unfilled at planning time.

| Gate | Current implementation status | Required evidence | Decision/reviewer record |
| --- | --- | --- | --- |
| G0: P00–P02 | NOT STARTED | Refreshed baseline, scope, contracts, selected blocking decisions | Pending |
| G1: P03–P06 | NOT STARTED | Reproducible persistence, real authorization, saved patient | Pending |
| G2: P07–P09 | NOT STARTED | Om's integration, stable course IDs, clinical-only draft review | Pending |
| G3: P10–P13 | NOT STARTED | Atomic issuance, separate settlement, real saved history | Pending |
| G4: P14–P15 | NOT STARTED | Recovery, constraints, race/security/browser results | Pending |
| G5: P16–P18 | NOT STARTED | Verified release, explanation/change/debug exercises, handoff | Pending |
