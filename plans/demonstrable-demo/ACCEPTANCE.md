# Demonstrable demo — acceptance and evidence

**Current result for every scenario in this file: NOT RUN.** This is a verification specification, not a report of a working application. Repository inspection and PR preview notices do not satisfy these checks.

The scenarios assume the proposed scope and recommendations in [DECISIONS](DECISIONS.md). Select those decisions before implementation. If an alternative is approved, revise the affected expected outcomes and [tasks](TASKS.md) together; do not quietly skip a required scenario. A blocked check remains blocked, not passed.

## Test setup and result recording

Use an isolated synthetic environment with known frontend, schema, API, and fixture versions. Record each result as `PASS / FAIL / BLOCKED / NOT RUN`, with actual steps/output, evidence location, actor/context, versions, date, and any failure explanation. Keep failed attempts visible when recording a later pass. Use automation for meaningful state, database, and concurrency checks; repeatable manual checks are appropriate for visual, keyboard, and learning outcomes. No claim of comprehensive security/accessibility certification follows from this set.

**Example fixtures, not newly approved clinic rates:** Routine therapy ₹200 (20,000 paise), Rehab therapy ₹300 (30,000 paise). First-attendance fee ₹350; return fee after 10–29 days ₹150. A first checkout treating one Routine and one Rehab complaint therefore bills ₹850; two Routine complaints bill ₹750. A first single Routine visit bills ₹550. Bundled procedures add no separate price. Use clearly synthetic names/MRNs and test tenders; the demo does not move real money or verify a payment gateway.

Prepare an authorized admin, an authenticated nonmember, and signed-out requests. A second ownership/clinic fixture may be used solely to attempt unauthorized access; it does not create a multi-clinic product requirement. Operator privileges may prepare fixtures but cannot prove application authorization works.

For deterministic fee tests, use a captured server-local date or an isolated test clock/fixture mechanism. Example anchor: 16 November 2026 in Asia/Kolkata. Create historical fixture records through privileged test setup, not a production backdating field. Test near local midnight with explicit instants. Runtime requests must still use authoritative server time.

Every mutation test checks both the response/UI and resulting database records. Failed checkout must leave the previously registered patient/course intact when appropriate, while leaving no new visit/fee/invoice/line. Payment rollback must preserve the already issued invoice. Same-key replay and different-key legitimate repeat work are different tests.

## Environment and repeatability

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-001 | Apply the approved migration sequence to an authorized empty isolated target; start the app using documented configuration. | Schema, constraints, grants, policies and supported queries exist; the app connects. Record actual versions and commands/results without secret values. No reliance on manually pasted undocumented SQL. |
| AC-002 | Upgrade from the inspected development baseline when one exists, with representative saved records. | Migration completes without unexplained loss or orphan relationships; preserved records remain readable. If no prior target exists, document that applicability decision and prove clean setup instead; do not invent an upgrade pass. |
| AC-003 | Omit an essential public config value, then restore it; inspect the built browser artifact. | Missing configuration produces an understandable setup failure. The working build contains no service/secret credential. Public client configuration is distinguished from privileged credentials. |
| AC-004 | Run the approved fixture setup twice and inspect catalog IDs, rates and records. | Repeat behavior matches the documented operator contract: no duplicate catalog entries, unexpected overwrite, historical repricing, or accidental reset of unrelated data. The normal demo dataset contains no real patient or package-bearing case. |

## Authentication and authorization

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-005 | Sign in with a provisioned authorized admin, refresh, then open an allowed patient route. | Session restoration and membership resolution complete before patient data renders. The correct account/clinic is used and no login loop or previous-account flash occurs. |
| AC-006 | Enter invalid credentials and simulate an unavailable auth service. | Distinct actionable errors; useful input retained; no authenticated state, patient data, or false success. Pending controls recover after the response. |
| AC-007 | Visit protected routes and call protected reads/mutations while signed out. | UI provides the approved sign-in path; direct backend access is denied. A route guard alone is not accepted as evidence. |
| AC-008 | Authenticate a nonmember and request patient, catalog, invoice and operation-result endpoints. | Authentication does not grant clinic authority. All protected data and mutations are denied according to the privilege matrix, including direct API calls. |
| AC-009 | With an authorized account, substitute a patient/course/invoice ID owned by the negative-test clinic. | No foreign data or mutation is allowed. Nested references are checked independently; a valid own-patient ID cannot authorize a foreign course. |
| AC-010 | Attempt direct table inserts/updates/deletes that would bypass supported transactional functions. | Application roles cannot create partial care/invoice/receipt state, rewrite issued amounts, or delete financial history through an alternate exposed route. Record actual nonprivileged responses. |
| AC-011 | Enumerate exposed function execution and invoke each supported mutation with a nonmember; inspect definer behavior if selected. | Grants and explicit caller checks agree. Fixed search path/qualified references are verified, and neither default PUBLIC execution nor another function bypasses membership checks. |
| AC-012 | Sign out from patient/invoice screens, then use back navigation and sign into another account. | Prior patient/session data and local recovery context are cleared as agreed; server records remain. No previous patient's details leak through a cached view or late response. |
| AC-013 | Expire/change the session while a read and then an issuance/payment request are in flight. | Obsolete reads are discarded. A potentially committed mutation enters authorized recovery rather than being declared rolled back; later lookup does not expose another account's outcome. |
| AC-014 | Forge recorder, owner, role, created/received time, invoice status, or price in a direct request. | Protected values cannot be supplied authoritatively; unexpected contract fields are rejected or safely excluded according to the approved contract. Persisted actor/time/amount derive from the backend. |

## Patient search and registration

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-015 | Search a known patient by each supported name/phone input, including the agreed shortest valid input. | Correct bounded matches with saved ID, MRN and distinguishing details; visible minimum-input guidance where applicable. No client-wide patient-table download. |
| AC-016 | Search duplicate names/shared phone numbers; select the second result by click and by keyboard. | Results remain distinct patients. Selected context uses that result's persisted ID, not a title/phone match or array position from an older result set. |
| AC-017 | Search for no matching record, then make the server return an error. | Genuine no-match and failed search have distinct UI states. Failure is not presented as proof that the patient does not exist; retry targets the current query. |
| AC-018 | Start query A, then B; deliver B first and A last; then clear the input during another request. | Only current-query results remain. Clearing/unmounting/sign-out prevents stale results from reopening or replacing the list. |
| AC-019 | Search with supported punctuation, literal `%`/`_`, whitespace, and SQL-like text. | Behavior matches the approved normalization/literal-search contract; requests remain parameterized and bounded. Inputs cannot broaden access or produce an unhandled database error. |
| AC-020 | Search more matches than the result limit and use the approved continuation behavior. | Stable ordering and explicit truncation/load-more behavior; no duplicate/missing entries caused by unstable pagination or misleading claim that all patients are shown. |
| AC-021 | Register a valid synthetic patient with agreed required fields and MRN; reload and retrieve it. | Exactly one saved patient with correct ownership, normalized supported fields and stable identity. The form advances/resets only after confirmed persistence. |
| AC-022 | Submit valid optional DOB, unknown DOB, invalid calendar date and future DOB. | Real dates retain the same date across serialization; unknown stays unknown; invalid/future dates fail. No birth date is manufactured from age and no timezone shift changes the day. |
| AC-023 | Use DOCTOR referral with missing/valid doctor details; switch to another referral mode; inspect submitted and stored data. | Approved conditional rules hold on client and server. Cleared/transformed fields actually use parsed output; stale doctor details do not survive through submission of the original unparsed object. |
| AC-024 | Submit an already used MRN with a different registration operation, and a new MRN with a shared phone. | Duplicate MRN produces a recoverable conflict without overwrite/new hidden identity. Shared phone is allowed unless an explicitly selected field rule says otherwise. Existing patient can be deliberately selected. |
| AC-025 | Fail a registration request before commit; then retry after correcting a field. | Useful values remain; no new patient exists from the failed attempt. Validation errors map to controls and pending state ends correctly. The successful revised action follows the approved operation-key rule. |
| AC-026 | Register a patient successfully, enter a treatment draft, then cancel and sign in again. | Registered patient remains retrievable. No unissued visit, fee, invoice or receipt was created; cancellation messaging does not imply patient deletion. |

## Catalog identity, patient context, and complaint courses

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-027 | Select a catalog complaint and inspect draft, save request and returned course. | Catalog ID remains intact through each boundary. Draft ID, catalog ID and persisted course ID are distinct; a local random UUID/title is not passed as a saved course identity. |
| AC-028 | Load a disabled service/catalog item, or disable a selected item before submission in isolated setup. | UI uses availability information and backend rejects unsupported new use according to the contract. Historical records remain readable; no silent replacement or free-text fallback occurs. |
| AC-029 | Start with an empty catalog and then simulate catalog/configuration read failure. | Empty and failed reads are distinguishable. Selection/issuance is blocked where required, with a useful explanation of seeded-demo limits rather than invented catalog records. |
| AC-030 | Select patient A and load saved courses through Om's integration. | Patient header, course request and rendered courses use A's persisted identity. Record the actual request/state path and the independently implemented portion. |
| AC-031 | Select A, immediately select B, deliver B's course response first and A's last. | Only B's courses can be displayed/selected; A's draft selections are cleared or handled by the approved patient-change contract. Om explains the stale-response mechanism. |
| AC-032 | Select a patient with no courses, then a patient whose course read fails. | Empty offers the approved new-course action; failure offers retry without pretending there are no courses. Neither displays a previous patient's courses. |
| AC-033 | Register a new patient and continue directly into complaint selection; retry after a course-read error. | The returned patient ID is carried through without mock substitution or re-search. Retry uses the current patient and preserves only valid context. |
| AC-034 | Create a course with Routine and another with Rehab, including approved required clinical fields. | Explicit classification and catalog identity persist correctly; ordinary complaint anatomy derives from that catalog. Missing/invalid mappings are rejected, not filled with invented values or an independently editable duplicate anatomy field. |
| AC-035 | Attempt to change a saved course's category through UI/direct request. | Category remains immutable under application privileges. A selected service, price or package cannot implicitly reclassify the course. |
| AC-036 | Select the same saved course twice, then create a distinct legitimate course sharing a catalog item where allowed. | Accidental duplicate selection is prevented without an unapproved lifetime patient/catalog ban. Saved relationships reflect the chosen uniqueness contract. |
| AC-037 | Save a batch of new courses with one invalid item; then submit a fully valid batch. | Invalid batch leaves none of its new courses. Valid batch returns a complete identity mapping before continuation; no visit/fee/invoice is created by course registration. |
| AC-038 | Mix existing and new courses, select procedures, then deselect/reselect one course. | Only the intended draft selections change under the approved rule. Persisted courses are not silently deleted; procedure data cannot drift to a different course. |
| AC-039 | Save a new course at the selected D09 boundary, then cancel treatment and reload. | If the recommended early-save option is chosen, the course remains and no care/billing exists. If another option was approved, update this expected outcome and dependent tasks before execution. |

## Database relationships and immutable financial sources

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-040 | Try creating a visit with a course belonging to a different patient/clinic. | No inconsistent visit commits. Verify the actual ownership constraint/write boundary, not just filtered UI options. |
| AC-041 | Attach a bundled service row to the wrong visit/course, or submit a nonexistent service. | Invalid linkage is rejected; valid per-course selections remain separate. No dangling service or partially saved visit remains after checkout failure. |
| AC-042 | Attempt an invoice line with no billable source and one with two source references. | Exactly-one-source enforcement rejects both. A valid visit or independent fee source can be billed only through the reviewed source contract. |
| AC-043 | Attempt to bill the same real source occurrence twice, including through a different request key/invoice. | Source uniqueness rejects reuse. The new key does not bypass occurrence integrity; genuine new visits remain separately billable. |
| AC-044 | Try linking an invoice line/fee source from a different patient or clinic to an otherwise valid invoice. | Consistent ownership is enforced for all source types supported by the demo. No cross-patient balance or history contamination results. |
| AC-045 | Issue an invoice, change current catalog labels/rates through privileged fixture setup, and reread history. | Saved descriptions and amounts remain the issuance snapshot. New permitted issuance uses current settings; historical totals do not derive from today's service-price algorithm. |
| AC-046 | Attempt to update/delete issued lines and recorded receipts through application privileges. | Historical financial records cannot be rewritten or deleted. The supported API provides no generic CRUD bypass; operator-only test preparation is not confused with app capability. |
| AC-047 | Mark fixture invoices PAID/WAIVED through valid isolated setup; attempt forbidden terminal transitions or further collection. | Terminal state and immutability rules hold. Unsupported waiver creation is not exposed in demo UI/API, but existing supported read behavior does not rename the state. |
| AC-048 | Generate invoices concurrently and inspect identifiers, amounts and transport values near the agreed money limit. | Invoice IDs/display references satisfy the approved uniqueness rule; no client count allocation collision. Exact paise transport rejects unsafe/fractional/out-of-range values rather than silently rounding. |

## Clinical draft and invoice issuance

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-049 | Build a valid draft and open clinical review before issuing. | Review shows patient, complaint/category and performed bundled services only. No prices, payable total or automatic examination/return fee appears before Create Invoice returns an issued invoice. |
| AC-050 | Select the same bundled service for two complaints and multiple bundled services for one complaint. | Clinical records retain each correct attribution. Ordinary therapy charges once per treated complaint, with no separate charge for those bundled procedures. |
| AC-051 | Edit/back through a draft and cancel at review; compare database records before/after. | No visit, fee event, invoice, line or receipt is saved. Previously registered patient/courses remain according to D09; selections do not constitute recorded care. |
| AC-052 | Issue a first-attendance checkout for one Routine and one Rehab course using example rates. | Two visits with correct bundled services, one ₹350 fee event, one invoice, therapy lines ₹200/₹300 and one fee line: ₹850 billed. No payment exists yet. |
| AC-053 | Remove required Routine/Rehab price configuration before issuance. | The whole checkout fails with a configuration error. No guessed price, zero-charge care, orphan visit or partial invoice is committed. |
| AC-054 | Bypass client validation with a duplicate course entry, missing required service/clinical field, unsupported operation, or foreign ID. | Backend rejects the invalid contract without partial effects. Valid empty-service behavior, if explicitly allowed, is tested separately and never simulated by a fake service. |
| AC-055 | Force a meaningful error after visits/fees are tentatively inserted but before final invoice completion. | Transaction rollback leaves no new visit, bundled row, fee, invoice, line or success receipt for that operation. Existing patient/course data remains. The draft can recover after a proven failure. |
| AC-056 | Successfully issue an invoice and stop before payment; reload and retrieve it. | Care and invoice persist together as UNPAID with positive due. No transaction waits for human tender selection and no receipt is fabricated. |
| AC-057 | Close an issued invoice without settling; reopen it from saved history and press Pay Later. | Same invoice and care remain; no new invoice or zero payment is created. Closing/deferment is distinct from canceling an unissued draft. |
| AC-058 | After a successful checkout, begin a genuinely separate same-day treatment on the same course. | The agreed repeat notice does not ban a genuine new visit/invoice with a new operation identity. No blanket patient/day or course/day uniqueness applies; reusing the earlier source occurrence is still rejected. |

## Patient-wide examination/return fees

All amounts below use one ₹200 Routine therapy line. Verify fee-event count as well as invoice total; ₹0 return fee means no fictitious positive collection or extra charge.

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-059 | Patient has no prior qualifying attendance; issue one Routine visit. | First-attendance fee ₹350, therapy ₹200, invoice ₹550; one fee event. Current checkout must not be counted as prior attendance and suppress its own fee. |
| AC-060 | Prior qualifying attendance occurred earlier on the same Asia/Kolkata date. | Return fee ₹0; invoice ₹200. A separate legitimate visit is allowed and does not receive another first-attendance fee. |
| AC-061 | Run separate fixtures with prior attendance 1 and 9 calendar days ago. | Both have ₹0 return fee and ₹200 invoice. Verify the inclusive upper boundary of the free-return interval. |
| AC-062 | Prior attendance is exactly 10 calendar days ago. | Return fee ₹150 and invoice ₹350. The boundary is not treated as day 9 by elapsed-hour or browser-time arithmetic. |
| AC-063 | Prior attendance is exactly 29 calendar days ago. | Return fee ₹150 and invoice ₹350; this is still the middle interval. |
| AC-064 | Prior attendance is exactly 30 calendar days ago. | Return fee ₹350 and invoice ₹550. The original first/return policy is not replaced by old ₹300/₹200 mock rules. |
| AC-065 | Prior attendance is more than 30 days ago; also exercise a month/year/leap-day boundary. | Correct calendar difference and ₹350 return fee where applicable; no approximate month-length or elapsed-24-hour shortcut. |
| AC-066 | Patient last attended for complaint A nine days ago and now receives care for complaint B. | Patient-wide recency applies: no return fee on B merely because B has no prior visit. Category and complaint selection do not reset the patient history. |
| AC-067 | The latest qualifying record is a therapy-only visit with no fee, later than the last fee-bearing visit. | Recency uses the later therapy attendance, not only fee events/invoices. Expected interval and amount match that date. |
| AC-068 | The latest qualifying record is an independent fee-only event prepared in isolated fixtures. | It resets recency even without a dummy clinical visit. Fee-only creation UI remains deferred; the query still honors existing qualifying data. |
| AC-069 | Last care was 40 days ago; an old invoice was paid yesterday, with no new care/fee attendance. | Recent payment does not reset attendance. New Routine care bills ₹200 + ₹350, not a free return. |
| AC-070 | Compare attendance at 6 November 23:59 IST with current time 16 November 00:01 IST; vary browser timezone. | Difference is 10 local calendar dates, so return fee ₹150 despite less than ten elapsed 24-hour periods. Browser timezone and UTC date must not change the result. |

## Replay identity and uncertain outcomes

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-071 | Repeat registration and course-batch requests with the same key and same approved canonical payload, including concurrent sends. | One patient/batch result and one set of courses per operation. Every retry returns the original stable identities; it does not rely solely on a disabled button. |
| AC-072 | Repeat a successful checkout with the same key/payload, including after a lost response. | One set of care/fee/invoice/lines. Response/recovery identifies the original invoice; no duplicated first fee or second invoice is created. |
| AC-073 | Repeat a successful payment with the same key/payload, including after response loss. | Exactly one receipt and original result identity. Current invoice state is re-read as needed; the same money is not recorded twice. |
| AC-074 | Reuse a key with materially changed patient, services, amount or tender; test each applicable operation. | Explicit conflict with no additional effects. Canonicalization handles irrelevant object-key ordering consistently but must not erase meaningful payload differences. |
| AC-075 | Request operation results with the wrong caller/clinic/operation scope; replay an issuance after the invoice was subsequently paid. | Unauthorized lookup reveals no result. Authorized recovery finds the correct operation and displays current invoice state, not stale UNPAID data from the issuance snapshot. |
| AC-076 | Drop a response while the server may still be running; an immediate lookup returns no result; reload the tab. | UI preserves/resolves the same frozen operation identity and does not infer rollback from absence. Safe same-key retry/lookup eventually confirms success or a proven failure; no replacement key duplicates the action. |

## Full payment and Pay Later

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-077 | Record full Cash receipt against an issued ₹850 invoice. | One 85,000-paise Cash receipt, backend actor/time, ₹0 due and PAID. Invoice lines and original billed total remain unchanged. |
| AC-078 | Record full UPI receipt against an issued ₹550 invoice. | One 55,000-paise UPI receipt and PAID. UI accurately records the tender; it does not claim to contact or verify a payment gateway. |
| AC-079 | Choose Pay Later immediately after issuance. | Invoice stays UNPAID with full positive due; zero receipt rows. Closing/navigating does not undo care or issue a second invoice. |
| AC-080 | Sign out, return later, open that unpaid invoice and record full payment. | Same invoice is settled; receipt uses actual server receipt date/time and correct actor. Invoice date and original lines are unchanged. |
| AC-081 | Submit zero, negative, fractional-paise, unsafe/out-of-range, and malformed received amounts directly. | Invalid amounts are rejected before effects. No rounded or zero receipt is recorded and no incorrect state transition occurs. |
| AC-082 | Submit less than full due, more than due, CARD, split tender, waiver or refund through this demo API. | Clear unsupported/invalid response and no mutation. The explanation distinguishes a demo limitation from the full clinic's approved partial/split/waiver/refund scope. |
| AC-083 | Keep an old payment screen open after another session settles the invoice; submit its old full amount with a different key. | Current due is checked under the transaction's lock. No second receipt/overcollection; UI refreshes to the settled invoice with a useful result/conflict. |
| AC-084 | Fail settlement after tentative receipt insertion but before transaction completion. | No receipt persists and the issued invoice remains UNPAID with its original due. Care/invoice issuance is not rolled back by this separate payment failure. |
| AC-085 | Cancel payment selection, close the invoice, or use browser Back before confirming receipt. | No receipt is recorded from dismissal/navigation. Issued care/invoice remain retrievable and unpaid. |
| AC-086 | Attempt to alter a receipt's tender/amount/actor/time or delete it after confirmation. | Application permissions/constraints deny mutation. Current invoice/receipt history remains intact and no UI offers a generic edit/delete escape. |
| AC-087 | Forge PAID, received_at, recorded_by or a different owner while sending otherwise valid payment inputs. | Server-derived identity/time/state wins according to strict contract handling. A client status flag alone never settles an invoice. |
| AC-088 | Delay or lose a payment response after commit, then retry from the recovery UI. | Explicit unknown state until confirmed; one receipt after recovery. Payment controls cannot start a fresh conflicting operation while the first remains unresolved. |
| AC-089 | Start a subsequent genuine treatment for a patient who still owes an earlier invoice. | Ordinary care is allowed; earlier due remains on its own invoice. No debt check silently blocks clinical recording or transfers old debt into a new line. |

## Saved history and accurate read models

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-090 | Reload a patient profile with saved courses, multiple visits, invoices and receipts, including tied timestamps. | Every record belongs to that patient, uses saved relationships and has deterministic ordering. No mock lookup, random failure generator or title-based cross-link remains in the supported read path. |
| AC-091 | Read daily activity around local midnight with two visits in one checkout and a later receipt. | Rows/date filters follow the approved unit and Asia/Kolkata boundary. Labels distinguish visits, checkouts, invoices and receipts; old Waiting/In Therapy badges do not invent a saved queue. |
| AC-092 | Prepare an isolated read-model fixture with invoice lines ₹200 + ₹300 + ₹350 and receipts ₹200 + ₹100. | Billed ₹850, collected ₹300, due ₹550, without line×receipt multiplication. This read fixture exercises future-compatible aggregation; demo UI still cannot create partial payments. |
| AC-093 | Fail a history/activity read after previously successful data, then retry. | Error is visible and stale data, if retained, is labeled. Failure cannot appear as a successful empty history or zero outstanding balance. Retry uses current patient/session. |
| AC-094 | View a two-complaint checkout from each complaint history and from patient billing. | Correct visits/services are attributed to each complaint; one shared invoice is not counted twice in the patient's billed total. Navigation targets the saved invoice identity. |
| AC-095 | Inspect all visible dashboard/profile/ledger values against saved rows and the agreed scope. | No random revenue, hard-coded referral statistics, fake health metrics, unsupported package expiry, or mock patient history is presented as live. Any displayed count has a defined unit/query. |

## UI states, navigation, and recovery

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-096 | Submit each form with validation failure, slow success, and confirmed server failure. | Relevant values stay available while pending/failed; field errors are associated and actionable; controls recover. Reset/advance occurs only after confirmed success or deliberate cancellation. |
| AC-097 | Use edit/back/forward within a treatment draft and open a later workflow step directly without context. | Valid selections survive approved intra-draft navigation. Missing context leads to a safe entry/fetch path, not a mock patient, crash or auto-submit on mount. |
| AC-098 | Change patient after selecting complaints/procedures and after one read is still pending. | The approved confirmation/clear behavior prevents mixed-patient drafts. Old course IDs, errors and delayed responses cannot contaminate the new patient. |
| AC-099 | Refresh an ordinary draft and, separately, refresh while a durable write is uncertain. | Behavior matches the documented D11 choice: ordinary unsubmitted data may be lost if stated; frozen uncertain-operation identity is recovered safely. The UI distinguishes these cases. |
| AC-100 | Paste valid, nonexistent and unauthorized patient/invoice URLs; add an invalid or another patient's complaint filter. | Authorized records load; missing/forbidden records reveal no unrelated detail. Invalid filters use the approved corrected default, not false empty history. Routes do not depend on earlier in-memory mock context. |
| AC-101 | Switch accounts/unmount screens while search, profile and course responses are delayed. | No obsolete response updates the new account/patient. Loading/error state remains tied to the active request identity rather than whichever promise settles last. |
| AC-102 | Exercise auth callback, session restoration, expiry and sign-out on supported routes. | Correct pending/redirect behavior, no protected-content flash, no redirect loop, and no navigation that repeatedly resubmits an earlier write. |
| AC-103 | Issue, choose Pay Later, navigate away, settle later, then revisit all affected screens. | Invoice, patient history and activity reflect the latest saved state. Original invoice amount is stable; receipt/due/state update without duplicate local rows. |
| AC-104 | Trigger each approved validation/conflict/auth/configuration/network/version error category. | Messages explain the next valid action and preserve relevant input. No raw SQL/secret/unrelated patient content appears; error is not swallowed or converted to success. |
| AC-105 | Use two tabs with stale invoice/history data; settle in one and attempt interaction in the other. | No optimistic duplicate success. The stale tab re-reads/reconciles server state and presents a useful outcome; a client-side cached due never authorizes collection. |

## Deployment and reproducibility

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-106 | Follow the documented setup on the authorized target and verify the deployed schema/API/frontend versions. | Known compatible versions are actually running; migration success and policy presence are verified separately from frontend build/deploy status. Record the exact target without credentials. |
| AC-107 | Inspect deployed browser assets/configuration and controlled account access. | No privileged secret in the client or published demo instructions; no public self-enrollment as admin, public reset endpoint, or reused shared admin password exposed to reviewers. |
| AC-108 | Load deployed patient/invoice deep links directly, refresh, complete login callback and reopen a saved record. | Hosting routes and allowed redirects work on the real domain. No hosting 404 or context-dependent wrong-patient view occurs. |
| AC-109 | Keep an old browser tab open through the approved rollout and test the compatibility/recovery plan. | Old requests either remain valid under the chosen contract or receive a clear safe reload/error; no malformed financial write. Record what frontend redeploy can recover and what needs a separate schema/data plan. |
| AC-110 | Review and, only with specific authority, exercise the isolated fixture restoration/operator procedure. | Resolved target is checked, valuable/other-environment data is preserved, and access is operator-only. Record whether restoration was actually run or merely reviewed; no destructive test is implied by this checklist. |

## Human ownership and project claims

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-111 | Om traces his patient-selection-to-course-loading path using the running app, network response and code. | He identifies state owner, saved IDs, async states and stale-response protection in his own words. Record what was Om-written, paired or AI-assisted rather than requiring memorized syntax. |
| AC-112 | Agree one small change in that path; Om implements/verifies it without a complete supplied patch. | The change behaves correctly and Om explains why. Hints/pairing are recorded; if extensive help is needed, do a smaller independent change rather than falsely recording independence. |
| AC-113 | Present one bounded failure in an isolated learning setup, such as an old patient response replacing a new selection. | Om uses observed state/network/code to locate the cause, applies an understood correction and verifies it. Remove the introduced failure from the release; guessing or repeating AI's answer is insufficient evidence. |
| AC-114 | Ask Om to predict normal/failed/replayed checkout, concurrent payment and unauthorized-call outcomes. | He correctly distinguishes atomicity, idempotency, source uniqueness, locking, and backend authorization using actual project evidence. Any missing critical concept gets a focused paired review. |
| AC-115 | Review task provenance and draft project talking points against the implementation/evidence. | Claims accurately describe a synthetic integrated demo, current scope, Om's contribution and AI assistance. No fabricated production adoption, independent authorship, performance result or unimplemented clinic feature. |
| AC-116 | Follow the code map/setup/walkthrough and inspect remaining gaps. | Another authorized reviewer can locate supported paths and limitations. Remaining noncritical understanding gaps have a small learning backlog; no claim of mastering the entire framework/database is required. |

## Concurrency, rollback, and transport faults

Use actual concurrent database/request sessions and an isolated fault harness. Sequential button clicks or mocked frontend responses alone do not establish these outcomes. Do not expose fault injection or an adjustable production clock to ordinary callers.

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-117 | Send two simultaneous identical checkout requests with the same operation key. | One committed invoice/care set; both callers resolve to that result or a documented pending-then-recovery path. No double first fee, dead unresolved receipt or orphan rows. |
| AC-118 | Send two different-key, genuinely separate first-attendance Routine checkouts for one new patient concurrently. | Both real visits are allowed, but serialized recency yields one ₹550 checkout and one ₹200 checkout: ₹750 total, one ₹350 fee. Order is nondeterministic; double first fees are not acceptable. |
| AC-119 | Race attempts to bill the same actual source occurrence in separate invoice operations using the approved test boundary. | At most one valid linkage commits. Constraint/transaction failure rolls the loser back without losing the winner or permitting an inconsistent invoice. |
| AC-120 | Induce the supported retryable transaction failure/lock contention in isolation; retry according to the contract. | No partial effects, inconsistent lock order or replacement operation identity. A safe retry resolves once; persistent failure remains visible instead of looping indefinitely. |
| AC-121 | Inject failure at different checkout stages, including line creation and operation-result persistence. | Effects and successful replay record commit together or neither does. A saved success marker without an invoice, or an invoice without recoverable operation identity, fails this scenario. |
| AC-122 | Race two different payment keys for the full due on one UNPAID invoice. | Exactly one actual receipt and PAID with zero due; the other cannot overcollect. Record both responses and final receipt sum, not only the UI state. |
| AC-123 | Race two identical-key payment requests and separately reuse the key with a changed tender/amount. | Identical requests resolve to one receipt; materially different reuse conflicts without extra receipt. Locking/replay works across sessions, not just within one browser. |
| AC-124 | Let registration/course/checkout/payment commit, then drop its response at the transport boundary and recover in the browser. | Each operation has exactly one persisted effect/result after same-key recovery. Include reload and delayed lookup; no immediate empty lookup is treated as proof of rollback. |

## End-to-end presentation and usability

| ID | Action / fixture | Required outcome and evidence |
| --- | --- | --- |
| AC-125 | Complete the supported workflow using only the keyboard; trigger field and server errors. | Controls have usable labels, visible focus and appropriate announcements; focus returns sensibly from dialogs/errors. Search Enter selects/submits only the intended action and delete does not also toggle its parent. |
| AC-126 | Render the accepted visual baseline at agreed desktop/narrow widths and both sides of relevant breakpoints, with long names and amounts. | No blocked primary action, clipped invoice data, unusable overflow or hidden focused control. Compare actual rendered results, including any adapted #95/#98 wrappers/styles; build success alone is insufficient. |
| AC-127 | On the deployed demo, register a new synthetic patient, create/select a course, record care, issue and fully pay Cash; reopen in a fresh session. | Complete coherent saved journey with correct example fee/therapy amounts, one receipt, correct history, and no seeded-only shortcuts. Record actual version and resulting IDs in private test evidence. |
| AC-128 | Select an existing returning patient, treat two complaints with multiple bundled services and fully pay UPI. | Saved patient context, course attribution, one applicable patient-level fee, correct flat therapy lines, one full receipt and accurate history. Bundled procedures do not become extra charges. |
| AC-129 | Issue with Pay Later, start another genuine care workflow while debt remains, then return later to settle the original invoice. | Debt does not block care; original invoice remains independently retrievable and can be fully settled once. No patient-level wallet, silent allocation, or transferred debt is invented. |
| AC-130 | Inspect every reachable control, label, metric and demonstration statement, including deferred features. | All supported actions work; unsupported package/partial/split/waiver/refund/notes/standalone/print/dashboard features follow the approved hide/unavailable contract. Synthetic-only limits are visible; no no-op success, real-patient/package case, fake queue or unverified production claim slips into the demonstration. |

## Coverage and closeout

The 130 scenarios are observable checks, not a requirement to write 130 separate automated tests. One meaningful test or complete journey can cover several scenarios; a single scenario may require database, browser and human evidence. Record the mapping explicitly so a passing UI screenshot is not used to claim an untested database invariant.

| Area | Task phases | Principal scenarios / evidence |
| --- | --- | --- |
| Baseline, scope, contracts | P00–P02 | EVIDENCE refresh, D01–D25 selection record, approved field/API/state/privilege contracts |
| Schema, environment and authorization | P03–P04 | AC-001–AC-014, AC-040–AC-048 |
| Search and patient persistence | P05–P06 | AC-015–AC-029, AC-071, AC-074–AC-076 |
| Human integration and course persistence | P07–P08 | AC-027–AC-039, AC-097–AC-101, AC-111–AC-113 |
| Draft and issuance | P09–P11 | AC-040–AC-076, AC-117–AC-121 |
| Settlement and read models | P12–P13 | AC-071–AC-095, AC-122–AC-124 |
| Failure and system verification | P14–P15 | AC-096–AC-105, AC-117–AC-130; actual results across supported earlier scenarios |
| Deployment and demonstration | P16 | AC-106–AC-110 and deployed AC-125–AC-130, critical authorization/financial smoke results |
| Learning and handoff | P17–P18 | AC-111–AC-116, final task/decision/result/provenance reconciliation |

Release is ready to demonstrate only when applicable supported-path checks have actual passing evidence, blocked items are resolved, and Om has completed the agreed human exercises. A decision that a scenario is genuinely inapplicable must state why and identify the approved scope/architecture choice; it cannot disguise a known supported-path failure. Real-clinic operational acceptance remains separate F24 work.
