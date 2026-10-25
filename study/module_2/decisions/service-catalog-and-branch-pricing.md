# Service catalog and branch pricing

Current discussion view reviewed against the supplied October 25 scope/payment records. **O06 remains open**; no new catalog architecture was selected there. Provisional names and preferred alternatives are not approved schema.

## Current position

- A definition, branch offering, current availability, branch price, performed occurrence and issued charge are different facts.
- **Settled:** reception sees only available services for its authorized branch; the backend must independently validate eligibility and authorization.
- **Settled:** configured branch prices are independent. Later global reference-price changes must not silently overwrite them. Disabled offerings must preserve clinical and invoice history.
- **Owner-reported update:** one branch operates today; the older three-operating-branch assumption is outdated. Future expansion remains intended. This is not verification of live branch records or permission to delete historical data.
- **V1 allowance:** rare branch provisioning may be manual. Bulk disable-all automation is optional; central safety lockout, promotional scheduling and administrative price history are not established requirements.
- Ordinary therapy is now flat-only in the proposed model; separately purchased services remain relevant to branch pricing. See [clinical records](clinical-records.md).

## Alternatives and trade-offs

| Alternative | Meaning of a missing branch row | Main trade-off |
| --- | --- | --- |
| Sparse overrides over global defaults | Inherit | Few rows, but unconfigured services may become available accidentally; default precedence needs explicit approval. |
| Every service configured for every branch | Missing configuration is exceptional | Explicit price/status, but provisioning must maintain all pairs, including irrelevant offerings. |
| Explicit branch enrollment | Not offered | Clear opt-in and no silent fallback, but adding an offering requires configuration rather than merely activation. |
| Fully branch-owned definitions | No shared service identity | Fewer shared concepts, but duplicated definitions and cross-branch comparison/mapping questions. |

The first three alternatives share conceptual service identities; the fourth does not. Price and availability in one configuration table is provisionally favored, not settled. `clinic_services` is only a provisional name.

## Open questions

- **O06-A/B:** need for shared identities, ownership and the meaning of branch configuration; one price/availability table versus separate tables.
- **O06-C/D:** branch onboarding, new-service propagation, missing rows, global defaults, temporary suspension and retirement.
- **O06-E/F:** administration and reception paths, valid zero/missing prices, price resolution and concurrent changes.
- **O06-G/O12:** standalone/specialty classification, tier assignment, negotiated rates and fee ownership.
- Runtime global-price fallback is not the accepted current path. Selecting an override model would require explicitly revisiting that choice.

## Scenarios and sources

[O06 brief](history/O06_service_catalog_branch_configuration_decision_brief_2026-10-23.md), §§5–8, retains the complete alternatives and scenarios: different equipment, branch opening, a new service offered at one branch, temporary disablement, stopping everywhere, price changes and selection/billing paths. Its per-procedure scenario is historical after the successor update.

## Changes in position

- [October 21 addendum](history/clinic_billing_decision_addendum_2026-10-21.md) called the unified catalog settled. [October 22 log](history/clinic-billing-decision-log-2026-10-22.md), §3, reopened it.
- [Parent register](history/clinic_billing_decision_log_2026-10-23.md), §§2.1 and 13: separate definitions preferred, not independently ratified; all-inactive new-branch setup initially settled.
- O06 brief §4 explicitly **reopens all-inactive provisioning**. Do not carry the parent's settled label forward.
- [Successor](history/clinic_module2_successor_addendum_2026-10-23_1726_IST.md), §§3, 11.1 and 12, updates operating branches and ordinary flat-only care while leaving O06 open.
- [October 25 scope continuation](history/clinic_module2_scope_decision_addendum_2026-10-25.md), §4, preserves the independent catalog workstream. Payment/voiding scope does not settle service identity, enrollment, branch provisioning or price/availability tables. [October 25 audit](history/clinic_module2_audit_gap_appendix_2026-10-25_184849_IST.md), G03, records a charge-source classification conflict, not approval to change the catalog model.
