# Updating Module 2 decision records

These instructions apply to work in this directory. When a user supplies new logs for incorporation, use the requested sources and scope; do not rely on a previous chat being available. Read this file, README.md, UNRESOLVED_AND_AUDIT_REGISTER.md, the relevant topic pages, and reconciliation-2026-10-27.md before making changes. Inspect the existing workbook when its tracked decisions or issues are affected. Follow higher-priority instructions and the user's current directions.

## Authority and evidence

- The six topic pages and the reconciliation decision register are the working decision records. The tracker records current interpretations and outstanding work alongside historical attribution. Read their present contents rather than assuming their current cutoff or status.
- **Business Scope Status:** As of 5 November 2026 (incorporation of S34–S37), all commercial/business questions in Module 2 are 100% locked (`User-review rows = 0` in the tracker). No package, fee, pricing, or payment rules are open for business redesign.
- **Master Unresolved & Audit Register:** All remaining open items are strictly categorized in [`UNRESOLVED_AND_AUDIT_REGISTER.md`](UNRESOLVED_AND_AUDIT_REGISTER.md):
  1. *Category A:* 10 Technical Implementation Audits (`A01`–`A10`, `N51`: DDL, composite keys, row locking, idempotency, safe views).
  2. *Category B:* 4 Legacy Schema Dispositions (`N09`–`N11`, `N23`: vitals, alerts, timeline events).
  3. *Category C:* 2 Parked Minor Behaviors (`A05`, `A13`: complaint inactivity, UI deselection).
  4. *Category D:* 8 Post-MVP Deferrals (`R03`, `R06`, `O01`, `N01`–`N08`, wallets, etc.).
- history/ preserves evidence and provenance; it is not a competing set of operative policies. Historical logs may contain gaps, superseded statements, disagreements and assistant proposals. Do not rewrite history to make it consistent, or treat gaps as errors to fill in.
- Distinguish explicit user decisions, reported clinic-owner positions, proposals, assumptions and unresolved questions. A later date or an assistant suggestion alone is not approval. Apply explicit corrections only to the subjects they address. If a conflict cannot be resolved from the evidence, record it as open and ask a focused question when an answer is needed.
- Respect the user's selected sources. Do not incorporate excluded material. If a known later source is excluded or unavailable and may affect a topic, clearly mark the source cutoff and the limitation. Do not describe a cutoff snapshot as verified latest policy.
- Business decisions are not proof that application behavior, database schema, deployment, enforcement or tests exist. Technical implementation audits must be resolved in DDL, migrations, and code.

## Incorporating logs & resolving audits

1. Locate and read the complete requested logs or technical specs, including appendices and timestamps. If a required source is missing, make only independent progress and identify the missing source. Check whether it or the same content is already recorded before adding it again.
2. Compare each substantive statement with the existing topic pages, register, UNRESOLVED_AND_AUDIT_REGISTER.md, and relevant sources. Identify decisions, clarifications, changes, conflicts and open questions, with citations to the supporting source and location. Preserve source-specific meaning and uncertainty; do not invent decisions.
3. Preserve original source files byte-for-byte when adding them to history/, unless the user directs otherwise. Do not overwrite an existing archive. Follow the existing source-ID, link and hash conventions in README.md and the reconciliation register (assigning S38 onwards for future incoming sources). Verify archive copies against their originals.
4. Update the affected current topic pages, UNRESOLVED_AND_AUDIT_REGISTER.md, and the reconciliation register so the operative interpretation and unresolved issues agree. Update the reading guide, source inventory, links and cutoff notes as needed. Change only what the new evidence supports; maintain historical attribution and label old coverage reports as snapshots when appropriate.
5. If affected, update the existing tracker in place: preserve its filename, tabs, IDs, historical attribution, layout and formulas; align current interpretation, sources and issue states with the decision records. Preserve the distinction between proposed and executed acceptance checks. Do not claim a check was run unless it was actually run.
6. Verify source hashes, references and links, consistency across changed records, and relevant workbook structure, formulas and cached values where possible. Run model checks (`node audit-2026-10-24/model-checks.cjs`). Report any verification that could not be performed. Inspect the final diff and preserve unrelated work. Do not alter code, schema or deployment solely because a log records a decision.

## Handover

Explain what was incorporated or audited, the meaningful changes, unresolved decisions and conflicts, the source cutoff, the files changed, and what was verified or remains unverified. Make clear whether this was documentation work or included separately authorized implementation. Do not stage, commit or push unless specifically authorized.

Agent instruction discovery varies by tool: a future agent must have this repository available and read the applicable AGENTS.md for these instructions to take effect.
