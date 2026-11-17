# Engineering decisions before implementation

**Status:** proposed implementation choices, 16 November 2026. Approved business rules are in the source records linked from [EVIDENCE](EVIDENCE.md). This file does not reopen those rules or silently approve new architecture.

Each entry has a recommended default so implementation can be discussed concretely. Om can approve related choices together at G0. A material departure, new dependency, abstraction, or broader scope needs explicit approval under the repository instructions. Routine details inside an approved approach do not need a new permission question. Record selected option, date, rationale and affected tasks when a choice is made. All choices below initially remain **PROPOSED**, unless described as an existing constraint.

## D01 — Exact demonstration boundary

**Choice:** full clinic MVP versus the non-package ordinary-therapy workflow in README. Full MVP serves the owner but includes substantial package and settlement engineering. The subset is faster to demonstrate but does not cover typical package users.

**Recommend:** the README subset, including multiple complaints, real authorization, issuance, full single-tender payment, Pay Later and later full payment. Seed catalogs; hide notes and package/standalone/fee-only entry; do not introduce partial/split/waiver/refund controls. Reject unsupported operations on the backend too. Document these as demo limitations, not forbidden clinic policy.

**Om decides:** this explicit boundary, including the additional notes/DOB/catalog cuts. **Blocks:** P02–P18 scope-sensitive implementation. A scope revision changes both tasks and acceptance scenarios before coding.

## D02 — Visual baseline and PR adoption

**Choice:** retain main's accepted presentation and port selected improvements, or adopt #98's broad redesign. The latter changes 79 files and shared primitives, increasing integration review work.

**Recommend:** main as baseline; consider #98 SearchInput submit prevention and ComplaintItem propagation handling as small verified patches. Selective #97 field-error presentation is another candidate. Do not merge old billing, validation, dashboard and styling branches wholesale. ScrollArea from #95 is optional only if a real scroll issue warrants it.

**Om decides:** intended appearance and any new shared abstraction. **Evidence:** [PR review](EVIDENCE.md). **Blocks:** UI implementation scope. Any selected styles need same-viewport visual checks, including keyboard focus and narrow layouts; source inspection alone cannot approve them.

## D03 — Stack and application boundary

**Choice:** existing React/Vite plus Supabase client/RPC, or an additional custom API server/framework migration. A server can centralize application logic but adds hosting, auth forwarding, routing and another implementation surface.

**Recommend:** retain React/TypeScript/Vite, existing router and UI primitives; add only the necessary Supabase client after version/config review. Use database operations for transactional domain writes. No Next.js rewrite, ORM, global-state library, query library or generic repository layer merely because integration starts.

**Om approves:** any dependency and module-boundary proposal. **Blocks:** P03/P04/P05. Actual installed-engine capabilities must be inspected before adding workarounds. Official reference: [database functions](https://supabase.com/docs/guides/database/functions).

## D04 — Demo identity and authorization

**Choice:** fake login, any authenticated account, or a provisioned account with explicit clinic/admin membership. Only the third demonstrates meaningful authorization.

**Recommend:** controlled admin account(s) for one synthetic clinic; sign-up cannot grant admin access. Backend derives identity and verifies membership, rather than accepting role/owner/recorder values supplied by React. Use an unauthorized authenticated account for negative tests; one operating clinic does not mean all logged-in accounts are trusted.

**Om supplies/selects:** authorized project, account-provisioning method and viewer-access arrangement; no credentials in this plan. **Blocks:** P04 and deployment. Test signed-out, expired-session, nonmember and forged-ID requests. Broader staff management remains F13.

## D05 — Isolated environment and migrations

**Choice:** adapt a known existing development database or create an explicitly authorized fresh isolated demo environment. Existing data/configuration may save setup but can hide obsolete schema and unsafe policies.

**Recommend:** inspect the actual project first. Prefer an isolated synthetic target with reproducible standalone migrations and seed fixtures. Treat Markdown SQL as source material, not an executable authoritative baseline. If any environment contains valuable data, inspect dependencies and design preservation/backfill separately; no destructive reset is implied.

**Om decides:** target and authority to create/apply/deploy. **AI prepares:** migration sequence, inspection results and recovery plan for review. **Blocks:** P03. Planning completion does not require credentials or database mutations.

## D06 — Demographics and age/DOB mismatch

**Choice:** collect exact optional DOB; model approximate age with its own semantics; or omit both for the demo. Converting age to an arbitrary birth date is not valid.

**Recommend:** optional real DOB with explicit unknown presentation, using the existing nullable SQL concept. Remove age-as-authoritative input in the supported demo path after approval. Define date-only serialization without browser timezone shifting. Determine name/phone/address requirements explicitly rather than inheriting arbitrary five-character names or a 50-character address limit from old validation.

**PAIR resolves:** field rules and nullable UI types. **Blocks:** P06. Keep M/F/X and referral mappings explicit; DOCTOR requires useful doctor details, other modes clear them. Shared phone numbers are not automatically duplicate patients.

## D07 — MRN and registration transaction

**Choice:** manual clinic MRN, generated counter, or treating a UUID as the displayed MRN. The old schema requires an MRN but the UI has no real allocation path.

**Recommend:** manual MRN for this small demo unless Om confirms an existing agreed generator. Enforce proper clinic/owner uniqueness with a friendly conflict response; do not silently generate a new identity on retry. Patient ID remains a separate database UUID. Patient registration is a legitimate saved record even if the subsequent visit is canceled.

**Om decides:** MRN rule. **PAIR resolves:** registration replay mechanism and whether any retained dependent patient records must save atomically. Notes are hidden if D01's cut is approved; never accept and discard them. **Blocks:** P06.

## D08 — Catalog identity, course creation and classification

**Choice:** seeded catalog-only selection, or admin catalog-add UI with attribution. Free-text course creation conflicts with current policy.

**Recommend:** seed a small representative complaint/service catalog; missing catalog entries explain the demo limit. New patient courses carry catalog ID and manually chosen Routine/Rehab. Existing course IDs and catalog IDs are different fields; a local UUID is never assumed to be a persisted course ID. Dedupe accidental draft selection by stable identity, while preserving the ability to represent distinct legitimate historical courses.

Current direction derives ordinary complaint anatomy through the selected complaint catalog. Do not add a second independently editable body-region field to the course merely because standalone treatments require anatomy. The shared physical representation remains audited.

Also select the minimum clinical content for the demonstration. Old issue #13 asks for at least one procedure per treated complaint but explicitly allows the business requirement to change; it is not conclusive policy authority. Recommend requiring one enabled bundled service for the illustrated demo journey unless current approved evidence specifies otherwise. Do not invent an unperformed service to pass validation; resolve the supported empty-service case before the checkout contract is implemented.

**PAIR resolves:** catalog/anatomy representation and course uniqueness rules; no unapproved lifetime patient/catalog uniqueness. **Blocks:** P05/P08. Full admin catalog additions and append-only attribution are F09.

## D09 — When new complaint courses are persisted

**Option A:** save selected new courses on Continue, then pass returned IDs onward. Clearer frontend learning path, but canceled visits may leave a valid unused course. **Option B:** include new-course creation inside final checkout, with draft-to-server mapping. Fewer interim writes, but a more complex checkout contract.

**Recommend for review:** A, with atomic batch course creation and stable replay identity. Course registration is distinct from recording performed care; no visits, fee events or invoices save at this step. Warn/describe that canceling treatment does not delete the registered patient/course.

**Om approves:** the write timing and resulting cancellation behavior. **Blocks:** P08/P10. If B is selected, revise payloads and tests together; do not keep both competing creation paths.

## D10 — Frontend workflow state and ownership

**Option A:** state at the existing workflow owner, with a small typed reducer and explicit props/callbacks. **Option B:** route-level/context/global store ownership. A is locally inspectable; B can help cross-route persistence but adds abstraction.

**Recommend:** inspect actual consumers, then propose the smallest owner that holds selected patient, selected courses, draft procedures, step, request state and issued invoice ID. A reducer is a proposal, not automatic authority to lift or restructure all state. Keep draft data separate from fetched persisted records. Do not duplicate backend totals as authoritative state.

**Om chooses architecture and writes:** the representative patient-selection/complaint-loading path. **AI:** surrounding repetitive plumbing after review. **Blocks:** P07/P09/P11.

## D11 — Draft cancellation and uncertain operations

**Choice:** in-memory draft only, minimal tab-local recovery storage, or backend drafts. Backend drafts are unnecessary product scope for this demo.

**Recommend:** ordinary draft in workflow state; preserve a bounded operation key and the frozen payload needed for retry in tab-local storage for synthetic demo operations, with explicit clearing on confirmed result/sign-out and a documented lifetime. Explain whether ordinary unsubmitted work survives refresh. Do not put credentials in that recovery record. If sign-out clears local recovery, saved history/server lookup must still allow outcome inspection after login.

**Critical:** unknown outcome is not failure. Lock edits to the submitted operation until resolved; a new key for the same uncertain action can duplicate it. A missing immediate lookup result does not prove an in-flight request rolled back. **PAIR decides:** exact persistence/recovery UX before P10/P12/P14.

## D12 — Checkout schema and source integrity

**Choice:** typed real source references and constraints versus a polymorphic text type/id with weaker FK enforcement. Current records prefer typed source FKs but leave exact DDL audited.

**Recommend:** review typed source relationships for demo visit and independent fee lines, with exactly one billable source and uniqueness per real occurrence. Prove patient/course/clinic/invoice ownership consistency through constraints and write-boundary checks. Choose one authoritative membership relation; do not create contradictory visit.invoice_id and line links without enforcement.

Do not add a generic sessions/encounters table without a separate justified decision. One checkout ID/operation grouping may support the demo, but attendance, invoice and visit are different concepts. **PAIR approves:** minimal ERD, constraints and extension boundary to packages/standalone care. **Blocks:** P03/P10; maps A01/A02/A06.

Also select the invoice display-reference contract: retain a stable internal ID and use either that reference in the demo or an approved server-allocated human-readable number. A separate sequential counter is not assumed necessary. If selected, define its clinic scope and concurrency behavior; never allocate from a browser row count or promise gapless numbers without an explicit requirement.

## D13 — Calendar recency and concurrency

**Options:** lock the patient's relevant row before reading recency; use stronger transaction isolation with retry; or design another proven serialization mechanism. A client check or uniqueness on patient/date alone does not solve all cases.

**Recommend:** a narrow patient-level lock around recency evaluation and issuance, subject to an explicit lock-order review. Read previous qualifying attendance before adding the current checkout. Use Asia/Kolkata calendar dates, and include existing fee-only qualifying events in the read model. Two complaints in one checkout add one fee; two genuine same-day checkouts remain allowed. No generic “one invoice per day” restriction.

S25 reports a preference for a warning about repeat same-day care rather than a ban. Recommend a nonblocking notice/confirmation of a genuine additional treatment where the relevant prior visit is known. Select its precise behavior at the draft contract review; it must not replace replay protection or automatically discard a legitimate second attendance.

**PAIR proves:** midnight, 9/10 and 29/30 boundaries and two concurrent first-attendance requests. Missing price/config causes rollback, never a silent free visit. **Blocks:** P10; A01/A04.

## D14 — Replay protection and outcome recovery

**Option A:** request identity on each created root record. Smaller per-operation schema, more repeated logic. **Option B:** narrowly scoped operation receipts with unique identity, server-computed canonical payload fingerprint and result reference. More explicit recovery, one additional persistence concept.

**Recommend:** evaluate B for registration/course batch/checkout/payment, without building a generic job framework. Scope keys by caller/clinic/operation as appropriate, authorize lookup, save the result reference in the same transaction as effects, and serialize same-key concurrent requests. Same key/same payload returns the original result; same key/different payload conflicts. Financial replay identities must not expire in a way that permits later duplication.

Re-read current invoice state after recovering an old issuance result; the invoice may since have been paid. A distinct request key does not excuse double-billing the same source. **PAIR approves:** key lifetime, fingerprint, atomicity, lookup and error behavior. **Blocks:** P06/P08/P10/P12/P14; A08.

## D15 — Money representation and balance reads

**Choice:** bounded safe-integer paise in JavaScript, or explicit integer-string paise at the API boundary with precise parsing/formatting. PostgreSQL BIGINT can exceed JavaScript's exact number range.

**Recommend:** define one exact transport contract before code; integer-string paise is a strong candidate. Backend computes authoritative amounts; frontend formats without floating-point rupee arithmetic. Never silently round an unsafe parsed integer. Amounts, limits and unexpected values need explicit validation.

Compute line totals and payment totals independently before combining. Do not join multiple lines and multiple receipts and sum their Cartesian multiplication. No new authoritative header total/outstanding column. **PAIR selects:** representation and query shape. **Blocks:** P02/P10–P13; A09. Demonstration fixture rates are not newly approved clinic prices.

## D16 — Database privileges and RPC execution

**Choice:** invoker functions with appropriate table privileges/policies, or tightly restricted definer functions with explicit caller checks. Both need a privilege design; naming a function “secure” is not enforcement.

**Recommend:** decide from the required mutation boundary. If only RPC may create financial records, a narrow definer function may be appropriate: fixed search path, qualified objects, explicit authentication/membership and ownership checks, minimal execute grants, no arbitrary SQL, and no exposed bypass through direct table writes. Invoker mode is preferable where it can enforce the same boundary without exposing invalid partial writes.

RLS/read policies must cover tables and views; test with actual nonprivileged callers, not only a service/admin connection. **PAIR reviews:** privilege matrix and bypass attempts. **Blocks:** P04/P10/P12. See [RLS](https://supabase.com/docs/guides/database/postgres/row-level-security).

## D17 — Settlement subset and unsupported states

**Recommend:** demo operation accepts invoice identity, stable request key, one Cash/UPI tender and the intended full received amount. Backend locks the invoice, computes current due, validates that the amount exactly settles it within this demo subset, writes one positive receipt and transitions atomically to PAID. Pay Later leaves the issued invoice UNPAID and creates no payment. A canceled settlement does not undo issuance.

Do not let the client choose PAID or fabricate received_at/recorded_by. Partial, split, waiver and refund requests are rejected as unsupported by this demo, not described as prohibited clinic policy. Decide whether physical enum reserves all four states now; no reachable state may lack valid read/error behavior. **PAIR approves:** contract and tests; **blocks:** P12.

## D18 — Registration and search validation

**Choice:** adapt #97's Zod pattern, or keep small explicit validation functions matching the existing code. A library can centralize parsing, but adds dependency and may reproduce old schema mistakes.

**Recommend:** select after D06 field contract. If Zod is used, consume validated/transformed output rather than validating and submitting the original object. Backend validates independently. Normalize safely; preserve legitimate names and shared phone numbers. Search must handle literal wildcard characters and bounded results without concatenated SQL or unbounded queries.

**Om approves:** dependency and field rules. **AI implements:** error associations and server mapping. **Blocks:** P05/P06. Do not silently expand this into a universal validation framework.

## D19 — Routes, history, and ledger meaning

**Recommend for review:** home is saved daily activity + patient search, patient/:id shows real history, and an issued invoice has a recoverable address/identifier. Decide whether workflow steps share one route or use separate routes; direct linking to a later step without context must recover safely rather than show another mock patient.

Propose one ledger row per saved checkout, with complaint/visit summary and separately labeled payment state. Counts must say whether they count patients, visits or checkouts. Existing Waiting/In Therapy badges must not imply persisted queue state. Patient history uses saved course/visit/invoice relations; unsupported package/vitals panels are hidden, not filled with fake data.

Every navigable ledger row needs its saved patient ID. If existing profile URL filters are retained, validate their complaint identity against the currently loaded patient's courses. Recommend a safe default history view with a visible correction for an invalid filter, not a misleading empty history. Use deterministic sorting with a stable tie-breaker. These adapt existing issues #88–#90 without adding a new timeline redesign.

**Om approves:** visible route/row behavior and reuse. **Blocks:** P07/P13/P16.

## D20 — Stale responses and read consistency

**Recommend:** make every request's patient/session identity explicit; abort or ignore outdated results and clear prior-patient data while loading the new one. After a confirmed write, refresh or update the relevant saved record using the server response, then invalidate affected ledger/history reads. On a read failure, show stale data only if clearly labeled; never convert an error into an empty successful result.

A query library is optional and requires approval; existing hooks may suffice. Payment/checkout should not claim optimistic success. **PAIR approves:** minimal read lifecycle approach. **Om implements one instance** in P07; AI extends it in P13/P14.

## D21 — Test tools and evidence

**Recommend:** retain Vitest for meaningful frontend logic/contract tests. Inspect installed tooling before choosing database integration and browser test tools. A new Playwright/SQL-test dependency is a proposal. Database tests must run against the actual engine/migrations; UI-only mocks cannot prove locking, RLS or rollback.

Fault injection belongs in an isolated test harness, not publicly available production switches. Use two database sessions for races and a transport harness that drops a response after commit for ambiguous outcomes. Report failures separately from tooling/environment blockers. **Om approves:** minimal additions; **blocks:** P15 automation. All scenarios in ACCEPTANCE start NOT RUN.

## D22 — Schema updates and version compatibility

**Recommend:** versioned migrations, fixture version and API contract version tracked with the frontend build. Test clean setup and an upgrade from the actual inspected development baseline when one exists. Preserve data unless a specifically authorized synthetic reset is chosen. Review old triggers, counters, generated rows and permissive policies; copying table definitions without their dependencies is insufficient.

Define rollout order and behavior for an older open browser tab against the new API. A clear incompatible-version error is preferable to a malformed financial write. **PAIR decides:** minimum compatibility strategy and rollback/redeploy procedure. **Blocks:** P03/P16; A10.

## D23 — Deployment and public demo access

**Choice:** guided demo with a controlled account, a publicly shared writable account, or a dedicated constrained demo-access mechanism. Public admin credentials invite arbitrary fixture changes and require a separate design.

**Recommend:** controlled access for the first demonstration and a recorded walkthrough; decide broader reviewer access only after the core works. Keep environment values appropriately separated; never ship secret/service credentials. Confirm auth redirect configuration and deep-link hosting behavior. Data reset is an operator-only, target-checked procedure and requires authority; no unauthenticated reset button.

**Om decides/authorizes:** deployment target, access sharing and external actions. **Blocks:** P16 public availability, not local implementation or planning.

## D24 — Human learning gates and AI boundaries

**Existing direction:** AI-heavy delivery with a human-owned representative frontend integration. **Recommend:** one 30–45-minute independent block within demo time; AI may explain/hint but not silently supply that exercise's full patch. If Om is stuck, pair explicitly and record what was assisted; no shaming or fake independence.

Before accepting critical generated backend code, Om predicts normal/failure behavior and explains the tested boundary. He need not master PostgreSQL internals before shipping a synthetic prototype. Unexplained noncritical details can become a learning backlog; core permissions, money and write timing cannot be deferred indefinitely while claimed as understood.

**Om owns:** explanation, small change and diagnosis at P17. **AI owns:** accurate evidence/provenance, bounded patches and useful instruction.

## D25 — Remaining source conflicts and change control

Do not turn the second owner call into implemented bulk settlement, retroactive package conversion, early upgrade eligibility or a new waiver prerequisite. Likewise, do not treat the old MVP index, Module 1 open alternatives, frontend ADRs, or issue titles as overriding S37.

**Recommend:** record each affected subject in EVIDENCE, seek a focused decision only when it blocks a supported task, and reconcile full records separately if explicitly requested. This scoping package may be complete while later engineering decisions remain pending. Any new requirement gets a demo/deferred/rejected disposition, owner, rationale, affected contracts and tests before implementation.

**PAIR owns:** the source and impact review at P01 and each phase gate. Existing audits close only through their required implementation evidence and separately authorized record updates.
