# Job Search Pipeline — Full Workflow

One-time setup, then a repeatable step-by-step pipeline from "found a posting" to "sent and tracked." Steps 0–3 are where AI does real work. Steps 4, 5, and 9 are deliberately yours — not gaps, design choices.

---

## PART A — One-time setup

**A1. Project setup.** Use a dedicated Claude project for this. Set two things once so they don't get retyped per message:
- **Custom instructions** = the rule sets from Part D below (Agent 1/2/3 rules), pasted once.
- **Project knowledge** = this file, plus your corrected profile facts (Appendix, below). Anything here is available to every future chat in the project automatically — a chat's contents alone are not.

**A2. Job alerts (zero AI, set once, run forever).**
- Naukri — free, up to 5 saved alerts, delivered to inbox.
- Wellfound, LinkedIn Jobs (filtered to <2 weeks old), Instahyre, Cutshort — each has an equivalent saved-search + alert feature.
- Glassdoor — also carries live postings, not just reviews; same alert setup applies.

**A3. Optional connectors.**
- **Gmail** — lets a chat search/categorize the alert emails landing in your inbox directly. Check your training/privacy toggle first (Settings → Privacy → Privacy Settings) — off means 30-day retention, on means up to 5 years for model training. Scope requests narrowly ("search only emails from these senders") rather than open browsing if you're cautious.
- **Firecrawl** — good specifically for crawling/mapping a company's entire careers page in one pass; doesn't unlock LinkedIn/X's login-walled content, since that's a platform restriction no crawler bypasses.
- Neither is required — the pipeline works with manual paste-in if you skip both.

**A4. The tracker.** Spreadsheet (or a build-it-later live tool) with columns: company, contact, channel, date sourced, date sent, follow-up date (= sent + 6 days), status (sent / replied / interview / rejected / follow-up-sent / closed), notes.

**A5. Set your actual thresholds** (used by Agent 1 — edit these to your real numbers):
- Experience cutoff for auto-reject: **2+ years required, with no fresher/junior track stated**
- Services-firm blocklist (auto-reject unless manually flagged for a separate volume-apply track): *Accenture, Infosys, Wipro, TCS, Capgemini, Cognizant, HCLTech, Persistent, Hexaware, Nagarro, Zensar, Birlasoft, Iris Software, CGI, UST, Photon, Synechron* — edit freely
- Salary floor: **[fill in]**
- Location constraint: **[fill in — remote-only / specific cities / none]**

---

## PART B — The recurring pipeline, step by step

### Step 0 — Sourcing (multiple parallel lanes, all feeding one intake)
Pick any combination; all of them funnel into the same Step 1 regardless of source:
- **0a. Passive — alerts land** in inbox from A2, no action needed to generate them.
- **0b. Manual LinkedIn/Twitter search** — you run the filtered search yourself (logged-in access reaches further than any outside tool), paste the results in.
- **0c. Grok/X** — you run the recency-filtered search on your own X/Grok account (same reasoning as 0b — an outside AI can't reliably reach X's content), paste results in.
- **0d. Firecrawl** (if connected) — batch-crawl your target company list's careers pages for new listings.
- **0e. Paid scraper tool** (your choice, if you add one) — same pattern: its output gets pasted/fed in.

**Output of Step 0, regardless of lane:** a raw batch of {company, role, posting text or link, source, date}.

### Step 1 — Triage (Agent 1: Triage Filter)
Every posting from any Step 0 lane gets checked against your A5 thresholds — mechanically, not by AI opinion. See Part D for the full prompt.
**Output:** three buckets — **PASS** (→ Step 2), **FLAG** (your manual call), **REJECT** (discarded, with the specific rule cited).

### Step 2 — Research (Agent 2: Research Brief)
PASS items get compressed into one fact about the role/team/stack — never mission or culture language. SIGNAL mode (default, one fact) for volume; DEEP mode for your 5-10 priority targets.

### Step 3 — Draft (Agent 3: Message Draft)
Takes Agent 2's output + your fixed "About Me" block (Appendix) → produces a draft. Hard rules baked in: no flattery ever, one fact only, 70% about you / 30% the fact, exactly one small clear ask, self-checked before it's handed back.

### Step 4 — Human review (manual, by design)
You read the fact Agent 2 pulled and the draft Agent 3 wrote, catch anything off, edit into your own voice. This is the one deliberate checkpoint in the whole system — collapsing it removes the only thing standing between a wrong "fact" and a message that's already sent.

### Step 5 — Send
You send it, wherever it needs to go (LinkedIn DM, email, freelance reply).

### Step 6 — Log
Add the row to the tracker: company, contact, channel, date sent. Follow-up date auto-calculates as sent + 6 days.

### Step 7 — Follow-up trigger (deterministic, date-based)
When a tracker row hits its follow-up date with no reply logged, it goes back through Agent 3 with type = follow-up, referencing the original message briefly. No new research needed.

### Step 8 — Status update on reply
Manual by default (you see the reply, update the sheet). If Gmail is connected, a chat can check for new replies on your ask and flag which tracker rows to update.

### Step 9 — Conversion (manual, by design — this was always yours)
Interview prep, DSA, the actual conversation. Nothing above touches this; it exists so the funnel above keeps feeding it real opportunities to convert.

---

## PART C — Automated vs. manual, at a glance

| Step | Who does it |
|---|---|
| 0. Sourcing | Mix — alerts fully automated; LinkedIn/X/scraper need you to pull, AI processes what you paste |
| 1. Triage | AI, against your fixed rules |
| 2. Research | AI |
| 3. Draft | AI |
| 4. Review | **You — by design** |
| 5. Send | **You** |
| 6. Log | You (or auto, if tracker is a connected tool later) |
| 7. Follow-up trigger | Deterministic (date math), draft generated by AI |
| 8. Reply status | You, or AI-assisted if Gmail is connected |
| 9. Conversion | **You — by design** |

---

## PART D — The agent prompts

### Agent 1 — Triage Filter
```
Classify this job posting against fixed rules. Do not use judgment beyond these rules — if none of the auto-reject rules fire and none of the flag conditions apply, it passes.

AUTO-REJECT if any are true (cite which one fired):
- Requires 2+ years professional experience with no fresher/junior track stated
- Company is on this list: [Accenture, Infosys, Wipro, TCS, Capgemini, Cognizant, HCLTech, Persistent, Hexaware, Nagarro, Zensar, Birlasoft, Iris Software, CGI, UST, Photon, Synechron] — unless marked as a volume-apply exception
- Primary/only stated stack is backend with zero frontend/React/TypeScript mention
- States a salary ceiling below: [fill in]
- Location conflicts with: [fill in]

FLAG for manual review if:
- Company type/identity is unclear from the text
- "Full stack" mentioned without specifying backend tech

Otherwise: PASS.

Output: verdict (PASS / FLAG / REJECT), the specific rule that fired if any, one line of reasoning.

Posting:
[paste]
```

### Agent 2 — Research Brief
```
Compress research on this role for outreach personalization. Don't do anything else.

Mode: [SIGNAL (default) / DEEP (priority targets only)]

Hard rules:
- Extract facts about the ROLE, TEAM, and TECH STACK only.
- Never extract mission statements, "culture," "values," or vision language, even if present — not usable as personalization.
- Only use facts explicitly present in the material below. If none exists, say "no usable fact found" — don't invent one.
- SIGNAL mode: exactly ONE fact — the most specific technical or role detail available.
- DEEP mode only: may add one verifiable business fact (funding stage, launch, team size) if explicitly stated.

Give me:
- The fact (or facts, DEEP mode)
- Fit check: mismatch for a React/TypeScript/Postgres-Supabase fresher? Flag honestly.
- Right-contact note: who to actually message, if the material suggests it

Raw material:
[paste]
```

### Agent 3 — Message Draft
```
Draft a short cold outreach message. Use only what's given below.

Type: [LinkedIn DM / hiring-manager email / freelance offer / follow-up]

Absolute rules — check every draft against these before returning it:
- No flattery, ever: no "love what you're doing," "inspired by your mission," "dream company," or variants.
- No invented achievements, no filler ("hope this finds you well"), don't oversell experience level — I'm a fresher, say so plainly if relevant.
- Use ONE fact from "About them" to prove this isn't generic — must be specific enough it couldn't be copy-pasted to a different company. If no usable fact, skip personalization rather than force one.
- ~70% about what I bring and want, ~30% or less the personalized fact.
- End with exactly ONE small, specific, easy-to-answer ask.

About me:
[paste Appendix block]

About them: [paste Agent 2's output]

Format:
- LinkedIn DM: 3-4 sentences
- Email: under 150 words, include subject line
- Freelance offer: lead with the offer, not background
- Follow-up: 2-3 sentences, briefly references the first message

Self-check against the rules above before returning the draft; flag anything cut to comply.
```

---

## Appendix — About Me (reusable block)

- Fresher, no full-time job history — MCA graduate, one internship at Sorreal Systems (built a schema-introspecting PHP CRUD tool, and a food-ordering/wallet system used by ~3,000 students and ~60 faculty)
- Building a multi-location clinic management system for one paying physiotherapy client — React/TypeScript frontend, Postgres/Supabase with multi-tenant row-level security
- 6 merged PRs on OpenMRS, 100% acceptance rate on substantive changes (patient chart/visit notes features)
- Looking for: [fill in — entry-level frontend/product roles / this specific role / a free 20-min schema review]
- Link: [fill in — OpenMRS PR / project case study / GitHub]
