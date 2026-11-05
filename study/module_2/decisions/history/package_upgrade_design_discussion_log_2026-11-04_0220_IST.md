# Package Upgrade — Discussion, Decision & Database Design Log

**Recorded:** 4 November 2026, 02:20 IST (UTC+05:30)  
**Scope:** Only the discussion of upgrading a **10-visit package to a 26-visit package**, its accounting, eligibility, lifecycle and proposed database schema, following the separate transcript/questionnaire discussion log. This is **not** a replacement for that log or the general Package Module specification.  
**Status:** Feature direction accepted in principle; **detailed design not yet fully approved or implemented**.  
**Decision standard:** Explicit user choices outrank assistant proposals. Unresolved choices are not silently resolved by examples or suggested SQL.

**Labels:** **CONFIRMED** = explicitly decided; **DIRECTION** = intended approach, subject to detailed design/complexity; **PROPOSAL** = assistant design recommendation, not separately approved; **OPEN** = requires a decision; **REJECTED** = not the intended feature.

## 1. Why the feature was raised, and scope

- **CONFIRMED — Real request:** A patient who bought a 10-visit package may ask to pay the difference and upgrade to the corresponding 26-visit package. The originally paid/charged amount should be economically recognized; the patient is not requesting an additional independent 26 visits.
- **CONFIRMED — Eligible directions:** **Routine 10 → Routine 26**, or **Rehabilitation 10 → Rehabilitation 26**, only for the **same complaint and treatment category**. There is no upgrade above 26, no cross-category upgrade, and no Routine↔Rehabilitation package conversion.
- **CONFIRMED — No formal downgrade:** A longer package is not edited into a shorter one. If the clinic settles a partly paid longer package using shorter-package and ordinary-visit economics, it handles the outcome through explicit package closure, invoices, payments, refunds and waivers as applicable; that is not a package conversion feature.
- **DIRECTION — MVP:** The user favors including this bounded upgrade if implementation is reasonably contained; it is **not unconditional approval irrespective of complexity**.
- **CONFIRMED — Four regular offerings remain:** Routine 10, Routine 26, Rehabilitation 10, Rehabilitation 26. An upgrade must **not** become a fifth freely purchasable offering.

## 2. Approaches considered

| Approach | Evaluation / disposition |
|---|---|
| **Edit original package** from 10 to 26 visits | Considered. Appears convenient for active packages but changes an already purchased entitlement and has implications for immutable/terminal original records. **Not chosen.** |
| **Sell a fresh full 26-visit package and waive/adjust its invoice** | **REJECTED as an equivalent upgrade:** grants a fresh 26 visits instead of only 16 additional visits; changes review timing. A waiver forgives an invoice balance, **not** transfers a previous payment. |
| **Standalone fifth “16-visit upgrade” catalog product** | **REJECTED in unrestricted form:** could be bought without a parent and lacks built-in eligibility and original-date linkage. |
| **Linked Upgrade Package / dependent extension** | **PREFERRED DIRECTION:** a second package record, linked to the specific original 10-visit record, contributes 16 visits. The parent remains historically intact. The upgrade is initiated from an eligible parent, never sold independently. |

The assistant described this as **moderate, bounded engineering complexity**, not a trivial catalog addition and not a general-purpose package conversion engine. No code or migration was applied during discussion.

## 3. Eligibility, timing and entitlement

- **CONFIRMED — Early purchase allowed:** The patient may buy the upgrade **before using all 10 original visits**. The suggested implementation places the upgrade in a **Queued** state until the original package's 10 covered visits are exhausted; then the extra 16 become eligible. Queue behavior is the **proposed implementation** of that confirmed business intent.
- **CONFIRMED — Upgrade after original exhaustion:** The user gave examples of buying the extension after the short package finishes, during the original longer review window. The original **Completed** record must never be reactivated; the linked extension is a distinct record.
- **CONFIRMED — Upgrade window:** No upgrades months afterward. The intended latest opportunity is the **30th calendar date counted inclusively from the original short package's coverage start**. An upgrade on a late day does not restart the clock. The user considered a three-non-Sunday-day grace window and then preferred the original **30-calendar-day window**.
- **CONFIRMED — Date anchor:** **The original 10-visit package's coverage start date**, not the upgrade purchase date. This was explicitly answered after the GitHub/schema discussion. The 26-visit review threshold is Day 30 (calendar days including Sundays, start date = Day 1).
- **CONFIRMED — Final quantity:** **26 total covered visits across the linked chain**: first package contributes up to 10, upgrade contributes **16 more**, not 10 + 26. Already consumed original visits remain counted.
- **CONFIRMED — No hard date expiry:** Day 30 is the **review milestone**, not a forfeiture date. Remaining eligible allowance may continue after review under the established post-review clinic-discretion/warning rules; late purchase does not extend or restart review.
- **NOT A RULE — “Extra 20-day cycle”:** The user initially described a 16-visit / approximately 20-day extension. The assistant clarified that there is **no fixed new 20-day period**: the 10-visit review excludes subsequent Sundays, the longer review counts 30 consecutive calendar dates, and upgrade timing varies. The upgrade has its **own 16-visit allowance and lifecycle**, but **inherits the original Day-30 review anchor**.
- **CONFIRMED — Authorization:** Both **admin and receptionist** can initiate an upgrade, subject to backend eligibility validation.
- **OPEN — Manually Closed parent:** The user was explicitly **not sure** whether an originally **Closed** (rather than normally exhausted/Completed) 10-visit package should be upgradeable. The assistant recommends **no**, but that restriction is **not approved**.

### Date example discussed

Take the original coverage start as **1 November 2026**. Its short-package Sunday-excluded review example reaches approximately **11 November**, depending on the agreed short-counting convention; the proposed upgraded **long review date is 30 November 2026** (Day 30). The user discussed purchases as late as 20 November or 30 November: they do **not** shift the long review date. Any allowance remaining after 30 November is subject to post-review rules, not automatic expiry.

### Already invoiced visits between exhaustion and a later upgrade

An assistant example: original 10 visits are exhausted, subsequent ordinary visits on **15 and 18 November** are invoiced, and upgrade is bought **20 November**. The assistant recommended that these **already committed ordinary visit invoices remain unchanged**, and upgrade coverage is available only for eligible visit transactions from purchase/activation onward. The user responded **“Yes,” but also said the distinction was unclear**. Treat the principle of **no retroactive invoice rewriting** as the established general accounting constraint, but **explicit confirmation of this precise upgrade-gap scenario remains prudent**. Backdating a review-clock anchor is not permission to reprice historical invoices.

## 4. Financial design and examples

- **CONFIRMED — Catalog prices are configurable:** Numeric examples are illustrative, not fixed product prices. The later/current configured 26-visit price was proposed as the upgrade's target price.
- **CONFIRMED — No negative upgrade charge:** The user expressly directed that a negative price difference be **floored to ₹0**, not turned into a refund or negative invoice line.
- **WORKING PRICING FORMULA (assistant-recommended; basis needs explicit final sign-off):**

  `upgrade_charge = max(0, current_price_of_26_visit_offering − original_10_visit_invoiced_charge)`

  The user initially described subtracting “what the patient already paid.” The assistant explained that using **cash received** could double-count an unpaid part of the original invoice and proposed subtracting its **original immutable invoiced charge** instead. This was presented as the working/final explanation in the exchange, but the user **did not explicitly restate approval of the billed-versus-paid distinction**. Preserve this provenance instead of treating that nuance as a direct user confirmation.
- **Example:** Original 10-visit invoice ₹1,800; current 26-visit catalog price ₹5,000 → proposed upgrade invoice **₹3,200**. If original invoice was only partly paid, its unpaid balance remains attached to that original invoice; it is not transferred, waived, or charged twice.
- **Floor examples:** Target ₹1,800 and original billed ₹1,800 → ₹0. Target ₹1,500 and original billed ₹1,800 → ₹0. No negative invoice or automatic refund.
- **PROPOSAL — ₹0 invoice compatibility:** The existing checkout design had authorized zero-total invoices for package-covered visits, **not** zero-priced upgrade sales. To support the user-approved zero-floor outcome, the assistant proposed a narrow new permitted ₹0 upgrade-sale invoice/settlement path with **no fictitious payment row**. Its precise schema/checkout exception still needs integration validation.
- **PROPOSAL — Financial records:** Original invoice and payment rows remain immutable; upgrade is a distinct billed purchase with its own invoice line; cash receipts continue to belong to their invoices. Waiver is not a method of transferring money from the old package to the extension.

## 5. Proposed database representation (GitHub-informed, not deployed)

### Existing repository context reviewed

- Backend specification repo: [`clinic-demo-be`](https://github.com/omsrivastava512/clinic-demo-be). Its [older Supabase SQL draft](https://github.com/omsrivastava512/clinic-demo-be/blob/main/docs/schema/supabase_migration.md) defines `packages` with a complaint FK but **no upgrade-parent linkage**. It also contains legacy expiry/payment fields that should **not be carried forward blindly**.
- [Package decisions](https://github.com/omsrivastava512/clinic-demo-be/blob/main/study/module_2/decisions/packages.md) and [package integration/deferral analysis](https://github.com/omsrivastava512/clinic-demo-be/blob/main/study/business_logic/packages_logic/DEFERRAL.md) propose package-specific visit attribution, financial separation and relevant uniqueness constraints; these are **design records, not proof of a deployed implementation**.
- Frontend [`clinic-demo`](https://github.com/omsrivastava512/clinic-demo) is a **mock-data component catalogue**, not a backend-connected production application.

### Preferred structure: two rows in the same `packages` table

```text
packages: P101                         packages: P102
  package_kind = STANDARD                package_kind = UPGRADE
  parent_package_id = NULL               parent_package_id = P101
  linked_complaint_id = C101             linked_complaint_id = C101
  visits_promised = 10                   visits_promised = 16
  status = Completed                     status = Active  [or Queued if early]
  coverage_start_date = 2026-11-01       coverage activation distinct from review anchor
  review_date = short-package date       review_date = 2026-11-30
```

`P101` and `P102` are **illustrative labels**; real primary/foreign keys would be UUIDs. Both belong to the **same patient, clinic, complaint, and treatment category**. An upgrade record cannot exist without an eligible original 10-visit parent.

### Field-by-field proposed `packages` record

| Field | Data type / expected value | Interpretation and status |
|---|---|---|
| `id` | UUID, e.g. illustrative `P101` or `P102` | Unique package/upgrade record identifier. **PROPOSAL**. |
| `clinic_id` | UUID FK, same for parent and upgrade | Clinic ownership/tenancy. Existing schema concept. |
| `patient_id` | UUID FK | Patient receiving entitlement; same in parent and child. |
| `linked_complaint_id` | UUID FK to `complaint_courses` | Coverage applies to exactly this complaint; must match parent. |
| `package_kind` | `STANDARD` or `UPGRADE` | Identifies ordinary purchase versus dependent extension; **PROPOSAL**. |
| `parent_package_id` | Nullable UUID self-FK; null on standard, parent `id` on upgrade | Enforces traceable parent-child link. **Core proposed mechanism**. |
| `visits_promised` | Integer: **10** for short, **26** for normal long, **16** for upgrade row | This row's contributed visit allowance, not the whole chain's total. |
| `coverage_start_date` | Clinic-local `DATE`; may be **NULL while a child is Queued** | When this row is eligible for coverage; not the review anchor if an upgrade. Null-while-queued was an **assistant refinement**. |
| `review_date` | Clinic-local `DATE` | For upgrade, derived from original parent coverage start + inclusive 30-day rule; milestone, not hard expiry. |
| `status` | Discussed values: `Queued`, `Active`, `Completed`, `Closed` | Proposed lifecycle. **Queued** supports early upgrade; **Completed** and **Closed** retain terminal meaning. Not an independently approved migration enumeration. Review/Overdue are date-derived, not terminal statuses. |
| `purchase_date` | Clinic-local `DATE` | Proposed explicit business purchase date distinct from `created_at`; user did not separately approve column. |
| `created_by` | UUID FK to staff/profile | Who registered original or upgrade; admin or receptionist can initiate upgrade. |
| `created_at` | `timestamptz` | Immutable technical creation timestamp. |

**Discussed and removed from preferred sketch: `offering_id`.** The assistant originally suggested values such as `ROUTINE_10` and `ROUTINE_26` as **illustrative product identifiers**, not confirmed real database IDs. The user questioned their purpose. The assistant then recommended **omitting a separate offering FK for this fixed-four-offering MVP**, if the existing complaint classification, stored entitlement kind/quantity and separately configured prices suffice. **This is an architectural preference, not a confirmed physical schema decision**; deriving the appropriate pricing/category reliably must still be defined. An upgrade child must never be interpreted as 26 visits just because its target is the 26-visit product.

**Price field:** The user asked whether price should be kept on `packages`. The assistant recommended **no authoritative package-row price field**: put the actual **immutable purchase charge** in `invoice_line_items`, actual receipts in `payments`, and invoice balance/status on or derived from `invoices`. Avoid duplicate money authority. This is a **recommended schema choice, not explicitly ratified DDL**.

**Closure fields:** Existing requirement for actor, reason and timestamp still applies; the assistant suggested storing these on the package or an associated closure record, without selecting exact DDL here.

### Linked records and constraints proposed

```text
complaint_courses.id              ← packages.linked_complaint_id
packages.id                       ← packages.parent_package_id (upgrade → original)
packages.id                       ← visits.package_id (each covered visit → actual consuming row)
packages.id                       ← invoice_line_items.package_purchase_id (sold item → package row)
invoices.id                       ← payments.invoice_id (actual receipts → invoice)
```

- For covered visits, **visits 1–10** link to parent `P101`; **visits 11–26** link to child `P102` as applicable. A visit consumes one allowance, never both. The existing SQL draft does not yet implement all these links.
- **Backend-enforced**, not merely UI-hidden: parent required for upgrades; matching clinic/patient/complaint/category; 10-visit parent; eligibility within original 30-day window; no standalone sale; no multiple upgrade records for the same original (proposed unique parent constraint); no allowance above the 26 combined ceiling.
- An upgrade purchase and its invoice/invoice line must commit **atomically**. Protect concurrent attempts and the tenth-visit handover against duplicate upgrades, duplicate redemptions or a child becoming active too early.
- The existing partial rule of **at most one Active package per complaint** must still hold. **Queued** child plus Active parent is the proposed safe representation. After the original naturally exhausts, the child can become Active, with no parent reactivation.
- Review/overdue treatment warnings and one-covered-visit-per-day rules continue to apply; this extension must not bypass them.
- **No migration, PR, SQL execution, implementation or deployed-schema verification was performed** in this discussion; only existing GitHub docs and source were read.

## 6. Outstanding upgrade-specific decisions / verification

1. **OPEN — Manually Closed parent:** Is a 10-visit package closed intentionally before/after its review eligible for an upgrade? The user has not decided; assistant recommended excluding it.
2. **OPEN FOR PRECISE CONFIRMATION — Retroactive coverage in the purchase gap:** The assistant recommends leaving previously committed normal-rate visit invoices untouched even when the later upgrade inherits an earlier review-date anchor. The user said yes but requested clarification of what was being asked.
3. **OPEN FOR EXPLICIT PRICING-BASIS SIGN-OFF — Invoice charge versus payment received:** `current long catalog price − original short invoice charge` is the assistant's proposed sound accounting rule; the user initially said to subtract what was “paid.” The zero floor **is confirmed**.
4. **OPEN / ENGINEERING DESIGN — Early-queued child after manual parent closure:** If the parent is explicitly Closed before exhausting 10 visits, does the already purchased queued upgrade become usable, require separate closure/refund handling, or follow a different rule? No answer yet; needs a safe decision if early upgrades and manual closures coexist.
5. **IMPLEMENTATION / SCHEMA VALIDATION — Exact DDL:** Whether a product/price reference is needed or a fixed-type snapshot suffices; unique constraints and transaction locking; whether queued child has nullable coverage start; normal versus ₹0 upgrade invoice handling; final authorization enforcement. These do **not** imply approval for more features.
6. **DIRECTION, NOT FINAL COMMITMENT — MVP inclusion:** User favors implementation if its real engineering cost stays bounded; evaluate against final schema before treating as committed scope.

### Explicitly *not* open again

No general downgrades, no cross-category upgrades, no standalone upgrade catalog purchases, no conversion to an unrelated complaint, no restart of the original 30-day review anchor, no time-based automatic forfeiture of remaining visits, no payment-linked coverage restrictions, no rewriting original invoices, and no reopening a terminal Completed package.

---

**Bottom line:** The working design is a **dependent 16-visit extension row** linked to a particular original 10-visit package, purchasable by admin or receptionist within **30 inclusive calendar days from the original coverage start**, with an unshifted Day-30 review milestone. Financial sale records remain immutable and separate. The schema and four explicitly flagged edge cases remain to be finalized; assistant recommendations are not represented as user approvals.
