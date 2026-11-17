# Inspection evidence and reconciliation

**Checkpoint:** 16 November 2026. This is a source inspection for planning, not a runtime or security certification.

## 1. Repository and Git baseline

| Repository | Local evidence | Live default branch | Verified relationship |
| --- | --- | --- | --- |
| clinic-demo-be | main, `33f6b38d812ae3530bb827e4a331e1de3b7739df` | main, same SHA | 0 ahead / 0 behind |
| clinic-demo | Readable local HEAD `e2c2f4db9fb565ee3562029d642be4c98220abae`; source files readable | main, same SHA | Commit tips identical; working-tree status/diff scan failed with permission/work-tree errors, so unstaged/untracked state is unverified |
| omsrivastava.in | codex/parent-route-back, `4c30ecd6d484679c138c1910d43b8a5703577596` | main, `b19f82222536ecf233227a0b73f8df56cac3c4c9` | 1 ahead / 0 behind; ahead commit concerns BackButton navigation |

All three authoritative remotes are Om's `omsrivastava512` repositories, not a portfolio template upstream. Live default names and tips were read through GitHub. No fetch, checkout, merge, rebase, staging, commit, push, or PR mutation was performed.

Pre-existing backend changes: AGENTS.md, plans/mvp/mvp-index.md, two historical discussion heading edits, and untracked audit/transcript/plan material. Pre-existing portfolio changes: old blogging paths deleted with files under notes/blogging, and notes/career untracked. These are not changes made by this task. A content-hash baseline of 187 existing tracked/untracked files across the two writable repositories was captured; the final comparison found every captured file unchanged and only the five new planning documents added.

Equivalent-work search covered frontend and backend open/closed PRs and commit searches for integration, Supabase, persistence and authentication. Backend returned no PRs. The old backend commit `ff4a450` adds documented Supabase migration SQL; it does not prove a deployed current backend. Frontend integration issue [#50](https://github.com/omsrivastava512/clinic-demo/issues/50), Final App [#49](https://github.com/omsrivastava512/clinic-demo/issues/49), and Workflow [#47](https://github.com/omsrivastava512/clinic-demo/issues/47) remain open. No completed equivalent to the proposed persisted workflow was established.

## 2. Source authority

| Source | Use | Limitation |
| --- | --- | --- |
| [Module 2 reading guide](../../study/module_2/decisions/README.md) and [reconciliation register](../../study/module_2/decisions/reconciliation-2026-10-27.md) | Current policy/source precedence through S37 | Documentation is not executed implementation |
| [Clinical records](../../study/module_2/decisions/clinical-records.md) | Course identity, immutable category, ordinary vs standalone care, atomicity | Exact schema remains an engineering decision |
| [Catalog/pricing](../../study/module_2/decisions/service-catalog-and-branch-pricing.md) | Single clinic, two flat rates, enabled services, admin catalog authority, stable names | No new final numerical flat rates are selected by the reconciliation; demo prices must be labeled fixtures |
| [Checkout](../../study/module_2/decisions/checkout-and-invoice-lines.md) | Clinical-only preview, issuance boundary, immutable source-linked lines | Idempotency and exact physical source mapping remain audited |
| [Payments](../../study/module_2/decisions/payments-and-settlement.md) | Four states, positive receipts, terminal settlement, no overcollection | Later package/refund additions coexist with an older closing paragraph saying no refund is introduced; use subject-specific S31/S33 additions for planned scope, do not pretend refund DDL is settled |
| [Packages](../../study/module_2/decisions/packages.md) | Required full MVP packages and S25–S37 rules | Upgrade delivery is conditional; demo exclusion does not repeal requirement |
| [Examination fees](../../study/module_2/decisions/examination-fees.md) | Patient-wide calendar recency, independent fee-only events | Attendance mapping and race handling remain audited |
| [Engineering audit register](../../study/module_2/decisions/UNRESOLVED_AND_AUDIT_REGISTER.md) | A01–A04, A06–A11; legacy and deferred work | Summary shorthand A01–A10 is not the actual contiguous list; A05 is parked and A11 is active. Its claim that upcoming is cleared is stale relative to the file currently present |
| [Second owner-call transcript](../../study/module_2/decisions/upcoming/clinic-owner-call-02-english-transcript_2026-11-05_22-22-25_IST.md) | New reported needs, ambiguities, and potential contradictions | Preparation date is not proven call date. Local untracked source at this checkpoint. Availability and approval require separate verification |
| [Module 1 decisions](../../study/module_1/DECISIONS.md), [gaps](../../study/module_1/GAPS.md) | Patient identity, creator attribution, old security gaps | Three-role, branching, invoice/session, retroactive-edit proposals may be superseded by Module 2 |
| [Documented SQL](../../docs/schema/supabase_migration.md) | Existing DDL draft and migration starting material | Markdown SQL, old business assumptions and permissive policies; not a ready-to-run current migration |
| [Retry audit](../../docs/schema/idempotency_retry_audit.md) | Lost-response failure analysis and candidate mechanisms | Local untracked source at this checkpoint. 17 October scope statements about deferred packages/Pay Later are superseded; mechanism choices were not approved |
| [Old workflow contract](../../docs/contracts/FRONTEND_WORKFLOW.md), [wiring proposal](../frontend-backend-wiriing-proposal.md), [screen audit](../workflow_screens_audit.md), [UX audit](../ux_design_audit.md) | Locate interfaces, mismatches and prior alternatives | The UX audit is local and untracked at this checkpoint. Use current policy for fees, sessions, roles and visit types |
| [Old MVP index](../mvp/mvp-index.md) and business handovers | Historical rationale and alternatives | Locally edited but still conflicts with current fee/rate/package/branch rules; newer file edit time does not establish authority |
| Career handoffs in C:/Github/omsrivastava.in/notes/career | Om's learning history, explicit AI provenance, capacity and career purpose | Old branch descriptions and deadlines must be refreshed; private career material is not copied into public project prose |

The decision tracker workbook was not edited or re-certified by this planning task. Its current interpretations are not inferred from historical columns. This plan does not claim that the staged call has been reconciled across the workbook and six topic pages.

### Staged-call points to preserve

T019–T022 reports packages used by roughly five or six of ten patients; this reinforces why a non-package hiring demo is not the operational MVP. T007–T017 discusses upgrade timing with ambiguity. T028–T031 supports an original-start anchor, but wording about month end must not replace the explicit S37 Day-30 rule without reconciliation. T043–T044 describes retrospective package-like arithmetic, not an approved conversion algorithm. T060 describes a payment prerequisite for waiver that conflicts with approved unpaid whole-invoice waiver use cases. T075–T090 raises accumulated invoices and later concessions; patient-level allocation/bulk settlement are proposals left for another discussion, not existing features. These affect F04/F05/F06/F16; the demo avoids making concessions or upgrades while preserving ordinary unpaid care and per-invoice settlement.

## 3. Current frontend evidence and intended reuse

Paths in this table are relative to [frontend main at e2c2f4d](https://github.com/omsrivastava512/clinic-demo/tree/e2c2f4db9fb565ee3562029d642be4c98220abae). Local counterparts were readable; no clean-tree claim is made.

| Files | Observed behavior | Planned reuse / required change |
| --- | --- | --- |
| src/router.tsx, src/App.tsx, DemoAppHeader.tsx | Catalogue, dashboard, roadmap and patient routes; child route error boundaries | Retain routing/error foundation; choose operational routes explicitly; hide dead/demo-only navigation |
| src/pages/vistitworkflow.tsx | Mock patients/complaints, no-op callbacks, alert invoice, hardcoded number, independent presentation sections | Reuse leaf components, replace catalogue composition with a single selected-patient workflow |
| features/ledger/index.tsx, PatientSearch/index.tsx, TableBody/TableRow | Local mock filtering, selection alerts, mock-dependent patient navigation | Real query results and callbacks; define saved row unit; preserve keyboard selection |
| features/patient/NewPatientIntake/index.tsx | Age form; transforms names; validates local strings; resets before onSubmit; optional notes array | Async submit contract, truthful DOB/unknown handling, field mapping, pending/errors; reset only after confirmed save |
| features/patient/types/index.tsx versus src/types/index.tsx | clinicalNotes array vs Patient.notes singular; sex/referral vs gender/referral_mode; age vs DOB | One documented boundary mapping; no lossy casting or invented date |
| ComplaintSelector/index.tsx and hook/useComplaintSelection.tsx | Catalog selection calls add(item.title); creates local UUID; title-based deduplication loses catalog ID | Stable catalog and course IDs, explicit category, separately named draft IDs, persist/reconcile new courses |
| ProcedureLogger/index.tsx and hook/useProcedureLogger.tsx | Composite selection keys; local procedure-price sum; fallback “Complaint Not Found” | Reuse grouping/selection; build clinical payload; remove routine per-procedure billing and reject invalid references |
| InvoiceBuilder and FooterActions | Sum of caller-supplied items; payment selection not a persisted settlement; confirm closes/alerts | Display issued server invoice; full-payment/Pay Later; no client-supplied authoritative status/total |
| PatientProfile/index.tsx | Mock read with random 20% error simulation; separately filtered mock visits/packages/invoices | Real reads; deterministic error tests; retain loading/error/not-found presentation |
| PatientProfile tabs, patientUtils.ts, status-badge.tsx | Old visit billing, package expiry/day-log model, Paid/Pending/Overdue labels | Saved invoice-line read model and current terms; hide deferred panels; retain relevant presentation only |
| package.json, vitest.config.ts, patientUtils tests | React 19/TS/Vite, existing primitives and Vitest; no Supabase SDK dependency in main | Keep stack; approve minimum SDK/test additions; existing test success cannot validate current clinic rules |

## 4. Open PR inspection and selective reuse

All four open frontend PRs were enumerated afresh. Changed-file lists, patches relevant to behavior/data/contracts, descriptions and combined review/comment timelines were inspected. All returned discussion entries were Vercel bot preview notices; no human review/approval was returned. Preview-ready/build claims below are historical reports, not checks rerun in this inspection. Source-level review does not establish rendered visual quality or absence of regressions.

| PR | Exact head | Ahead / behind current main | Intent and documented waiting reason |
| --- | --- | --- | --- |
| [#95](https://github.com/omsrivastava512/clinic-demo/pull/95) | 537b75a2b212fdf4e17594fde15accdc864efcb2 | 3 / 1 | Derive old visit billing at render time + ScrollArea. Open, not draft. No explicit human blocking reason found in returned GitHub discussion; do not invent one |
| [#96](https://github.com/omsrivastava512/clinic-demo/pull/96) | ba6a4fbd054b65fc131450b54752d595da454f4f | 3 / 33 | Configurable dashboard mockup. Draft; description explicitly says do not merge pending main type/data reconciliation |
| [#97](https://github.com/omsrivastava512/clinic-demo/pull/97) | c9f540332a14a2c7acd3ce6d768cafc703af1adb | 1 / 1 | Client Zod write validation. Draft; visit-payload portion explicitly blocked. Om's career handoff says single-prompt output not yet reviewed by him |
| [#98](https://github.com/omsrivastava512/clinic-demo/pull/98) | 8049c91a5e0ad36fe6080f909dfdb82526c3f753 | 6 / 0 | Editorial visual redesign. Open, not draft; Om's handoff says single-prompt output not yet reviewed. No additional human blocker recorded in returned GitHub discussion |

### PR #95 — separate the reusable presentation from superseded finance

12 changed files. The ScrollArea wrapper centralizes scrollbar classes and forwards a ref. It is a possible reuse candidate if the selected UI needs it; its extra wrappers and height behavior still require rendering checks. It is not a prerequisite for persistence.

The billing half removes consultationFee/servicesTotal/grandTotal fields, adds standalonePrice to VisitService, and recomputes history through calculateConsultationFee/calculateServiceCharge. Its premise is the old CONSULTATION/MACHINE_ONLY and STANDARD/PREMIUM model. Current requirements instead bill immutable invoice lines and flat complaint therapy. The general goal of deriving totals is useful, but the source of truth must be issued lines/payments, not current procedure/rule calculations. Do not port that billing implementation unchanged. Added ADR-PP-27/28 are branch-local explanations, not approval to override Module 2. The diff modifies a src/data/mock_data.tsx path whereas main uses .ts; inspect rename/history when rebasing or manually porting, rather than assuming filename parity.

### PR #96 — useful future dashboard, explicitly unfinished

18 changed files. Introduces seven widget types, a registry, layout configuration and settings modal; modifies shared button/card/input primitives and adds dialog/checkbox components. Dashboard needs can selectively reuse its visual composition later.

Observed mismatches: revenue sums Paid invoice amounts rather than receipt records; action items use Pending/Overdue and treat an overdue-days number as a boolean filter; this would hide dues against the accepted all-positive-balance rule. Recent patients means the first five mock profiles, not recorded recency. Referral/clinic-health values are hardcoded; 30-day revenue is random on render. Quick-action routes do not exist in current router. Settings are in-memory and the modal initializes draft state from props; reopen/cancel synchronization needs explicit verification. The broad *.txt gitignore and diff.txt artifact are not integration requirements. The three billing models and notes/DOB mismatches described in the PR remain valuable diagnostic evidence, not proof all eight listed problems are still identical on current main.

Keep dashboard customization out of the demo path. When revisiting, define data semantics and routes first, then selectively port widgets and test shared primitive changes. Do not merge the entire 33-behind branch to obtain a card layout.

### PR #97 — useful error presentation, obsolete payloads and remaining gaps

12 changed files, adds Zod dependency. FieldError and field-level feedback are useful candidates. Referral conditional validation is also useful after the final field contract is chosen. However, validation still collects age, has broad any typing for notes, uses old visit enums, accepts CARD, and uses Paid/Pending/Overdue. The payment handler fabricates paymentStatus='Paid' in the browser and falls back to onClose when onConfirm is absent. Current settlement must derive state from saved receipts on the backend.

An additional integration gap exists even within the intended validation approach: the schema transforms doctorInfo but validatePatientIntake returns only success/errors; the caller submits the original form after deepTrimStrings, not parsed/transformed data. It also resets before an awaited backend save. Reuse the intent and error UI selectively; define the new validated output contract, pending state and failure retention. FieldError needs proper input association/announcement where used. The PR does not implement database persistence or backend validation.

### PR #98 — visual work with small behavior changes worth considering

79 changed files spanning theme, shared primitives, shell, ledger, intake, profile, complaint and invoice presentation. It adds external Outfit/Newsreader fonts, a constrained main layout and new visual tokens. It removes the initial Start Workflow gate but leaves the component catalogue, mocks, no-op callbacks, hardcoded invoice and alert payment behavior intact.

Selective functional candidates: SearchInput prevents native form submission; ComplaintItem's delete click stops propagation so it does not also toggle the parent selection. These are candidates to verify in the integrated UI independently of adopting the whole redesign. Its “zero logic regressions” description is not a test result from this review: behavior does change. App.tsx falls back to Date.now for a missing last-updated environment value, which can imply a fresh build on every load; use a real build identity in the final demo. Some integration TODO explanations are removed while the underlying catalog-ID gap remains. Visual choices require Om's selection and viewport/focus checks before use.

### PR interactions and adoption order

| Pair | Overlap | Consequence |
| --- | --- | --- |
| #95 / #98 | Seven profile files: history panel, service tag, profile root, billing/overview/packages/visits tabs | Scroll wrappers and financial presentation interact with redesign; manually inspect intended composition |
| #96 / #98 | button, card, input primitives and dashboard page | Shared component and page changes can affect unrelated screens; do not treat these as independent merges |
| #97 / #98 | Intake demographics/referral/root, invoice footer/root, workflow page | Preserve new validation and async contracts when adapting styling; either wholesale version could discard desired work |
| #95 / #96 / #97 | No exact changed-path intersection between these pairs in returned lists | Still semantically coupled through old types, billing and shared data; no textual overlap does not mean compatibility |

Recommendation: current main as the starting point; decide visual baseline once; port approved useful changes in bounded diffs with provenance. Before adopting an old branch, update it against live main through an explicitly authorized merge/rebase or manually reapply selected changes to an authorized new branch. Rebase rewrites branch commit identities; merge retains history and adds a merge commit. Neither was performed. No recommendation here closes or labels a PR useless.

## 5. Old-to-current reconciliation map

| Older assumption / location | Current rule or unresolved point | Planning consequence |
| --- | --- | --- |
| Markdown migration: row exists = complete + paid; old MVP index full-payment-only | Care issuance and later collection separate; Pay Later approved | Replace write/read contracts, not just labels; P10–P13 |
| PR #95/main: first/subsequent consultation + per-procedure amounts | Flat Routine/Rehab per complaint and separate calendar fee | Backend pricing and invoice snapshots; P09–P11 |
| Old docs: lifetime joining fee or fee on every new complaint | Patient-wide qualifying attendance, calendar bands 0–9/10–29/30+ | D13, A01/A04 and fee boundary tests |
| Current migration: single invoices.visit_id | One checkout can include multiple visits and distinct billable sources | D12; no fake one-invoice-per-visit shortcut |
| Old header amount / render-time repricing | Original billed = sum immutable issued lines | D15; independent payment aggregation; history remains stable |
| PR #97/CARD/old payment labels | Cash/UPI, four current states, backend transition authority | New payment contract; preserve valid subset only |
| Mock “Waiting/In Therapy/Paid” clinical ledger | Stored visits are completed care; invoice state is separate | Decide ledger row meaning and label; no arbitrary status toggling |
| Main/PR #98 title-based complaint identity | Stable complaint_catalog FK and permanent course category | P08; typed draft/server IDs, no free-text identity fallback |
| Intake age / profile DOB | No exact birth date can be recovered from age | D06; explicit optional DOB proposal; no invented January 1 |
| Clinical notes array / singular notes / alerts | Separate concepts; no approved automatic notes-to-alerts bridge | Hide notes in demo if deferred; revisit F10 with exact schema |
| Earlier multibranch price matrix and three application roles | One clinic/admin-only; reception post-MVP | D03/D04; keep existing identity integrity without expanding tenancy |
| Old package expiry / payment-gated activation / freeze | Soft review, visit entitlement independent of payment, no freeze, permanent closure | Hide old mock packages in demo; build F01–F04 to current policy |
| Old scope excludes packages | Packages required for clinic MVP | Demo is explicitly a subset, not a renamed MVP |
| Old refund-exclusion paragraph | S31/S33 planned refund workflow with admin authority | F06 remains planned; full accounting design must be settled separately |
| Summary says upcoming fully cleared | Second owner-call transcript is present with new unresolved proposals | Do not claim universal reconciliation; track F16 |
| Old issues #34/#35/#38, “fake login” #51, optimistic updates #56 | Atomic completed care, real auth, conservative financial success | Adapt issue intent; do not implement literal old wording |

## 6. Existing issue and merged-work disposition

The returned GitHub collection contains 98 entries: 90 issues (73 open, 17 closed) and eight PRs (four open, four merged). These counts describe the inspected collection at this checkpoint. The following map accounts for every returned open issue, while separating its useful intent from old wording. It is a planning disposition, not an instruction to close issues or an invented explanation for why an author left them open.

| Open issues | Disposition for this demo and later work |
| --- | --- |
| #47, #49, #50 — workflow/final app/integration | Central demo objective: P07–P13 connects saved patient, course, care, invoice and receipt. Existing umbrella tickets do not establish completion. |
| #51, #67, #68 — authentication/users/permissions | #51's fake login and three-role example are superseded for this slice. Real admin membership/enforcement is P04; staff-management UI/reception are F13. |
| #37, #39–#42, #66, #70 — preferences/settings/charges/visit types | Demo uses approved seeded settings and backend pricing. Do not implement old visit-type billing, a new generic preference hook, or password prompts merely from issue titles. Operational management is F09; future branch differences are F14. |
| #13, #28–#30, #86 — procedure minimum, catalog and selection | Persisted stable catalogs and actual enabled services are P05/P08/P09. #13's minimum-one-procedure wording explicitly allows business changes; D08 resolves the demo contract. Preserve usable existing search/navigation; do not rebuild an exhaustive mock catalog or custom popover framework by default. |
| #10, #11 — Zod and trimming | D06/D18 choose approved field/validation rules. Main already trims submitted intake fields (#11). Verify that normalized data reaches persistence. Adapt #97 selectively; parsed output, backend validation and async persistence matter more than adopting a named library everywhere. |
| #8, #9, #33–#36, #38, #88 — ledger/status/patient ID | P13 uses actual saved activity and required patient IDs. Old Enrolled/In Therapy/end-of-day discard/Waiting/Paid-as-visit-state proposals must not override atomic completed care and separate invoice state. Queue features are F19. |
| #43–#45 — credit/Pay Later/discounts | Pay Later and later full settlement are P12. Partial/split/whole-residual waiver are F05; do not interpret “credit” as a wallet or “discount” as mutable issued prices. |
| #52–#55 — loading/errors/notifications/confirmation | Supported feedback and safe draft cancellation are P06–P15. #55's patient deletion/payment void examples do not authorize those deferred/restricted mutations. Use appropriate existing feedback without a generic notification-system rebuild. |
| #61–#64 — IST/accessibility/tests/error boundary | Calendar semantics, meaningful real-engine tests and usable keyboard/error behavior are demo requirements. Reuse current route error boundaries; test their behavior instead of duplicating them. |
| #56–#58 — optimistic updates/SSE/offline | Online failure and uncertain-outcome recovery are P14. Offline writes/realtime/optimistic financial success are not part of this demo; broader capabilities are F22. |
| #21, #48, #59, #82, #89, #90 — search/navigation/URL/sorting | Current search hints and keyboard usability fit P05/P15. Patient-bound URL-filter validation and stable ordering fit P13. Universal search/recent-patient workspace/global shortcuts remain F20 unless a supported usability need requires a bounded change. |
| #71–#78, #81, #84 — dashboard and mock aggregation | Hide unsupported metrics and replace supported history/dues with real queries. Dashboard widgets/analytics are F12/F20; Main already removes #84’s repeated per-card filtering and addresses #81’s module-scope calculation. Data and metrics remain mock-based. [Dashboard change](https://github.com/omsrivastava512/clinic-demo/commit/0479140) |
| #69 — reports | F12 after trustworthy source records. Reporting uses actual receipt dates, not Paid-only invoice sums. |
| #7, #14, #60 — localization/printing | F21. No fake Print Receipt success in the demo, and no silent assumption that a printed payment receipt is required now. |
| #15–#20, #27, #46 — appointments/queues/reminders | F19. They do not create demo scheduling, no-show or arrival-fee rules. |
| #91–#93 — contextual new complaint, expanded history, resolve/relapse | #91 explicitly says V2; #92 says future history growth; #93 explicitly awaits clinical lifecycle decisions. Keep bounded existing history readable now; F20/F17 preserve expansion and parked A05 behavior. |

Closed issue groups are useful regression/reuse context, not fresh implementation tasks: #1–#2 search layering/keyboard scrolling; #3–#6 intake address/notes fixes; #12/#31/#32/#87 complaint search/scroll behavior; #22–#26/#79/#80 profile, tabs/routes, responsive/light-mode work. Re-verify retained behavior where integration touches it. A closed issue does not prove the current persisted application exists.

| Merged PR | Observed purpose / why it matters now |
| --- | --- |
| [#65](https://github.com/omsrivastava512/clinic-demo/pull/65), merged 8 July | Patient profile, URL filters, mock courses/billing and utility foundation. Reuse presentation and route patterns selectively; its STANDARD/PREMIUM, consultation prices, optional patientId and clinical mock types are not current domain authority. Follow-up issues #88–#90 remain relevant. |
| [#83](https://github.com/omsrivastava512/clinic-demo/pull/83), merged 1 June | Presentational catalog/captions/development notices. Evidence of component presentation work, not database integration. |
| [#85](https://github.com/omsrivastava512/clinic-demo/pull/85), merged 4 July | Complaint catalog-search popover, portal, sticky input and controlled scrolling. Reuse accepted selection affordances; title-only handoff still needs stable catalog identity. |
| [#94](https://github.com/omsrivastava512/clinic-demo/pull/94), merged 21 July | Compact package cards, day-log popovers and Expired/date presentation. Later package soft-review/entitlement policy changes make its data semantics historical; visual elements may be revisited for F01–F04 after current contracts exist. |

These records explain why visually substantial work can already be merged while the end-to-end application remains unfinished. They also prevent creating duplicate profile/search components just because persistence has not been connected.

### 18 November 2026 ticket follow-up

Live backend main was `69cfca4` and live frontend main was `e2c2f4d`. The frontend inventory still contains 73 open issues, 17 closed issues, four open PRs and four merged PRs. The backend has zero GitHub issues and PRs. The rows below capture findings that affect execution. [Frontend issues](https://github.com/omsrivastava512/clinic-demo/issues), [frontend PRs](https://github.com/omsrivastava512/clinic-demo/pulls)

| Finding | Effect on the plan |
| --- | --- |
| #11, #64, #81 and #84 have existing frontend work | Verify intake trimming, reuse the route error boundaries and retain useful dashboard fixes when replacing mocks. [Intake](https://github.com/omsrivastava512/clinic-demo/blob/e2c2f4d/src/features/patient/NewPatientIntake/index.tsx#L128), [router](https://github.com/omsrivastava512/clinic-demo/blob/e2c2f4d/src/router.tsx#L15), [dashboard change](https://github.com/omsrivastava512/clinic-demo/commit/0479140) |
| #30/#86 are only partly addressed | Complaint search already supports keyboard selection. The Procedure Logger search input has no handler. P01.12 records whether the demo wires or hides it. [Complaint search](https://github.com/omsrivastava512/clinic-demo/blob/e2c2f4d/src/features/visit/ComplaintSelector/index.tsx#L117), [procedure input](https://github.com/omsrivastava512/clinic-demo/blob/e2c2f4d/src/features/visit/ProcedureLogger/components/primitives.tsx#L74) |
| #8, #13 and #58 still need scoped decisions | Confirm the saved-ledger filter, minimum bundled procedures and whether an offline banner belongs in the demo. Retry recovery is already in P14. [#8](https://github.com/omsrivastava512/clinic-demo/issues/8), [#13](https://github.com/omsrivastava512/clinic-demo/issues/13), [#58](https://github.com/omsrivastava512/clinic-demo/issues/58) |
| #90 needs a specific fixture | AC-090 already checks tied timestamps. Include two complaint courses with the same start date if the timeline is retained. [#90](https://github.com/omsrivastava512/clinic-demo/issues/90) |
| N23 applies to bundled-service writes | P03.06 and P10.09 carry attribution into the demo. AC-041 checks creator identity and creation time on the saved bundled-service rows. [N23](../../study/module_2/decisions/UNRESOLVED_AND_AUDIT_REGISTER.md#category-b-legacy-schema-dispositions) |

## 7. Audit coverage and limits

Inspected: live repository/default metadata, local HEAD/history/status where permitted, pre-existing diffs, all four open frontend PR descriptions and changed-file inventories, relevant behavior/data patches, returned PR comments/reviews, all frontend issues returned by the collection, current Module 2 topic/register material, staged owner-call text, Module 1 identity gaps, and integration-relevant frontend sources. Older contracts and plans are supporting history; they are not wholesale approved specifications.

Not established: clean frontend worktree, deployed Supabase schema, credentials/project configuration, production use, correctness of old CI claims on today's environment, rendered acceptance of PR #98, every CSS change in that broad redesign, branch merge conflict freedom, or passing application/concurrency tests. These limits are explicit tasks before implementation/release, not evidence that work is absent everywhere.

Technical references checked for the proposed implementation direction: [Supabase database functions](https://supabase.com/docs/guides/database/functions), [Supabase row-level security](https://supabase.com/docs/guides/database/postgres/row-level-security), and [PostgREST transactions](https://docs.postgrest.org/en/stable/references/transactions.html). RPC can expose database functions; exposed data needs appropriate authorization; a request transaction does not combine separate browser requests into one transaction. Exact installed versions and privileges must be verified during implementation.
