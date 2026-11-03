# Package Module — Reconstructed Final Decision Questionnaire

**Reconstruction date:** 3 November 2026  
**Source of decisions:** The original 40-question questionnaire pasted in this conversation, followed by the voice-mode discussion in this same conversation. Exact transcript quotations are distinguished from paraphrased summaries where used.  
**Scope:** Complete questionnaire, Q01–Q40; decisions/discussion recorded for Q01–Q17 only.  
**Important:** This is a *discussion record*, not proof that any rule is implemented. I have not re-verified GitHub or other chats in preparing this reconstruction. The full set of 40 question scenarios and answer options is reconstructed below (some source wording is condensed rather than reproduced verbatim), with decisions and discussion added; a past assistant's claim that Q18–Q20 were already settled was incorrect.

## Read this first — status and fidelity

- **Discussed and decided:** Q01, Q02, Q04–Q16.
- **Decided with a clarification still worth confirming:** Q03. The latest instruction is a **default maximum of 1 covered visit per day for every package**, with an admin-adjustable limit. Earlier, the user said **2** for the 10-visit package. The later default supersedes a universal default of 2, but whether **2 should be preconfigured for the 10-visit catalog offering** was not explicitly revisited.
- **Discussed but explicitly dropped from this decision round:** Q17. Do **not** record option A, B, or C as accepted; an intermediate financial workflow was brainstormed but then the scenario was rejected as not worth covering.
- **Not yet discussed or approved in this conversation:** Q18–Q40. They remain unanswered; some concepts may overlap previously settled principles, but no option is selected here.
- **Cross-cutting modification to the old baseline:** Package *purchase/registration date* is no longer automatically the review-clock start. A separate **package start date** determines when coverage begins and when the review clock starts. The start date is purchase date or purchase date + 1 day, depending on whether a previous package already covered the same complaint on that day.
- **Operational scope restraint:** Do not add new audit infrastructure, a delinquent-renewal oversight report, a reminder scheduler, a note-retention policy, or extra complaint alerts based on exploratory assistant questions in the conversation. None was approved.

### Decision register (quick reference)

| Question | Recorded disposition | Caveat |
|---|---|---|
| Q01 | Custom: review clock starts on package **start date**, not necessarily purchase date | Changes original registration-date premise |
| Q02 | A: same-day replacement coverage allowed if no earlier same-complaint package coverage that day | Prior package must be terminal |
| Q03 | Custom: default daily cap 1 for all; admin can change cap at sale or later, including 26-visit | Earlier preference of 2 for short package needs configuration distinction |
| Q04 | A: explicit irreversible-closure/unused-allowance warning | Later reinforced under Q13 |
| Q05 | A with warning: allow renewal despite old debt; no blocking or admin approval | No new credit-oversight report in MVP |
| Q06 | A: reason for each post-review covered visit | May reuse suggested text |
| Q07 | A: submitted reason immutable for everyone | No edits even by admin |
| Q08 | B in substance: admin-authorized post-review visits do not generate Needs Attention or pings | Preserve underlying visit record; do not assert general audit log exists |
| Q09 | C: separate persistent Needs Attention queries for overdue-unused packages and post-review coverage | No reminder cycle |
| Q10 | A: confirmation on each post-review covered visit | Mandatory receptionist reason remains per visit |
| Q11 | A: browser-local suggestion/autocomplete only | Stored reason itself belongs to the visit, not just the browser |
| Q12 | A: deliberate closure is permanent, no reopening | No reinstatement for patient request |
| Q13 | A in substance: mistaken closure not reversible in MVP | Strengthen preclosure identification and warnings |
| Q14 | Custom / original hypothetical N/A: **nonpayment alone is not a closure trigger**; subsequent payment cannot reopen a closed package | Receivable remains separate; new package for future course |
| Q15 | C: suggested closure reasons plus editable custom text | Exact preset list not individually approved |
| Q16 | A: closed/completed packages visible in patient history | Show usage, original allowance, closure reason, invoice link |
| Q17 | **Dropped / not selected** | No transfer/refund/reclassification exception approved here |
| Q18–Q40 | **Unanswered** | No option inferred |

## Already settled before this discussion — with superseding clarification

The original questionnaire declared the following settled (quoted in substance). Preserve them unless a later explicit decision below modifies them:

- Registration activates the package regardless of whether ₹0, partial, or full payment has been collected. **Clarification from Q01–Q02:** registration creates the purchase immediately, but the newly introduced *package start date* controls the first day coverage can be consumed. If coverage starts tomorrow, payment still does not control that date.
- ~~Registration date starts the review clock.~~ **Superseded in this discussion:** the review clock starts on the **package start date**, which can be the purchase date or the next calendar date.
- There is no separate historical treatment-occurrence timestamp in the workflow; operational date/time is used.
- Short offering: 10 covered visits, with established Sunday-excluded review counting. Long offering: 26 covered visits maximum, with a fixed 30-calendar-day review period. **Clarification:** use the chosen start date as day 1 for the relevant review calculation; do not infer a new counting algorithm from this record.
- Review is advisory; unused allowances do not expire automatically.
- Overlap of active packages for the same complaint is forbidden. The old active package must be completed or deliberately closed before replacement.
- Closing forfeits unused allowances; no automatic transfer.
- A new package cannot cover a second treatment for the same complaint on a day when the previous package already covered a treatment.
- Admin can close packages anytime; future receptionists only from the day after review, with a recorded reason. **Later operational clarification:** early admin closure should ordinarily be for patient-requested termination or correction of an erroneous registration, not automatic debt enforcement. The broader permission and this intended usage should not be conflated.
- Post-review treatment justifications are in MVP: receptionist mandatory, admin optional.
- Complaints with active packages cannot be closed.
- Package coverage, invoice payment, invoice waiver and refund are distinct operations.
- No automatic holiday extension engine, automatic expiry buffer, or general refund/void module is being added to MVP. **Important unresolved tension:** the discussion of Q17 briefly contemplated recording refunds, but the scenario was subsequently dropped; that brainstorming must not silently authorize a refund module.

---

# A. Renewals, activation, and entitlement boundaries

## Q01. When does the replacement package's review clock start?

**Original scenario.** November 10: Package A covers its final treatment and completes. The patient buys Package B at noon. B cannot cover another treatment that day.

Should B's 30-day review clock begin:

- **A.** November 10, its registration date, even though today's coverage is unavailable.
- **B.** November 11, its first eligible coverage date.

*Original recommendation:* A, consistent with the then-current registration-date rule.

**Recorded answer — custom rule (closest to B in this scenario).** The system distinguishes **package purchase/registration date** from **package start date**. If the previous package has already covered a treatment for that complaint on the purchase date, the replacement's **start date is the following clinic-local calendar day (T+1)**. If there has been no such treatment that day and replacement is otherwise eligible, start date is T. The review clock uses the **package start date**, not the transaction/purchase date. Therefore, in the November 10 example, B is purchased November 10, starts November 11, and the 30-day review clock starts November 11.

**What the user said, in substance.** Purchase date and package start date are different; a separate start-date field is needed; *“from package start date.”* This **supersedes**, rather than merely rephrases, the original registration-date rule.

**Boundary:** The user proposed adding a field in the packages table. That is an implementation suggestion, not a mandate to use a specific schema design; the required business fact is that both dates have distinct meanings.

## Q02. What if no coverage was consumed earlier that day?

**Original scenario.** November 10: Package A has 18/26 visits used. No treatment has occurred today. Admin closes A, forfeiting eight allowances, and registers Package B. The patient then receives a legitimate treatment under B that afternoon.

- **A.** B can cover it because no package has covered that complaint today.
- **B.** A replacement package cannot cover treatments until the next calendar day, regardless of earlier usage.

*Original recommendation:* A.

**Recorded answer — A, explicitly confirmed.** Close the old package; if **no earlier same-complaint treatment was covered by another package today**, the newly bought package starts today (T) and can cover today's legitimate treatment. Otherwise it starts tomorrow (T+1). The user confirmed this explicit Q02 restatement with “Yes.” The previous package must first be Completed/Closed; this does not permit concurrent active-package overlap.

## Q03. Can a short package cover two genuine visits on one day?

**Original scenario.** A patient has an active 10-visit package with seven allowances remaining. They receive treatment at 10 AM and again at 6 PM for the same complaint. Both are genuine treatments under the same purchased package.

- **A.** Both are covered, consuming two allowances.
- **B.** Only the first is covered; the second is ordinarily chargeable.

*Original context:* A had been proposed in the October 28 handover but was less conclusively settled than the 26-visit offering's daily limit.

**Recorded answer — custom configuration rule.** Each **purchased package** has a **maximum covered visits per clinic-local calendar day** configuration. **Default is 1 for every package.** Admin may set or change the cap **at purchase or later**, including on the 26-visit offering; at redemption time, the current cap determines whether another visit is eligible. Every covered visit consumes one allowance, subject to the overall remaining-allowance limit. No retroactive change to committed visits is contemplated.

**Discussion sequence (preserve both statements):**

1. User first answered that the maximum for the 10-visit package was **2**.
2. User then corrected the design: *default* should be **1 for every package**, with admin discretion to raise it when purchasing **or later**.
3. User specified this is a mutable operational configuration, not a historical entitlement term they want tracked through versions.
4. Assistant recommended recording each edit in an audit log; user stated there is **no general audit-log table in MVP** and explicitly deferred that work, accepting the blind spot.

**What is decided:** Default 1, changeable by admin, for both 10- and 26-visit packages, with no edit-history audit in MVP. **Potential ambiguity, not silently resolved:** Is the 10-visit catalog *initial configured value* meant to be 2 while global fallback/default is 1? The later instruction says “default ... for every package ... 1,” so this document records **1 as the operative default** and **2 as an allowed admin-set value**. Do not hardcode 2 as the automatic short-package default without a later explicit confirmation.

**Explicit MVP concession:** Changes to this operational cap are not tracked in a dedicated audit trail. Committed actual visit records remain retained as usual. This is a chosen accountability limitation, not a claim that no risk exists.

## Q04. Should early renewal require an explicit forfeiture confirmation?

**Original scenario.** Package A has 8/26 unused. Staff select Close to permit a replacement purchase.

- **A.** Show a confirmation stating that eight remaining treatments will be forfeited.
- **B.** Show an ordinary closure confirmation without a separate allowance-loss warning.

*Original recommendation:* A.

**Recorded answer — A, supported by subsequent explicit discussion.** Require a prominent, destructive confirmation identifying **the exact package being closed**, **unused allowances being forfeited**, and the fact that **closure is irreversible**. Under Q13 the user explicitly insisted that staff see the exact package, the remaining allowance information, clear warnings, and an affirmative confirmation. Do not treat a generic OK dialog as sufficient.

## Q05. Can a patient buy another package while the previous package's invoice remains unpaid?

**Original scenario.** Package A has been legitimately Closed or Completed. Its ₹4,500 purchase invoice still has ₹2,000 outstanding. The patient wants Package B for the same complaint.

- **A.** Allow the purchase; old debt remains independently collectible.
- **B.** Prevent a new package purchase until the previous balance is settled.
- **C.** Allow only admin to authorize the new purchase.

**Recorded answer — A, with prominent warning.** The application **assists rather than blocks** the staff's credit decision. Allow new package purchase despite outstanding older package invoices; clearly show the unpaid amount and provide access to invoice details. There is **no automatic rejection**, **no compulsory admin approval**, and **no new credit-scoring mechanism**. Old debt remains independently collectible; paying it does not retroactively buy or renew entitlement.

**Discussion scope boundary:** The assistant floated a separate admin report showing patients who purchased new packages while owing old balances, including purchase dates/registering staff. User said an unpaid-invoice report already exists, but this additional cross-report is **deferred as scope creep**. Do not mark it as an MVP requirement.

---

# B. Post-review treatment justification and admin oversight

## Q06. How often must a receptionist provide a justification?

**Original scenario.** A package is review-overdue. The receptionist covers treatments on days 31, 33, and 35 because of a Diwali clinic closure.

- **A.** Every post-review covered visit needs its own reason, but previously used text can be selected again.
- **B.** Enter one package-level justification that applies to all subsequent post-review visits until closure.
- **C.** Enter one reason per day, covering all eligible visits for that package that day.

*Original recommendation:* A.

**Recorded answer — A, emphatically confirmed.** **Each individual post-review covered visit must have its own submitted receptionist justification.** Reused typing suggestions do not change the per-visit storage requirement. A one-time package-level or daily blanket justification is not sufficient.

## Q07. Can a submitted justification note be edited?

**Original scenario.** A receptionist enters “Diwali holiday” while covering a day-31 visit. Two days later, they realize the actual cause was a different clinic closure.

- **A.** Notes are immutable once the covered visit is committed. Any correction requires a separately audited future workflow.
- **B.** Admin can edit notes, preserving original text, actor, and modification history.
- **C.** Both admin and receptionist can edit their own submitted notes with full audit history.

*Original recommendation:* A for MVP.

**Recorded answer — A, stricter role clarification.** **No one can edit a committed note, including admin.** A separate future correction workflow was *not* approved or designed here. Do not quietly introduce a backdoor edit operation.

## Q08. Should admin see their own overdue treatments in Needs Attention?

**Original scenario.** Admin personally authorizes five post-review covered treatments.

- **A.** All five remain visible in Needs Attention for review, even though admin performed them.
- **B.** They remain in audit history, but no redundant Needs Attention item is shown to that same admin.
- **C.** Show them in Needs Attention, but not as separate push/inbox notifications.

**Recorded answer — B in substance, with clarified global suppression.** When **admin personally authorizes** post-review covered visits, **no Needs Attention item or ping is created for any admin** on account of that authorization. A receptionist-authorized post-review visit does enter admin Needs Attention. The underlying committed treatment and its associated actor/coverage facts remain part of normal records.

**Caution:** The original B option says “audit history”; user explicitly stated elsewhere that a general audit-log table does not exist in MVP. Do not invent a general audit-history feature. Preserve already-required committed visit information; separate audit architecture remains unapproved.

## Q09. Should packages that are overdue but have no post-review treatments appear in Needs Attention?

**Original scenario.** A package reached its review date 20 days ago, retains 12 allowances, and has not recorded any treatment since review.

- **A.** Show it in Needs Attention as an overdue inactive package.
- **B.** Show Needs Attention only when an actual post-review covered visit occurs.
- **C.** Show both categories separately: overdue unused packages and actual overdue redemptions.

*Original recommendation:* C, different labels.

**Recorded answer — C.** Two **distinct dashboard query categories** must exist: **overdue/inactive with unused allowances** and **actual post-review covered visits** requiring admin review. They are not equivalent clinical/operational events. The results **remain persistently visible while their criteria remain true/unresolved**. They are **not time-based repeat reminders**, and no reminder scheduler was requested.

**Discussion correction:** The assistant repeatedly misstated Q09 as alerts for *new patient complaints*. User rejected such alerts: recording a new complaint is ordinary business, and should not generate an admin package alert. That tangent is **not** a new Q09 requirement.

## Q10. How often should the overdue warning interrupt the workflow?

**Original scenario.** A receptionist records several legitimate post-review treatments over the following week.

- **A.** Show confirmation before every post-review covered visit.
- **B.** Require confirmation once per package per calendar day, while still collecting the mandatory reason for each later visit.
- **C.** Confirm only the first post-review continuation, then show a persistent warning on subsequent days.

*Original recommendation:* A.

**Recorded answer — A.** **Every single post-review covered visit** requires an explicit confirmation and, for a receptionist, a per-visit reason. The earlier discussion about whether overdue dashboard queries persist was **not** the content of Q10; persistent dashboard queries are addressed in Q09.

## Q11. How should reusable justification suggestions be managed?

**Original scenario.** The receptionist frequently enters “Clinic closed for festival” across many patients.

- **A.** Browser-local history/autocomplete only.
- **B.** Centrally configured clinic presets maintained by admin.
- **C.** Both: clinic presets plus browser-local recent free-text suggestions.

*Original recommendation:* C. Actual reasons must always be stored against covered visits server-side.

**Recorded answer — A.** For MVP, use **browser-local autocomplete/history only**, not an admin-managed list of clinic-wide justification presets. The text actually submitted for each visit must still be retained with that committed covered visit. Do not confuse this feature with **closure reason** presets (Q15), which **were** chosen.

---

# C. Closure, reopening, and complaint lifecycle

## Q12. Can a manually Closed package ever be reopened?

**Original scenario.** Admin closes a package with six unused allowances. Next day, patient requests reinstatement.

- **A.** Closure is permanent; no reopening in MVP.
- **B.** Admin may reopen it with documented reason and full audit trail.
- **C.** Reopening only when original closure was erroneous, not routine commercial reconsideration.

*Original recommendation:* A for MVP.

**Recorded answer — A.** A **deliberately closed package is permanently closed**; patient requests cannot reinstate unused allowances. Future treatments require appropriate new entitlement. Error reversal is addressed separately by Q13 and was also rejected there.

## Q13. What happens when staff close the wrong package accidentally?

**Original scenario.** A patient has separate active packages for knee and shoulder complaints. Admin intends to close knee but mistakenly closes shoulder.

- **A.** No correction in MVP; admin must handle through a new purchase and separate financial action.
- **B.** Narrowly scoped admin-only reversal of mistaken closure, with audit record.
- **C.** Defer entire case to future generic correction module.

**Recorded answer — no reopening/reversal in MVP (A in substance).** User explicitly rejected even an admin-only reversal for erroneous closure. The defense is **strong, informed, multi-step destructive confirmation**: display patient, exact complaint/package, remaining allowances and forfeiture, irreversibility, and require positive confirmation. It must not be possible to lose the entitlement with an unnoticed single click.

**Important limit:** No specific new purchase, reimbursement, or financial correction sequence was approved here for the rare mistaken closure. Original option A proposes those as handling; the **approved part** is **no reversal**. Do not invent a complete financial-resolution policy.

## Q14. What happens if payment arrives after closure for nonpayment?

**Original scenario.** Package A is manually closed because patient has not paid. Five days later patient settles the entire invoice.

- **A.** A remains Closed; payment changes only financial balance.
- **B.** A automatically reactivates.
- **C.** A remains Closed but admin may explicitly reactivate it.

*Original recommendation:* A.

**Recorded answer — original nonpayment-closure premise rejected; A's non-reactivation consequence remains valid.** **Nonpayment by itself is not a reason to auto-close, or ordinarily close, an otherwise valid package.** Even a zero-paid package remains active under the original rule. Typical closure patterns described by user: (1) **patient explicitly requests termination**, permitting early admin closure; (2) patient **stops attending**, and a discretionary closure would generally occur only **after the review date**; (3) **mistaken registration**, a legitimate early administrative correction. Payment amount (zero/partial/full) does not dictate package closure.

If a package **has** validly been closed, **later payment does not reactivate it**; payment belongs to the old receivable. If treatment is to resume after terminal closure, register a **new package**, subject to separate eligibility and purchase terms. Do not infer automatic abandonment closure or any inactivity threshold.

**Reconciliation note:** Earlier baseline allows **admin closure anytime**; user's description above is the *intended business practice*, not an expressly enacted new technical restriction on every conceivable admin reason. Do not quietly block the admin from all other early closures without confirmation.

## Q15. What is the closure reason-entry format?

**Original scenario.** Admin closes package because patient abandoned therapy; receptionist closes another overdue package because patient wants replacement.

- **A.** Mandatory free-text reason only.
- **B.** Fixed reason dropdown with no custom reason.
- **C.** Suggested reasons with editable free-text field.

*Original examples:* `Patient discontinued`, `Early renewal`, `Made by mistake`, `Nonpayment`, `Treatment plan changed`, `Other`.  
*Original recommendation:* C.

**Recorded answer — C.** Mandatory **closure reason** supplied via **suggested choices plus an editable custom-text option**. The user stated this had already been discussed and wanted it treated as settled. The user did not explicitly approve the precise suggestion labels. Suitable candidates discussed were **patient requested termination**, **patient stopped attending**, **incorrect registration**, and **early renewal**, but **`Nonpayment` should not be assumed as an accepted ordinary closure preset**, in view of Q14. The reason records why closure occurred; it is not payment or refund proof.

## Q16. Should closed packages remain visible in ordinary patient history?

**Original scenario.** Patient returns six months after package was closed.

- **A.** Show old packages under separate Completed/Closed history section, including original allowance, usage, closure reason, and outstanding invoice link.
- **B.** Hide them from standard patient screen and show only in admin history/audit view.

*Original recommendation:* A.

**Recorded answer — A, explicitly accepted.** Keep **Completed** and **Closed** packages in normal patient history, distinct from active offerings, with original allowance, usage, closure reason, and associated invoice/outstanding balance link.

## Q17. What if a new complaint is required while an old package is active?

**Original scenario.** Patient has an active Routine knee complaint with twelve package allowances remaining. Clinician determines case should instead be Rehabilitation. Established clinical rule would require a new complaint rather than changing the original classification.

- **A.** Keep old package on old complaint. To terminate course, close it and forfeit unused allowances; new complaint needs its own package.
- **B.** Allow admin to transfer remaining allowances to new complaint.
- **C.** Permit narrow exception only when original clinical classification was incorrect.

*Original note:* A follows existing complaint/forfeiture rules; B/C introduce new exceptions.

**Recorded disposition — DROPPED FROM THIS DECISION ROUND. No A/B/C selection.** User said the reclassification scenario was not a meaningful operational case for this package specification and should be left to clinical discretion/outside scope. Do **not** turn this into an approved transfer or correction feature.

**Discussion that occurred *before* user rejected the scenario:** User rejected transfer between Routine and Rehabilitation offerings because they have different package prices. User tentatively sketched: close old package, decide any financial refund/waiver with old invoice according to actual payments, start a fresh complaint, and purchase a new package. User then reconsidered and explicitly said the scenario was wrong/not worth covering. **Therefore the tentative refund/waiver sequence is *not approved policy***. It would also intersect the still-unanswered refund limitations in Q22/Q23.

**Baseline remains:** No general allowance transfer; closing a package forfeits unused allowance, and complaints cannot be closed with active packages. This is not the same as having approved an edge-case reclassification workflow.

---
# D. Mistaken purchases, invoice waivers, and financial exceptions

**Status for this section:** Q19–Q26 have **not been answered** in this conversation. Do not infer approval from the user having mentioned refunds/waivers while thinking aloud about Q17. Closing an entitlement and correcting its financial record are not the same operation. Current MVP has invoice-level waiver but no general refund or void mechanism.

## Q18. What if a previously closed complaint is reopened?

**Original scenario.** A complaint was closed after its package reached Completed. Two months later, the same complaint is reopened clinically, assuming the future complaint module permits that.

- **A.** Reopening the complaint does not reactivate the old package. A new package must be purchased.
- **B.** If the old package was manually Closed with unused allowances, admin may decide whether to restore it.
- **C.** Do not support complaint reopening in the initial complaint-lifecycle MVP.

*Original caveat:* Conditional on complaint reopening becoming supported at all.

**Status — UNANSWERED.** The package finality principle (Q12–Q14) strongly points away from restoring a closed package, but the **complaint-reopening feature itself has not been decided in this discussion**. Do not record A or C as newly approved here.

## Q19. Must package purchases have their own invoices?

**Original scenario.** At the counter, a patient purchases a ₹4,500 package, receives a covered treatment, and owes a separate ₹350 examination fee.

- **A.** Issue a dedicated ₹4,500 package-purchase invoice and a separate treatment/examination invoice.
- **B.** Put the package purchase, treatment, and examination fee on one combined invoice.
- **C.** Allow both formats depending on checkout workflow.

*Original recommendation:* A. Isolates package-purchase corrections from legitimate examination and treatment charges; compatible with same counter interaction.

**Status — UNANSWERED.** ₹4,500 and ₹350 are **illustrative scenario figures**, not approved package-catalog prices.

## Q20. How should an entirely mistaken, unpaid package purchase be handled?

**Original scenario.** Admin registers a ₹4,500 package for the wrong patient. No treatment has been redeemed and no payment collected.

- **A.** Close the package with reason “Made by mistake”; admin separately waives unpaid purchase invoice with a reason.
- **B.** Close the package but retain collectible invoice balance until a future void mechanism exists.
- **C.** Require a genuine void/cancellation capability in MVP rather than using waiver.

*Original context:* A was the previously documented working direction, but not an explicitly accepted final procedure. It leaves an auditable waived invoice rather than erasing the sale.

**Status — UNANSWERED.** Q13/Q14 recognize mistaken registration as a potential reason for **closure**; they do **not** resolve the subsequent **financial invoice** correction method.

## Q21. What if an erroneous package purchase shares an invoice with legitimate services?

**Original scenario.** A ₹4,500 package was added incorrectly to an invoice that also contains a valid ₹350 examination charge.

- **A.** Prevent this situation by requiring a separate package-purchase invoice.
- **B.** Introduce selective invoice-line correction or voiding.
- **C.** Accept current invoice-level waiver limitation and defer correction.

*Original recommendation:* A, consistent with Q19. Waiving entire ₹4,850 would wrongly forgive valid ₹350 charge.

**Status — UNANSWERED.** Depends in part on Q19.

## Q22. What if a mistaken package has already been partially paid?

**Original scenario.** A ₹4,500 package was purchased by mistake; clinic collected ₹1,500 and ₹3,000 remains outstanding. Current invoice model can waive ₹3,000 residual but cannot return ₹1,500.

- **A.** Close package, waive residual, and handle actual refund outside MVP, explicitly accepting software cannot record refund correctly.
- **B.** Include minimal refund-recording capability in MVP.
- **C.** Defer paid mistaken-purchase correction until future module; do not pretend waiver is refund.

*Original warning:* Manual cash return must not be reported as if no return occurred.

**Status — UNANSWERED.** A transient Q17 discussion mentioned possible refunds or waivers, but that scenario was explicitly dropped. **No refund feature or manual reporting compromise was approved.**

## Q23. What if the entire mistaken package was already paid?

**Original scenario.** Patient paid ₹4,500 before error discovered.

- **A.** Close package and handle reimbursement outside software, accepting reporting limitation.
- **B.** Require minimal auditable refund workflow before supporting correction.
- **C.** Handle only by exceptional administrative process outside MVP and explicitly mark it unsupported in software.

*Original note:* Waiver cannot substitute for refund on already-PAID invoice with no collectible balance.

**Status — UNANSWERED.**

## Q24. What if the mistaken package has already covered treatments?

**Original scenario.** A package was registered against wrong complaint and has already covered two recorded visits with issued ₹0 therapy lines. Mistake discovered afterward.

- **A.** Preserve committed records and close package; correction outside MVP.
- **B.** Narrowly scoped admin correction for package-to-visit attribution with safe financial consequences.
- **C.** Require broader visit/invoice correction capability before launch.

*Original distinction:* Different from an unused unpaid mistaken purchase.

**Status — UNANSWERED.** Q13's refusal to reopen *closed packages* does not settle whether incorrect *visit attribution* can be corrected.

## Q25. Does waiving a valid package invoice affect its coverage?

**Original scenario.** Clinic deliberately gives a complimentary package by waiving its ₹4,500 purchase invoice. Package remains active.

- **A.** Coverage continues. Waiver changes receivable, not entitlement.
- **B.** Waiving package-purchase invoice must also close its package.
- **C.** Package-purchase invoice may only be waived after closing its package.

*Original recommendation:* A, consistent with payment-independent activation.

**Status — UNANSWERED as an explicit selection.** The previously settled independence of financial operations provides a strong constraint, but do not invent a fresh affirmative answer.

## Q26. Should a closure reason labeled “Refunded” require proof of actual refund?

**Original scenario.** Admin closes a paid package and chooses “Refunded”, although money has not yet returned.

- **A.** Permit free-text closure reasons but make clear they are descriptive, not financial proof.
- **B.** Use neutral reasons such as “Refund requested”/“Refund-related closure”; reserve “Refund completed” for verified financial record.
- **C.** Exclude refund-related presets until actual refund module exists.

*Original recommendation:* B as long-term rule; A potentially for MVP if evidence unavailable. Closure label must never alter collections reporting.

**Status — UNANSWERED.** Q15 approved **preset + custom closure reason UX**, not a verified-refund label policy.

---

# E. Package catalog, pricing, and configuration

**Status for section:** Q27–Q31 remain unanswered. Q03's permission to change an **individual purchased package's daily visit cap** does not answer catalog prices, negotiated sales prices, or access controls on catalog settings.

## Q27. What are the actual prices for all four offerings?

**Original catalog:**

| Category | 10-visit offering | 26-visit offering |
|---|---:|---:|
| Routine | ₹? | ₹? |
| Rehabilitation | ₹? | ₹? |

Provide current prices, or answer **“admin-configurable; actual figures to be entered during clinic setup”** if values not needed for specification. Earlier illustrative figures must not become clinic-approved prices.

**Status — UNANSWERED.** The ₹4,500 amounts used in earlier scenarios were examples, **not a final price list**.

## Q28. Who may change package offering prices and settings?

**Original scenario.** Receptionist notices long Rehabilitation package price is incorrect.

- **A.** Only admin may edit package prices, availability, and commercial configuration.
- **B.** Receptionist may edit prices under restrictions.
- **C.** Only owner-level accounts may change package pricing.

*Original recommendation:* A, matching existing master-service pricing permissions.

**Status — UNANSWERED.** Admin's control of **purchased-package daily cap** was agreed at Q03. This does not amount to approval of **catalog-wide pricing permissions**.

## Q29. Can staff override a package's catalog price during purchase?

**Original scenario.** A ₹4,500 package is being registered; clinic offers exceptional ₹500 discount.

- **A.** Sell at configured ₹4,500; admin waives ₹500 residual after ₹4,000 collected.
- **B.** Admin may set different purchase price before invoice issuance.
- **C.** No negotiated package prices/discounts in MVP.

*Original note:* A preserves invoice-line immutability while reporting ₹4,500 billed and ₹500 forgiven; B introduces overrides.

**Status — UNANSWERED.**

## Q30. Can the clinic issue a genuinely free package?

**Original scenario.** Owner gives staff or charity patient a 10-visit package at ₹0.

- **A.** Package can have zero purchase price when explicitly authorized.
- **B.** Always issue normal catalog price and use admin waiver.
- **C.** Free packages unsupported in MVP.

*Original distinction:* ₹0 purchase price is different from a covered ₹0 therapy-visit invoice line.

**Status — UNANSWERED.** The earlier rule “activation regardless of ₹0 collected” refers to **money collected**, not **a ₹0 selling price**.

## Q31. Can admin disable an offering for future purchases?

**Original scenario.** Clinic stops selling 10-visit Rehabilitation packages but existing patients retain valid purchased entitlements.

- **A.** Admin may disable new sales, existing entitlements unchanged.
- **B.** Offerings cannot be disabled in MVP; prices only modifiable.

*Original recommendation:* A. Disabling catalog offering should not revoke existing entitlements.

**Status — UNANSWERED.**

---

# F. Existing patients and deployment migration

These questions are conditional. If no preexisting packages are being carried forward, relevant follow-ons can be N/A. Do not make migration an artificial blocker without evidence.

## Q32. Are there already active packages outside this new system?

**Original scenario.** On deployment day, patients may have previously purchased packages recorded on paper, Excel, older software, or prototype database.

- **A.** Yes, real active patient packages must be carried forward.
- **B.** No, launching with no active historical packages to import.
- **C.** Only prototype/test data exists; may be discarded/rebuilt.

*Original note:* Determines real business-data migration versus technical schema migration.

**Status — UNANSWERED.** No verified fact about actual deployment data was established in this chat.

## Q33. If historical packages exist, how should they be introduced?

**Original scenario.** Patient purchased package 20 days before launch, used 12 of 26, with 14 remaining.

- **A.** Import original purchase date, actual usage, remaining allowances, status, commercial terms, without fresh sale.
- **B.** Admin manually registers opening package balance, preserving original entitlement info.
- **C.** Start all imports as new purchases on launch date.

*Original recommendation:* A or B; C wrongly resets review clock.

**Status — UNANSWERED / conditional on Q32.** Do not infer which dates a historic package would use under the newly clarified **purchase date versus start date** distinction.

## Q34. What if old package data is incomplete?

**Original scenario.** Paper register confirms purchase of 26-visit package but cannot reliably determine 13 versus 16 consumed treatments.

- **A.** Admin verifies and approves opening balance before migrating as usable.
- **B.** Best available estimate, explicitly marked estimated.
- **C.** Do not import unverifiable packages; handle manually outside new system.

*Original recommendation:* A. Never silently invent remaining allowances.

**Status — UNANSWERED / conditional.**

## Q35. What happens to packages purchased under older commercial terms?

**Original scenario.** Patient bought older calendar-month package with terms different from new fixed 30-day/26-visit model.

- **A.** Honor original purchased terms; migration preserves legacy entitlement where applicable.
- **B.** Convert everyone to new policy after explicit clinic approval.
- **C.** Do not migrate incompatible packages; settle obligations outside app.

*Original recommendation:* A, consistent with immutable terms.

**Status — UNANSWERED / conditional.** Q03 creates a **narrow approved mutable operational daily-cap setting**; do not generalize it into arbitrary retroactive changes to commercial terms.

## Q36. Must historical unpaid package balances also be imported?

**Original scenario.** Preexisting patient has 10 treatments remaining and unpaid ₹2,000 package balance.

- **A.** Import active entitlement and independently collectible opening balance.
- **B.** Import only package coverage; old debt stays in previous accounts/register.
- **C.** No financial migration because no historical balances.

*Original note:* Financial migration scope must not control entitlement activation.

**Status — UNANSWERED / conditional.**

---

# G. Other coverage, checkout, and operational edge cases

## Q37. Can packages ever cover separately billed standalone services?

**Original scenario.** Patient with active knee complaint package receives packaged knee therapy plus separately purchased standalone traction service.

- **A.** Package covers eligible complaint-therapy visit only; standalone traction independently chargeable.
- **B.** Some standalone services included when configured in offering.
- **C.** Standalone services covered only by special admin authorization.

*Original recommendation:* A for MVP, following separation of complaint therapy and standalone purchases.

**Status — UNANSWERED.**

## Q38. What warning should appear when a different complaint is selected?

**Original scenario.** Patient has active knee package but receptionist selects shoulder complaint for today's therapy.

- **A.** Nonblocking warning identifying other active package; continue ordinary shoulder billing.
- **B.** No warning; selected complaint independently determines coverage.
- **C.** Confirmation requiring acknowledgement that shoulder visit is not covered by knee package.

*Original recommendation:* C if not repetitively disruptive when treating multiple complaints routinely.

**Status — UNANSWERED.** Q05's **debt warning** does not answer this separate **complaint selection warning**.

## Q39. Should unpaid packages appear in admin Needs Attention separately from overdue review events?

**Original scenario.** Patient has active package with ₹4,500 outstanding but legitimately consumed five covered visits.

- **A.** Separate unpaid-package category in Needs Attention.
- **B.** Debt only in ordinary invoice/payment reports.
- **C.** Balance indicator in package view but no Needs Attention item.

*Original note:* Informational only, never automatic coverage stop.

**Status — UNANSWERED.** Under Q05, staff see a debt warning **during new package purchase**, and a general unpaid-invoice report already exists. The user deferred a **new delinquent-renewal oversight report**. None of this selects Q39's treatment of all unpaid packages in admin Needs Attention.

## Q40. What is the minimum safe response to an incorrect committed visit?

**Original scenario.** Receptionist records treatment against wrong complaint; invoice and package redemption are already committed. Current MVP lacks issued-invoice editing and general retroactive clinical correction.

- **A.** Accept MVP limitation: flag for manual investigation; committed record not changed.
- **B.** Narrow admin-only correction for visit/package attribution, with invoice integrity checks.
- **C.** Require full future correction/void framework before real production use.

*Original recommendation:* B for live clinic use if carefully designed; A risks unfixable integrity problem; C expands project scope.

**Status — UNANSWERED.** Do not equate Q07's **immutable justification note** or Q13's **non-reversible closure** with a decision about correcting **wrong-complaint visit attribution**.

---

# H. Engineering acceptance and verification requirements (not new owner questions)

The original questionnaire explicitly separated these from the owner's 40 choices. After business decisions are finalized, implementation should be checked against the actual application schema and code. A written rule is not evidence of implementation.

| Engineering work | Required verification |
|---|---|
| Lifecycle state machine | Registered/Active, Completed, Closed, review-due and overdue distinguished; payment independent. Include newly split purchase date and coverage start date. |
| Registration and checkout atomicity | No half-created purchase, missing invoice, or incorrectly committed first redemption. |
| Entitlement provenance | Every covered complaint-visit links to exact purchased package providing allowance. |
| Concurrency | Concurrent requests cannot both consume final allowance or exceed applicable daily cap. |
| Idempotency | Double clicks, retries and lost responses cannot create duplicate invoices/visits/redemptions. |
| Daily eligibility | Clinic-local dates; do not invent historic treatment-occurrence timestamp; compute first-coverage date from prior same-day covered usage. |
| Review arithmetic | Inclusive **package start date** (supersedes original registration date), Sunday-excluded short-package counting, 30 consecutive days for long offering, exact due/overdue boundaries. |
| Historical terms | Later price/catalog changes do not mutate prior purchase terms or issued invoice lines. **Explicit narrow exception:** admin may change purchased-package daily cap operationally, without edit audit in MVP. |
| Invoice correctness | Covered ₹0 lines, mixed covered/uncovered visits, independent fees, payment records. |
| Authorization | Closure timing, admin waiver, receptionist notes, admin daily-cap changes, catalog modifications and financial actions enforced server-side according to whichever questions become settled. |
| Complaint integration | Active-package complaint-closure restrictions hold under concurrency. |
| Historical migration | Do not import fabricated counts, obsolete fields, or assumptions as real entitlements. Migration policy still conditional/unanswered. |
| Reporting | Billed, collected, outstanding, waived separate; covered visits not fictional receipts. No new delinquent-renewal report approved. |
| Traceability | Closure reasons, post-review reasons, package actions, and real visit associations inspectable. **Do not assume a general audit-log table exists or is approved for new daily-cap edits.** |

# I. Scope boundaries / deferred concepts

The original November 2 patch also mentioned a general void/cascade module, refund infrastructure, persisted notification dismissal, attendance/holiday ledgers, automatic expiry buffers, simultaneous active packages for the same complaint, and broader correction workflows. Their appearance in older documents does **not** automatically put them in MVP.

The wider complaint-lifecycle project—automatic inactivity-based closure, future complaint reopening rules, and full clinical record correction—remains separate. Q17 was dropped from this package round. Q18 is unanswered and conditional. Q22–Q24 and Q40 remain the specific future conversation points for the real-life risks of deferring correction infrastructure.

## Integrity checks against the voice discussion

1. **Q01 overrides an older settled line.** Do not write both “review starts on registration” and “review starts on start date” as simultaneously effective. The latter is the latest explicit decision.
2. **Q03 has a temporal nuance.** The user said “2” and *then* said “default 1 for every package” and “admin can change later.” The operative specification is **configurable daily cap; default 1**, with optional admin-set 2, even on long offering. If intended default differs by offering, that particular point needs explicit confirmation.
3. **Q03 audit decision is deliberate.** Assistant recommended tiny change log; user rejected adding even that for MVP and accepted loss of edit provenance. Do not put it back in as mandatory work under H.
4. **Q04/Q13 require real warnings.** The no-reversal decision relies on meaningful package identity, remaining-allowance and irreversible-action confirmation, not only a generic warning.
5. **Q08 has no ping for admin-authorized post-review visits, including other admins.** Do not turn every admin action into an attention item; retain normal visit records.
6. **Q09 and Q10 were repeatedly confused by the assistant.** Correct numbering: Q09 = *dashboard categories*; Q10 = *confirmation frequency*. Dashboard queries persist; a post-review redemption requires fresh confirmation **each time**.
7. **Q11 and Q15 are different.** Q11 = browser-local suggestions for *post-review notes*. Q15 = *closure reasons* using presets plus custom text.
8. **Q14 does not introduce automatic debt enforcement or automatic absence closure.** Payment/collection is separate; clinician/clinic discretionary closure timing remains as described.
9. **Q17's tentative refunds were not a final business decision.** The user first worked through a hypothetical and then explicitly rejected the entire reclassification scenario for this round. Refund workflow remains unknown pending Q22–Q23.
10. **Q18–Q20 were incorrectly described by the assistant as already agreed.** In fact, the conversation stopped before asking or deciding them. Their source wording is present in the original user message and has been restored here, unselected.
11. **An invented retention question was withdrawn.** No note-retention duration was approved; the assistant's arbitrary three-year suggestion must not be included as a rule.
12. **An invented new-complaint admin alert was rejected.** Opening a normal new complaint merely records it; it is not a post-review package attention event.
13. **Debt-renewal reporting expansion was deferred.** Existing unpaid invoice reporting is known to exist; a proposed enriched report showing subsequent package purchases was *explicitly deferred*.

## Next discussion checkpoint

**Resume with Q18**, read its exact case aloud, and do not skip Q19 or Q20. Do not treat recommendations as approved answers, and do not re-ask decided Q01–Q16 without a concrete, newly discovered contradiction.

## Original source references (supplied by user, not reverified for this file)

- [Package decisions](https://github.com/omsrivastava512/clinic-demo-be/blob/main/study/module_2/decisions/packages.md)
- [November 2 discussion patch](https://github.com/omsrivastava512/clinic-demo-be/blob/main/study/module_2/decisions/history/clinic_packages_discussion_patch_2026-11-02_181036_IST.md)

*Reconstruction rule: explicit user statements overrule assistant suggestions; assistant summaries are not treated as fresh owner approvals. Uncertainty is labeled instead of invented away.*
