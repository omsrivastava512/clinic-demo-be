# Clinic demo — delivery and learning state

**Updated:** 16 November 2026. **Workflow:** `$delivery-learning-cycle`.

Installed skill: [SKILL.md](C:/Users/HP/.codex/skills/delivery-learning-cycle/SKILL.md). If a session has not discovered the skill by name, explicitly ask it to read that file and resume this record.

This is the active progress record, separate from the proposed plan and its unchecked execution tasks. The user's green light authorized creating the reusable workflow and this record. It did not approve D01–D25, an implementation slice, code changes, migrations, deployment, or Git mutations.

## Current checkpoint

| Field | Value |
| --- | --- |
| Plan | C:/Github/clinic-demo-be/plans/demonstrable-demo/README.md |
| Slice | Not selected |
| Task IDs | Not assigned; first preparation session should consult P00–P02 / G0 |
| State | PREPARE |
| Status | NOT_STARTED |
| Pending human action | Start the first PREPARE session; no implementation approval is pending yet |
| Next transition | Remain PREPARE until a bounded slice and human contribution have been discussed |
| Approval needed to build | Explicit agreement on that slice's scope, decisions, ownership, file boundaries and transition to BUILD |

## Scope and ownership

- **Approved now:** reusable skill setup and persistent workflow record only.
- **Implementation scope/files:** none approved under this workflow yet.
- **AI-owned work:** to be specified in the first slice proposal.
- **Om-owned work:** to be specified before building. The plan reserves the representative patient-selection-to-complaint-loading integration for Om; it is not being started here.
- **Paired checkpoints:** to be specified per slice, especially permissions, transaction boundaries, retries and payment behavior.
- **Learning objective:** to be made concrete for the first slice; no understanding assessment has occurred.
- **Decisions:** D01–D25 remain proposed except where they describe existing policy constraints. Workflow approval is not approval of their implementation recommendations.
- **Excluded:** clinic implementation, migration/deployment, Git operations, PR/issue changes and unrelated portfolio edits during setup.

## Progress and evidence

| Track | Current result | Evidence |
| --- | --- | --- |
| Planning package | Created in the earlier scoping task | README, EVIDENCE, DECISIONS, TASKS, ACCEPTANCE |
| Workflow setup | Skill installed and this record initialized; not a completed delivery slice | User's “sounds perfect. green light.” authorized setup; installed files hash-match the reviewed source |
| Demo implementation | NOT STARTED under this plan | Execution checklist remains unchecked |
| Application acceptance | NOT RUN | ACCEPTANCE contains future scenarios, not runtime results |
| Human learning exercises | NOT STARTED | No trace/change/debug exercise claimed complete |

Setup validation: frontmatter, UI metadata, state coverage, initial state and whitespace checks passed. The bundled Python validator could not execute because the configured Python launcher is inaccessible. Transition rules were manually reviewed for questions versus approval, pending human work, verification returns and resume behavior; a real delivery-cycle trial remains pending. This skill provides instructions and a progress record, not mechanically enforced tool permissions.

## Repository checkpoint and limits

- Backend local HEAD and live origin/main verified at `33f6b38d812ae3530bb827e4a331e1de3b7739df` during setup. Existing unrelated modified/untracked work remains in place.
- Portfolio local HEAD remains `4c30ecd`; its newer uncommitted `site.config.ts` bio changes are user work and outside this setup. No new claim about portfolio remote divergence is made here.
- Existing clinic plan and upcoming transcript remain present. This setup does not reconcile clinic policy or replace EVIDENCE's source authority.
- Frontend Git cleanliness and deployed database remain unverified. Refresh applicable repository/remote evidence before proposing implementation; this checkpoint is not execution authorization.

## Session protocol

Read this record before resuming. Announce slice/state/status and the pending human action. Execute only the current state; completing it does not authorize the next. Record the checkpoint and end the turn while awaiting the human. Natural-language approval of a clear pending transition is sufficient; questions, praise and silence are not approval.

On resume, read the [18 November ticket follow-up](EVIDENCE.md#18-november-2026-ticket-follow-up) and check the current task and acceptance criteria before continuing the selected slice.

Record implementation and learning progress separately. Re-read before updates to avoid overwriting another session's newer work. Preserve prior checkpoints and decision provenance; do not store secrets or fabricate evidence.

## Checkpoint history

| Date | Event | Authorization / outcome |
| --- | --- | --- |
| 2026-11-16 | Workflow setup initialized | User approved the proposed skill plus state record. PREPARE / NOT_STARTED; no delivery slice selected or implemented. |
