# Clinic Management System — Packages: User-Authored Requirements & Decision Handover

**Cutoff:** 28 October 2026, ~10:32 IST  
**Purpose:** Preserve the user's stated requirements, rejected approaches, evolving design choices, reported conversation with the clinic owner, and outstanding questions. This is a **decision-history handover, not an implementation specification**.  
**Evidence:** The available conversation beginning with “Let's talk about this,” earlier contextual history recovered from prior clinic discussions, and the `omsrivastava512/clinic-demo-be` repository decision records (paths at end). The conversation includes quoted assistant responses; **those are not user approvals**.  
**Fidelity notice:** This is a high-detail reconstruction of all accessible user statements, **not a claim that every utterance from every earlier conversation was retrievable verbatim**. Where the user has not accepted an assistant proposal, it is not marked as confirmed. Distinguish doctor-reported policy from the user's own designs and engineering suggestions.

## Legend and precedence

- **[OWNER—REPORTED]** The user says the doctor/owner explicitly said or agreed to this; not a direct recording from the doctor.
- **[USER—DECIDED]** Clear design choice by the user.
- **[USER—PROPOSED]** An approach the user favors, is testing, or suggested; not necessarily final.
- **[USER—BELIEF]** An intuition/assumption rather than established business practice.
- **[OPEN]** Not settled, contested, or explicitly awaiting owner/engineering decision.
- **[HISTORICAL / SUPERSEDED]** Earlier policy/proposal retained for traceability; do not silently treat as current.
- **[REPO RECORD]** Prior decision file; may be inconsistent with October 28 statements. Repository documentation is not proof that migrations/code are implemented.

**Authority:** The user's latest explicit correction overrides outdated assistant interpretations; a reported owner answer informs business policy, but does not decide unspecified implementation details. The October 28 owner call supersedes older *labels and proposed accounting mechanisms* where incompatible. Do not let assistant recommendations become decisions merely because they appeared in responses.

---

## A. Business objective, scope and why the design changed

**A1. Real clinic:** The system is being designed for an operating physiotherapy/therapy clinic, with the doctor/owner consulted for actual counter workflows. Accuracy of charges, collections, visits and package entitlements is vital. Packages are not merely speculative: an October 25 decision record says packages must be in V1/MVP and the user reported roughly **60–80% of patients** use them. Earlier scope discussions had deferred packages, then reversed that. [REPO RECORD]

**A2. Original clinic mental model:** Clinic staff/doctor referred to packages as “10 days” and “one month”; the owner informally uses days, attendance, discretionary forgiveness, Sundays, holidays, leaves and extensions. Originally the doctor said a 10-day package runs from issuance for ten days, then corrected himself when the user asked about a Sunday/clinic closure: practically he does **not** close it until the agreed visits have occurred, subject to restrictions against spreading it indefinitely. [OWNER—REPORTED]

**A3. Earlier complex approach:** Previous package exploration considered date-based working-day subscriptions versus session-based vouchers; attendance/missed days, clinic closure days, patient-specific leave/waiver, exemption of Sundays, expiry movement, mid-day emergency closure, default model vs per-patient exception, and whether to derive usage from package-linked visits. These were **questions and explorations**, not all accepted requirements. Historical Aug 17 discussion exists at `study/business_logic/packages_logic/DISCUSSION.md`. [HISTORICAL / SUPERSEDED]

**A4. User's core simplification:** Remove automated attendance-policy machinery. The software should not determine whether an absence was the patient's fault, a Sunday, a festival, an owner closure, or deserving of mercy. It should not automatically decide missed-day deductions, leave allowances, waiver days, attendance extensions, or complex clinic calendars. The owner/reception decides when to continue/stop coverage. Save straightforward facts and warn when review is needed. This was the pivotal **10-visit plus review-date** proposal. [USER—DECIDED direction]

**A5. Important owner refinement on 28 Oct:** There are now **two distinct regimes**, not simply one generic 10-visit regime: (i) ten-treatment/visit allowance with an expected working-day timeframe and soft review; (ii) **one calendar month**, explicitly not a fixed thirty-day count, with Sundays included in elapsed time and discretionary clinic-caused extensions. [OWNER—REPORTED]

**A6. Terminology:** The user repeatedly corrects himself in speech (“ten days” vs “ten visits”; “thirty” vs “one month”). Preserve the intended distinction. “Covered” means a prepaid/package entitlement, **not waived/free**. An independently granted waiver and a package-covered therapy are distinct. [USER—DECIDED / owner report]

---

## B. Offering types, price and applicability

**B1. Two current product regimes:** **10 visits** and **1 calendar month** [OWNER—REPORTED]. Avoid rewriting one month as exactly 30 days. The one-month product is relatively stricter and time-driven, while the ten-visit product is count-driven but cannot be used at any arbitrary time years later. [OWNER—REPORTED]

**B2. Historical offerings:** Prior decision log mentions routine and rehabilitation packages, each once described as 10-day and 30-day variants. Tentative 30-day rates: **₹4,500 routine, ₹6,000 rehab**; prices for shorter packages unknown. These were unconfirmed historical examples and **must not be assumed to price the newly clarified calendar-month product**. Therapy tier/category can depend on rehab/post-operation versus ordinary pain relief. [REPO RECORD / background]

**B3. Complaint-specific packages:** Package is associated with a **particular complaint/course**; e.g., knee complaint covered while an independent shoulder/back complaint remains chargeable. “Package-covered complaint” should appear clearly in complaint selection/details with coverage context and age. [USER—DECIDED direction]

**B4. Multiple procedures within a complaint visit:** An ordinary complaint-visit receives **one flat therapy fee for that complaint**, regardless of how many routine therapeutic procedures/services were performed. It is not a price per procedure. A visit is the occurrence of treatment for a complaint. Package coverage should relate to this complaint-visit charge, not to the number of procedures. [USER—EXPLICIT correction, reinforced by REPO RECORD]

**B5. Different complaint on same attendance:** If someone has an active package for knee therapy but staff select another complaint, **do not block** the other complaint. User considered warning on complaint selection, confirmation before proceeding, persistent information in next procedures/services screen, or both. User assumed such encounters would be rare; that rarity is an **assumption, not confirmed clinic policy**. Whether interruptive confirmation or passive persistent banner is best remains **[OPEN UX]**. [USER—PROPOSED]

**B6. Independent charges:** Examination fee and lab fee are **billing charges**, not proof that a clinical examination took place. They are patient-level/invoice-level, need **no complaint, visit or visit-services record**, and **never consume package allowances or become covered merely because a complaint package exists**. Do not infer a package deduction from an exam fee. [USER—EXPLICIT correction]

**B7. Other billable items:** Standalone treatments/services can be separate chargeable occurrences and invoice lines; ordinary routine procedures within a flat-fee complaint-visit do not each create a billed line. Package coverage of a complaint's routine therapy must not indiscriminately erase independently chargeable items. [REPO RECORD / user's flat-fee model]

---

## C. Ten-visit package: entitlement and the soft review date

**C1. Allowance:** Patient buys ten **covered complaint-treatment visits**. Every eligible package-covered visit consumes **one allowance**. On ten consumed allowances, the package is **Completed/exhausted**, irrespective of whether ten calendar days have elapsed. If the patient comes twice on the same day for the same complaint, the doctor eventually said two sessions can be permitted; the user proposed/accepted deducting two visit allowances for the ten-visit package, with duplicate warning. [USER—DECIDED direction; OWNER—REPORTED same-day permissibility]

**C2. Not infinite vouchers:** The owner explicitly rejected the idea that a patient attending only two times per week can stretch ten nominal “days” over nearly a month as of right. The ten-visit design therefore needs a **boundary/review mechanism**; ten credits must not be silently advertised as usable indefinitely. [OWNER—REPORTED]

**C3. Review date per purchased package:** When created, every package should receive a review date. The ten-visit package should show allowance usage/remaining, age, and a “needs review” indication after its review threshold, without secretly consuming unused visits. [USER—DECIDED direction]

**C4. Original soft-review workflow:** When the package is past review date, warn staff while moving to the next workflow screen. Choices envisioned: **Continue/Proceed**, **Back**, and **Go to Packages** (link/button). Package management lets an authorised person explicitly **close/deactivate** rather than an accidental checkout toggle disabling coverage today and re-enabling tomorrow. If closed, subsequent eligible visits are charged normally; prior covered visits remain historical facts. [USER—PROPOSED / strong design preference]

**C5. First review-date formula proposed 28 Oct:** Start from **purchase/issuance date**, count ten nominal days, but **exclude Sundays** to avoid predictable false overdue warnings. User offered to use date/calendar-library logic to count Sundays, e.g. ordinary completion might take 11 or 12 calendar days depending on Sundays. Owner replied the proposal was “perfect.” Festivals and clinic closures are *not necessarily calculated automatically*; user explicitly sought to avoid general closure calendars. [USER—PROPOSED; OWNER—REPORTED agreement]

**C6. Incomplete boundary specification:** Whether the purchase day itself is day 1, whether the warning fires on the tenth eligible day or the following day, whether Saturdays are working, timezone/branch-local date, and handling a package bought late in the day remain **[OPEN implementation/policy detail]**. Assistant-supplied date examples are **not** owner-confirmed.

**C7. Soft warning versus actual termination:** The user favors allowing staff to proceed after an overdue warning while an authorised person can manually close. That permits real-world discretion but **does not actually prohibit indefinite use** if staff keep clicking Continue. Need decide whether overdue use needs merely acknowledgement, required justification, or owner permission. [USER—PROPOSED; OPEN]

**C8. Closing early:** Explicit closure may occur with remaining allowance. Keep remaining count and closure reason/actor/timestamp; whether ordinary receptionist can close or only admin/owner is **not settled**. A closure action must be deliberate, not a routine coverage toggle. The system should not label an overdue but still usable package “Completed.” [USER—PROPOSED; engineering audit]

**C9. No absence machinery:** The user deliberately declines day-by-day absent/present states; no automatic leave waivers, no Sunday “visit consumption,” no clinic-off-day deductions, no systematic holiday extension calculation. Sundays are considered **only to compute a sensible review threshold** in this proposal. [USER—DECIDED direction]

---

## D. One-calendar-month package: different entitlement

**D1. Product is “one month,” not “30 days.”** Owner insisted on this distinction. An informal example was purchase on the 1st, coverage through the 30th, but that **does not resolve months of 28, 29, or 31 days, or purchases mid-month**. [OWNER—REPORTED; OPEN precise boundary]

**D2. Sundays do not extend coverage:** Unlike the ten-visit review calculation, Sundays occurring inside the purchased calendar month **pass normally**. A patient receives no automatic extra days because of Sundays. [OWNER—REPORTED]

**D3. Actual clinic closures/festivals:** Owner said that if the clinic shuts for a festival/personal reason, they ordinarily grant a concession/extension. This is discretionary and means the one-month product isn't absolutely immutable in real operations. [OWNER—REPORTED]

**D4. End-of-month enforcement unsettled:** User expressly asked whether monthly expiry should have the same soft-warning model as ten visits, or something **more blocking**. No final user decision or doctor approval of a hard block. Prior assistant recommended a hard cutoff plus manually authorised extension; that is **ASSISTANT RECOMMENDATION ONLY**, not ratified. [OPEN]

**D5. Monthly same-day double treatment:** Owner said the patient may genuinely be treated twice for the same complaint in one calendar day. With a one-month package, **one visit is package-covered, the other charged at normal complaint-visit rate**. User wondered whether this necessitates a per-visit toggle-off; no toggle design was approved. The business result is clear but technical selection (which visit gets coverage; ordering; corrections) is **[OPEN]**. [OWNER—REPORTED]

**D6. Monthly definition unsettled:** “One month” might be from purchase/start date to previous day before next-month anniversary, subject to end-of-month adjustment; **not confirmed**. Whether a month package has a visit-count ceiling was never expressly specified; do not invent 30 allowances or unlimited twice-daily free treatments. The same-day one-covered rule is the only explicit daily limit. [OPEN]

**D7. Extensions:** User originally wanted **no extension machinery** for the simplified ten-visit design. The later owner report creates an actual monthly extension need. Potential implementation of a manually changed end date, permission, reason, and audit is a discussion option only; precise handling not decided. [OPEN, historical context preserved]

---

## E. Same patient + same complaint twice in one day

**E1. Earlier user expectation:** User initially thought one complaint should not be treated twice on the same day and contemplated a warning or hard guardrail depending on owner confirmation. [HISTORICAL / SUPERSEDED]

**E2. Owner reply on Oct 28:** Doctor said **why prohibit it?** They may accept two treatments the same day; it is rare, but possible, and their choice. **Show a warning, do not prohibit the second genuine complaint-visit.** [OWNER—REPORTED]

**E3. Ten-visit case:** Owner entertained the extreme example of consuming ten visits in five days by attending twice a day and expressed discretion rather than absolute refusal. User proposed that recording a second legitimate same-day complaint-visit consumes another allowance in the 10-visit package. [USER—PROPOSED / supported by doctor scenario]

**E4. Month case:** First same-day treatment covered, second normally billed, per owner. Neither a `UNIQUE(complaint_id, date)` across all visits nor an automatic second free coverage is correct. Warning should disclose the earlier occurrence and what will happen on confirmation. [OWNER—REPORTED; implementation implication]

**E5. Corrections and technical handling OPEN:** Differentiate genuine second treatment from double-click/retried submission, duplicate data entry, or editing existing visit; avoid two checkouts concurrently both taking the “first” monthly covered slot. Need visit-level linkage to actual package redemptions (not inference from shared complaint alone), idempotency, concurrent locking/constraints. These are engineering safety requirements, not owner decisions. [OPEN technical audit]

---

## F. Automatic coverage, checkout and complaint-selection user experience

**F1. Earlier exploration:** A user-selectable “Use package for this visit” control was discussed, with package usage saved only on final confirmed checkout, not on preview/toggle. Assistant recommended an explicit choice to avoid accidentally failing to link an available package. These were **assistant-developed suggestions**, not approval. [HISTORICAL proposal]

**F2. User rejection of ordinary toggle:** The user argued strongly that no normal patient with prepaid relevant treatment would ask to be billed at regular price for the same included therapy. They therefore want **no arbitrary on/off coverage toggle** at visit checkout. An active applicable package should ordinarily apply automatically. [USER—DECIDED direction]

**F3. Deliberate manual closure instead:** To stop package coverage intentionally, go to **Packages** module and close/deactivate it explicitly, rather than a receptionist unpredictably toggling package coverage per appointment. [USER—DECIDED direction]

**F4. New exception after owner call:** The calendar-month product's **second same-day complaint-visit must be chargeable** even if the monthly package remains otherwise active. This is a **rule-based non-covered visit**, not necessarily a user-controlled toggle. User explicitly raised whether a toggle-off is needed but **did not settle the mechanism**. [OWNER—REPORTED outcome, OPEN design]

**F5. Active package visibility:** Complaint-selection UI should show a complaint has a package, package age/usage and status. Post-review date: interruptive confirmation with Continue/Back/Packages; a package with unused benefits and overdue review must not be falsely marked completed. [USER—PROPOSED]

**F6. Selecting a different complaint:** User wants no block, but considered warning/confirmation and/or persistent banner on the next procedures/service selection screen indicating an active package exists for a different complaint. Specific triggers/content not locked. [USER—PROPOSED]

**F7. Admin attention:** Admin dashboard should have **Needs Attention** for overdue packages; user later suggested elevating unpaid/partially paid package purchases there. Notifications are primarily to receptionist once review threshold is reached; decide whether they are per encounter, dashboard item, or persistent flag. [USER—PROPOSED]

**F8. Coverage and invoices distinct:** A package-covered visit must remain a real recorded clinical visit/redemption. Invoice/zero line decisions must not determine whether allowance was consumed. A waived charge is not a package-covered charge. [Concept discussed; REPO RECORD specifically requires ₹0 covered-visit invoice lines]

---

## G. Financial events, payment, activation and billing source

**G1. Purchase is billed:** Buying a package creates a **package-purchase invoice**, while receiving money creates **separate positive payment record(s)** against that invoice. The invoice's amount is what is owed; a payment receipt is what was collected. Do not call package purchase itself a payment. [USER—EXPLICIT]

**G2. Partial payments are allowed generally:** Invoice may be Pending (no payment), Due (partial payment), Paid (fully settled or otherwise authorised settlement). The clinic may physically collect part/none of the package invoice at counter. [USER—DECIDED direction, existing payments policy]

**G3. User's October 28 stance on partial-payment treatment:** Do **not** build automatic package-consumption blocking based on partial or zero payment. The receptionist/admin/clinician enforces the collection policy in the real clinic. The software should accurately show outstanding and may prioritise unpaid/partial packages in Needs Attention. This applies even if a partially paid package is used for treatments. [USER—DECIDED direction]

**G4. Historical conflict:** The Oct 25 package decision file states “coverage activates after full payment.” Assistant later highlighted the inconsistency. User rejects adding a package-specific automatic blocking rule based on invoice balance. **This is a superseded or at least contested older rule and requires an explicit update to decision docs**. User did *not* supply an exact activation event for all purchase/issuance/cancellation edge cases; assistant's “invoice issuance automatically activates” is a recommendation, not necessarily an explicit user resolution. [OPEN precise trigger]

**G5. Separate accounting from coverage:** Outstanding debt remains visible and collectible regardless of how many covered treatments happened. Partial collection neither consumes extra treatment allowances nor changes the original billed value. [USER policy implication]

**G6. Flat rate and package invoice lines:** The current Oct 25 repository docs establish one flat therapy invoice line per ordinary complaint-visit, with a package-covered visit represented as a **₹0 line** and an invoice still produced when all covered visits total ₹0. A coverage linkage is needed independently of printed invoice lines. There is a recorded historical **zero-total invoice vs positive-total constraint conflict**; don't silently erase it. [REPO RECORD]

**G7. Patient-level exam/lab charges independent:** Exams/labs are billable amounts on the patient/invoice side, not complaint-specific treatment or proof of assessment. They never consume package credits and should not be auto-covered by a complaint package. [USER—EXPLICIT]

**G8. Examination/gap/late-fee context:** Historical Module 2 logs record an initial examination example **₹350**, 1–9-day no new fee, 10–29-day **₹150**, 30+ days **₹350**, but a subsequent Oct 23 examination-fee decision leaves recurrence, whether old ₹150 gap-return fee survives, exact thresholds and patient-wide computation open. User separately said **every completed encounter/visit can matter for examination-gap recency, even therapy-only**, rather than only visits that paid examination fees. Do not infer that covered package therapy never affects recency just because exam fees never consume the package. **Whether the old “late/lapse fee” is a distinct billable component is not resolved here.** [HISTORICAL / OPEN cross-module]

**G9. Waiver vs package coverage:** Admin/owner settlement waivers are a separate fact from covered treatment. A ₹0 payment record should not be invented for a waiver or a package-covered zero-priced line. There is a separate out-of-scope refund discussion and no permission to conflate refunds, waivers, discounts, original billed charges or coverage. [REPO RECORD / prior user concern]

**G10. Financial integrity from prior project decisions:** Original invoice line charges frozen on issue, receipt rows strictly positive and linked to invoice, CASH/UPI split recorded separately, outstanding and collections separately computed, later Due receipts remain permitted, and no retroactive repair of saved payments in MVP. Package data must work with rather than override these general policies. [REPO RECORD]

---

## H. Refunds, cancellation, manual close and MVP boundaries

**H1. Earlier MVP deferral:** User explicitly considered deferring package refunds, extensions, package-specific adjustments and other lifecycle complexities to avoid turning the MVP into a much larger project. **This was a scope/priority statement, not a claim refunds never happen.** [USER—DECIDED direction at the time]

**H2. User later reopened refunds as valid business cases:** The user contemplated a **separate refund module**, especially for a purchased/paid package needing a refund. Even if the clinic says “no refunds” as policy, a rare exceptional refund may be necessary. Concern: refunding (e.g. ₹200) must not make reported total collections/revenue inaccurate. Asked how to represent it relative to `payments` and source invoices, and which business actions refunds would affect. [USER—PROPOSED / OPEN]

**H3. No selected refund architecture:** Earlier assistant suggested dedicated refund records rather than negative payment rows, positive refund amount, audit details and separate gross/refund/net reporting; **these are assistant recommendations, not user approvals**. Refunds must not be declared implemented, included in MVP, automatically restock visit allowances, or automatically close a package unless user explicitly chooses such a rule. [OPEN / historical assistant proposals]

**H4. Manual closure vs refund:** Closing a package with unredeemed benefits raises whether any amount is refunded, written off, retained or handled outside the app. The user's latest goal is **explicit staff/owner-controlled closure**; reimbursement policy and authority remain open. A closed package with unused visits is not equivalent to a fully consumed package. [USER—PROPOSED / OPEN]

**H5. Extension tension:** Earlier “no extensions or attendance tracking” applies to the simplified ten-visit design. Owner explicitly says monthly plans occasionally need concessions when **clinic** closes for festivals/personal reasons, reopening a narrow manual-extension requirement. Do not generalise from this into full automatic attendance-policy machinery. [OWNER—REPORTED, OPEN implementation]

---

## I. Multiple packages and coverage association

**I1. Earlier rule/intuition:** Do not allow another simultaneously active package for the same complaint, to avoid ambiguous entitlement selection. Original schema discussion highlighted that package and visit both relate to complaint but this shared parent does **not** identify which particular package covered a visit. [HISTORICAL proposal and structural concern]

**I2. Latest user challenge (Oct 28):** User explicitly questioned that old one-active-package restriction after discussing twice-daily attendance and potential multiple purchases for same complaint: “What if the patient wants to take two packages of the same?” Concern that two active packages create ambiguous consumption. **No user decision to allow or disallow overlaps was made.** [USER—OPEN]

**I3. Distinction not yet agreed:** Buying a second package ahead of time, starting it later, and having two concurrently eligible packages are **different scenarios**. Prior assistant recommended a maximum of one active entitlement per complaint and possible sequential renewal, but this **must not be recorded as confirmed user/owner policy**. [OPEN]

**I4. Specific coverage proof:** Historical package design discussion proposed linking each covered visit to specific package (such as `visits.package_id`) and counting those links, instead of an independently drifting attended-days counter. A single complaint relationship does not prove particular package redemption when there are several sequential/overlapping packages. Current migration files may still use older duration/missed-day structures. Schema details, normalized link vs redemption table, uniqueness/concurrency controls remain for engineering audit. [HISTORICAL PROPOSAL / OPEN exact representation]

---

## J. Proposed lifecycle vocabulary (NOT yet a ratified schema)

- **Purchased / invoice issued:** Package sale recorded; balance can be zero, partial or full payment. Exact entitlement activation trigger open.
- **Active:** Package presently covers eligible treatments under product rules; **do not equate with paid**.
- **Review due/overdue:** For ten-visit package, review threshold elapsed; advisory overlay, not necessarily a change of lifecycle state.
- **Completed / exhausted:** Ten-visit allowance actually consumed; don't falsely complete on review date. For monthly package, term-end/expired is distinct from exhaustion.
- **Expired / period ended:** Monthly coverage boundary elapsed, subject to still-open enforcement/extension policy.
- **Closed/deactivated:** Explicit deliberate clinic action, potentially with unused allowance; record reason/actor/time.
- **Extended:** Owner-reported possible monthly exception, not a final implementation model.
- **Payment states:** Pending / Due / Paid belong to invoice settlement, **not package coverage status**.

These labels and separation are working vocabulary for the handover. Actual columns and transitions remain undecided.

---

## K. Confirmed / chosen / open register (as of 28 Oct)

| ID | Requirement / issue | Evidence-based status | Notes |
|---|---|---|---|
| P01 | Packages required in MVP | REPO RECORD user decision | Earlier deferral superseded |
| P02 | Two regimes: 10-visit and one-calendar-month | OWNER—REPORTED | Not 10 calendar days and not fixed 30 days |
| P03 | Complaint-specific coverage | USER—DECIDED direction | Keep separate fee and occurrence sources |
| P04 | Flat charge per complaint-visit, not each procedure | USER—EXPLICIT | Routine procedures don't add fee lines |
| P05 | Exam and lab fees patient-level, not complaint-package use | USER—EXPLICIT | Also distinguish fees from clinical examination proof |
| P06 | Ten-visit allowance consumption | USER—DECIDED direction | Same-day two genuine visits may consume two |
| P07 | Ten-visit Sunday-excluded soft review | USER—PROPOSED + OWNER AGREED | Exact date boundary unresolved |
| P08 | No absent-day, leave, holiday/closure attendance engine | USER—DECIDED direction | Sunday's review-date skip is limited exception |
| P09 | Manual intentional closure instead of ordinary toggle | USER—DECIDED direction | Who can close, reasons, refunds still open |
| P10 | Warning after overdue review, Proceed/Back/Packages | USER—PROPOSED | Approvals vs click-through open |
| P11 | Active package indicator, usage and age on complaint UI | USER—PROPOSED | Display rules/open UX |
| P12 | Warn/inform on different complaint but don't block | USER—PROPOSED | Modal vs persistent header unresolved |
| P13 | Admin Needs Attention on overdue packages | USER—PROPOSED | Add unpaid/partial visibility proposed |
| P14 | Monthly Sundays do not extend coverage | OWNER—REPORTED | Dates of month remain ambiguous |
| P15 | Monthly clinic-caused disruption may justify concession | OWNER—REPORTED | Manual extension workflow undecided |
| P16 | Monthly review vs hard stop | OPEN | Assistant recommended hard stop, user did not decide |
| P17 | Same complaint twice in one day allowed with warning | OWNER—REPORTED | Reverses user's earlier expectation |
| P18 | Ten-visit same-day two uses | USER—PROPOSED after owner discussion | Clarify any rare discretion override if needed |
| P19 | Monthly second treatment same date billed normally | OWNER—REPORTED | Which record takes covered slot is open |
| P20 | No ordinary checkout coverage toggle | USER—DECIDED direction | Monthly second treatment may be automatic exception |
| P21 | Package invoice distinct from payment receipts | USER—EXPLICIT | General finance design retained |
| P22 | Partial/unpaid balance does not auto-block usage | USER—DECIDED direction | Conflicts with older full-paid activation doc |
| P23 | Exact event activating an entitlement | OPEN | Issued invoice vs saved purchase vs start date |
| P24 | Multiple packages same complaint/overlapping periods | OPEN | Do not assume one-active restriction is accepted |
| P25 | Covered-only invoice ₹0 line | REPO RECORD | Separate design conflict with zero-total checks |
| P26 | Package refunds | OPEN/possibly future | User raised real use case, not ratified schema |
| P27 | Outstanding package invoice reporting/attention | USER—PROPOSED | Do not mistake visibility for enforcement |
| P28 | Monthly exact “one month” calculation | OPEN | Month-end, leap year, date-inclusive rules |
| P29 | Existing patients' imported active packages | OPEN | Prior repo flagged migration |
| P30 | Same-checkout package purchase + treatment coverage | OPEN | Historical proposal, activation/payment ordering affects eligibility |

---

## L. Actual questions to resolve, prioritised

### Business questions for doctor/owner (not technical SQL questions)

1. **Ten-visit overdue authority:** Once the Sunday-adjusted review threshold passes and unused allowances remain, is a receptionist's Continue acknowledgement sufficient, or must the owner approve/close? Can credits remain open indefinitely if nobody closes?
2. **One-month exact coverage dates:** From a 15 October purchase, does coverage end 14 November or another date? What does “one month” mean for 31 January and leap years? Does eligibility begin on purchase, a chosen start date, or first treatment?
3. **Monthly end enforcement:** At the end of the month, does coverage stop automatically (with authorised manual extension), or should staff be free to acknowledge an overdue warning and continue?
4. **Monthly clinic closure extension:** Who authorises it, how many days are added, and can it be entered after the cutoff? Is reason required?
5. **10-visit same-day double treatment:** Are two consumed allowances always acceptable when the owner agrees to genuine repeated care, or can the owner choose to bill the second visit differently?
6. **Early/unused package closure:** Who may manually close it, what reason is required, and is unused prepaid value refunded, forfeited or handled outside MVP?
7. **Purchase while earlier package still active:** Does the clinic actually sell an advance renewal or concurrent packages for the *same complaint*? If so, which one should cover treatments and from what date?
8. **No-payment package sale:** Is zero-receipt invoice issuance enough for clinic staff to allow covered therapy? User wants no software auto-block; doctor may still specify an operational cashier policy.
9. **Is a month's included therapy limited to one visit per complaint per day in every case?** Owner said second same-day treatment billed separately; confirm what happens if the first visit is cancelled/voided or a later entry is backdated (if future corrections ever supported).

### Engineering / product questions, **do not make doctor decide database architecture**

- Model offerings and purchased entitlement snapshots separately from invoices, payments, complaint-visits and redemptions.
- Define exact review-date calculation, timezone, date-inclusion, historical pricing snapshot and whether every package type has a review date or monthly uses an end date instead.
- Validate package/visit same patient/clinic/complaint; one redemption per actual intended treatment event; retry idempotency and row locking for last remaining allowance and one-per-day monthly coverage.
- Keep usage truth independent of invoices/zero-fee lines, and distinguish waivers.
- Avoid unconditional `UNIQUE(complaint_id, local_date)` on visits: multiple genuine same-day sessions are allowed.
- Define status derivation versus persisted lifecycle transitions; overdue notification de-duplication; access control for closure/extension; audit information.
- Clarify whether second same-day monthly visit is automatically regular-priced (preferred *technical possibility*, not confirmed user mechanism), or needs explicit staff decision.
- Resolve invoice creation for fully covered checkout and source-link audit; avoid creating fake ₹0 receipts.
- Resolve existing package imports, old `duration_days`/`expiry_date`/`missed_days` schema and staged migration; distinguish design logs from working schema.
- Separate package refund/cancellation from original receipt, waiver and invoice accounting; MVP boundary to be revisited explicitly.

---

## M. Key chronology / change log

- **Aug 17 earlier package exploration:** Calendar subscriptions vs session vouchers, closure dates, missed days, exemptions and possible package-visit direct linkage; detailed exploratory discussions, not all approvals.
- **Oct 22–25 Module 2:** Packages moved into mandatory MVP; separate package purchase from treatment, flat complaint fee, immutable invoice lines, ₹0 covered-visit lines, positive receipt records. Package log contained historical “10-day/30-day” and “full payment activates” statements; some items explicitly still open.
- **Oct 26 cross-module:** Examination fee timing/occurrence and patient-level treatment context debated; complaint lifecycle/closure policy and normalization/denormalization questions marked separately for audit, not automatically package policy. User emphasised separate patient-level examination charges rather than confusing clinical proof with charge.
- **Oct 27 initial redesign (quoted at start of current conversation):** Ten consumed complaint visits, review date not hard expiry, abandon absence/extension calculation. Earlier assistant advocated deliberate coverage choice and proposed some invoice mechanics; user later rejects normal toggle.
- **Oct 27–28 user clarifications:** No ordinary toggle; explicit package management closure; package badge/age, overdue Continue/Back/Packages warning; non-covered complaint warning; admin Needs Attention; purchase invoice/payments separate and no blocking usage for partial payment; exam/lab independent patient-level charges.
- **Oct 28 ~10am doctor call:** Two regimes—10 visits with Sunday-adjusted review boundary (doctor agreed), and one actual calendar month (Sundays count as elapsed time, occasional clinic-closure extensions). Twice-daily complaint therapy allowed with warning. For 10-visit coverage, second treatment may consume another allowance; for monthly, second same-day treatment is regular-priced. User reopens overlapping package possibility and monthly enforcement question.

---

## N. Relevant repository references (state of records, not proof of deployment)

Primary backend: `https://github.com/omsrivastava512/clinic-demo-be`  
- `study/module_2/decisions/packages.md` — historical package options, full-payment activation, ₹0 covered lines and open O08. **Update required** for Oct 28 changes.
- `study/module_2/decisions/payments-and-settlement.md` — invoice payment states, positive receipts, waivers, historical immutability and outstanding.
- `study/module_2/decisions/checkout-and-invoice-lines.md` — flat therapy per complaint-visit, covered visit line, original invoice-line authority, checkout boundaries.
- `study/module_2/decisions/examination-fees.md` — revised recurring examination discussion; older price/interval policy cannot be uncritically imported.
- `study/business_logic/packages_logic/DISCUSSION.md` — older attendance/calendar and package ID debates.
- `study/business_logic/packages_logic/README.md` and `DEFERRAL.md` — historical specification/deferral records, not guaranteed current policy.
- `study/business_logic/general/migration/handover-packages.md` — previous migration notes, may describe superseded calendar-oriented structure.

Other relevant precedent: the user's separate Oct 27 refund architecture discussion; Oct 26 examination-gap and standalone-service discussions; explicitly parked complaint-closure and normalization audits. This file does **not** resolve those adjacent audits.

---

## O. Handover instructions for the next discussion / coding agent

1. **Do not treat the last assistant answer as the user's decision.** Check the `[OWNER—REPORTED]`, `[USER—DECIDED]`, `[USER—PROPOSED]`, `[OPEN]` tags.
2. **Don't rebuild the old attendance/missed-day engine** just because historic repo files still describe it. The user's core simplification is intentional.
3. **Don't equate “ten visits” with “ten days,” “one month” with “30 days,” “covered” with “waived,” “invoice issued” with “payment received,” or “review overdue” with “completed.”**
4. Never impose full-payment gating, a hard monthly expiry, one-active-package restriction, refund semantics, or a mandatory toggle as already user-approved. These are contested/open or assistant-origin suggestions.
5. Package coverage must not suppress patient-level exam/lab charges; flat complaint-visits are distinct from performed procedure counts and from standalone charged occurrences.
6. Record each new decision with exact user/owner provenance, date, status and any earlier rule it supersedes. Prefer bite-sized sequential resolution rather than settling all open issues at once.
