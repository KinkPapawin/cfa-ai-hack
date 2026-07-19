# Thesis Ledger — Handoff Brief for the Build Team

Read this first. It's written so a second builder can pick up the project cold, with zero prior context, and start writing code without re-deriving decisions that are already locked.

## 1. What we're building, in one paragraph

**Thesis Ledger** monitors whether the *reasons* behind a stock purchase are still true — not whether the price moved. Research theses get converted into falsifiable claims (`metric + operator + threshold + horizon`). Evidence (filings, news, price data) is collected continuously against each claim, and each claim gets a status: **INTACT / AT_RISK / BROKEN / STALE**. The system is recommendation-only — it never decides anything; every decision is signed off by a human (a Portfolio Manager). Client case for the pitch: **Alpha Fund (InnovestX)**. This is a CFA × AI hackathon submission.

Positioning line, use it verbatim on slides/UI: *"Thesis Ledger doesn't decide for the Portfolio Manager — the PM is always the one accountable for the final investment call (fiduciary duty, CFA Institute Standard III(A) — Loyalty, Prudence & Care). The system's job is only to raise a flag with the source evidence attached, for a human to check."*

## 2. Real constraints — read this before estimating anything

- **Build team: 2 people, 3 days.** Not the 5-day/larger-team plan implied by some of the older docs below — those are superseded.
- **A visible UI is mandatory.** There is no backend-only fallback under any circumstance.
- Budget: ~3,000 THB in API spend. Plenty, because scope is only 2 companies.
- Demo runs on a laptop. Not live/interactive — it's scripted/pre-loaded replay. This was chosen deliberately to cut engineering risk on the hero screen.

## 3. Where the actual requirements live (read in this order)

1. **`.claude/skills/thesis-ledger/references/requirements.md`** — **the locked, final scope.** Thai-language, 8 sections: scope, data/claims model (seed companies, tier policy), guardrails (exact traffic-light rule with numbers), demo/eval approach, UX rules (banned words, freshness-confirm), infra, the full Day 1/2/3 build plan with per-person tasks, and the two items that are still genuinely open (see §7 below). **If anything conflicts between this file and the two files below, this file wins.**
2. **`.claude/skills/thesis-ledger/SKILL.md`** — one-page product summary, client context, locked decisions, guardrails, rehearsed Q&A for pitch. Good first read for orientation.
3. **`.claude/skills/thesis-ledger/references/architecture.md`** and **`references/frontend-spec.md`** — the original 5-day/6-company spec. Still useful for data model and pitch material, but their build-plan/seed-count sections are explicitly superseded by `requirements.md` (marked with ⚠️ inline).
4. **`Fund_Manager_User_Journey_v2.md`** (repo root, Thai) — the original user-research artifact explaining *why* this product exists (the gap: every existing monitoring trigger fires on price, none fire on "the story changed"). Background/pitch material, not implementation detail.

## 4. The four visual artifacts in this folder

These were built during the design phase, after requirements were locked, to work out UI/architecture decisions visually before writing app code. **Open each one directly in a browser** (double-click, or drag into a tab) — they're self-contained HTML, no server needed.

| File | What it is | Use it for |
|---|---|---|
| **`01-app-flow.html`** | End-to-end flow: data/pipeline (thesis note + evidence → 4 pipeline stages → verdict → decision), the screen-to-screen user journey, and the 3-day build timeline mapped to who does what. | Understanding how everything connects before you touch code. Start here if `requirements.md` alone doesn't make the shape of the system click. |
| **`02-system-architecture.html`** | Layered architecture (Client → API → 4-stage Processing Pipeline → Data → External Services), a component→tech-stack table marking what's **locked** vs **still TBD at design time** vs **cut from this build**, and a full request trace for one Drop & Trace run (PDF drop → API → background task → PyMuPDF/bge-m3 → adjudicator → Postgres write → UI update). | Deciding how to structure the backend. Notable: it flags the frontend framework and the realtime channel (SSE vs polling) as **not yet decided** — that's a real open decision for whoever builds the frontend, not an oversight. |
| **`03-design-system.html`** | The actual design tokens (CSS custom properties for light/dark, color palette incl. the 4 verdict colors, type system: serif for headers / system sans for body / monospace for data), plus a literal `globals.css` and `tailwind.config.ts` you can copy in. Design principles: quiet-by-default, status readable in 1 second, evidence as a first-class citizen, professional density (not consumer-app spacing). | Skinning the real app — this is meant to be copy-pasted into the actual frontend, not just looked at. |
| **`04-product-prototype.html`** | The actual 4-screen prototype: **Portfolio Board**, **Drop & Trace**, **Thesis Evidence** (was "Thesis Card"), **Verification Queue** — built using the design system above, with realistic MWG/CPALL data in the layout. **No longer a static mock** — it's been iterated on past the original design-phase snapshot and now has real (if hardcoded) interactivity: data-driven overview stats, sortable/filterable cards, expand/collapse, working tabs. See §11 below before assuming it still matches the original design-phase description. | This is the closest thing to "what the finished demo should look like." Treat it as the visual spec for frontend implementation — build the real screens to match this, don't redesign from scratch. |

Read them in the numbered order (01 → 04): flow, then architecture, then design tokens, then the actual screens.

## 5. Locked decisions — do not re-open without new evidence

- Pitch scope is narrowly **"Thesis Ledger"** — a broader "Data Ledger" is a roadmap slide item only, not in this build.
- Unit of knowledge is a **structured Claim**, not a document — this is a Claim Store, not a document/data lake.
- **Postgres + pgvector**, explicitly *not* Databricks (wrong scale, wrong bottleneck, licensing forbids bulk-storing purchased data, no data-eng team).
- The demo **never touches licensed data** — sidesteps ToS risk entirely.
- **Seed set: MWG (Vietnam, the money-shot company) + CP All / CPALL (Thailand, chosen for easy public-data sourcing).**
- Evidence layer (filings/news/price) = **real public data** for both companies. Thesis/research-note layer = **mocked by the team** (tier=inhouse) — there's no real in-house note to use. This split keeps grounding credible without ever touching licensed data.
- Slack/LINE notification is **cut from this 3-day build** — it's Future Requirement FR-1, not part of the demo.
- Positioning is a **"thesis-health nowcast,"** never a price predictor or investment-advice engine.

## 6. Guardrails — these are scored on feasibility, not optional polish

- **Grounding is mandatory**: no `page_or_span` → the item does not count as evidence.
- **Abstention over guessing**: nothing found → `NO_DATA`, never a guess.
- **Adversarial check**: a second model argues against a claim's verdict; disagreement routes to the Verification Queue for a human.
- **Human veto everywhere**: every `decision.decided_by` is a person; the decision log is append-only.
- **Tier policy**: `tier=licensed` documents can never be sent to an external LLM or persisted raw (not used in this build at all, but the schema reserves the field).
- **`as_of` vs `knowledge_date`** always kept separate — prevents lookahead bias, this is what makes the MWG "we saw it before the price dropped" claim legitimate rather than hindsight-fitted.
- **STALE is distinct from INTACT** — confidence decays over time without fresh evidence. The system must never read as "safe" when it actually means "nobody checked."
- **Traffic-light rule (exact, from requirements.md §3):**
  - 🟢 INTACT — value satisfies threshold **and** (≥2 independent sources **or** 1 official source) **and** no contradicting evidence.
  - 🔴 BROKEN — evidence with a clear page/span contradicts the threshold in the negative direction.
  - 🟡 AT_RISK — adversarial models disagree **or** the value sits within ±20% of the threshold.
  - ⚪ STALE/NO_DATA — no fresh evidence within the decay window (≈100 days for quarterly-filing-based claims, ≈30 days for news/event-based claims).
- **Banned words in any verdict/alert copy**: "buy", "sell", "recommend", "guarantee" (ซื้อ/ขาย/แนะนำ/การันตี) — use neutral phrasing like "thesis is in BROKEN status — REVISIT."

> ⚠️ **Label mismatch, not a spec change — updated twice, read this not §11's history.** `04-product-prototype.html` currently *displays* the red state as **RISK** and the amber state as **REVIEW** (status set: `INTACT / RISK / REVIEW / STALE`) — not the `BROKEN`/`AT_RISK` wording used above and in `requirements.md` §3. It went through two rounds of renaming during prototype iteration (`BROKEN→AT_RISK→RISK`, `AT_RISK→NEED_REVIEW→REVIEW`); treat **RISK/REVIEW as the current name**, full stop — don't resurrect the intermediate `AT_RISK`/`NEED_REVIEW` pairing. The underlying rule (thresholds, tolerance bands, decay windows, color meaning) is unchanged throughout — only the words on screen moved. Whoever wires the backend should pick one vocabulary and make the schema, the guardrail doc (this file + `requirements.md` §3), and the UI agree before build — don't silently carry three different wordings forward. See §12.

## 7. Two things that are still genuinely unresolved

These are called out explicitly in `requirements.md` §8 and are **not** design bikeshedding — they're real risk items.

1. **B1 — Falsifiability test (top risk).** Nobody has actually tried extracting claims from a real Alpha Fund research note yet. Before locking in Day 1 engineering, run a manual 30–60 minute test: pull a real note, manually extract 3–5 claims in `metric + operator + threshold + horizon` form. The result determines whether a co-authoring/correction UI needs to be added to scope, or whether straight LLM extraction is good enough.
2. **D2 — Money-shot verification owner.** Someone needs to be named (one of the two builders, or someone else on the wider team) to confirm the MWG "marked BROKEN before the price actually dropped" scenario is genuinely reproducible under correct `knowledge_date` discipline — not hindsight-fitted by someone who already knows how the MWG story played out. Recommended timing: Day 1–2, before it's baked into the scripted demo.

Whoever picks this up should treat resolving these two as pre-Day-1 or Day-1 work, not afterthoughts.

## 8. Build plan (from requirements.md §7 — the authoritative version)

Hero screen: **Drop & Trace**. Secondary: **Portfolio Board** (real DB-backed). **Verification Queue** as a static snapshot only (to show the guardrail exists). **Thesis Card cut first** if time runs short.

- **Day 1 (heaviest — data sourcing):** run the B1 test first (go/no-go on scope). Then in parallel: schema + seed DB + entity map + real MWG public data (person 1); real CPALL public data + mocked inhouse thesis note + claims for both companies + start extraction prompt (person 2). DoD: DB seeded for both companies, at least 1 verdict computed from real data.
- **Day 2 (core loop + hero screen):** Drop & Trace wired to the real pipeline, scripted/replay (person 1); adjudicator — deterministic comparator + real adversarial check on MWG's load-bearing claim + traffic-light rule (±20% band + decay) + per-claim freshness nudge (person 2). DoD: MWG money-shot scenario works correctly (BROKEN before the real price drop, correct `knowledge_date`); CPALL shows a contrasting verdict so Portfolio Board looks varied.
- **Day 3 (integration, polish, rehearsal):** Portfolio Board (minimum), cite button, static Verification Queue, banned-word sweep, accountability line in product/slides. Rehearse the scripted demo ≥3 times, fix only show-stoppers — no new scope. Explicitly cut: Slack/LINE, any live demo path, live Verification Queue, formal backtest/eval framework, Thesis Card if time is short.

## 9. Tech stack (decided, not yet built)

- **Postgres + pgvector** for storage and multilingual embedding search.
- **PyMuPDF** for PDF parsing, tracking page + character offsets (needed for grounding/traceback).
- **Claude API** with structured JSON output for claim/assertion extraction and the adversarial check.
- **bge-m3** embeddings for multilingual (th/vi/id/zh/en) search.
- **FastAPI** backend with background tasks, streaming extraction results to the UI.
- **Frontend framework and the realtime channel (SSE vs. polling) are still open** — see `02-system-architecture.html` — this is a real decision for whoever starts frontend work.

## 10. Core data model

`thesis` (per-stock reasoning) → `claim` (metric, operator, threshold, horizon, `is_load_bearing`) → `verdict` (status, `as_of`, `knowledge_date`, confidence) → `decision` (HOLD/ADD/TRIM/EXIT/REVISIT, human-signed, append-only, references the triggering verdict).

Support tables: `document` (`tier`: public/licensed/inhouse, `can_send_to_llm`), `chunk` (text + embedding + page), `price_daily`, `source_health` (freshness monitor), `entity_map` (multilingual ticker/company alias mapping).

## 11. Prototype iteration log — changes to `04-product-prototype.html` since the design phase

The design-phase snapshot described in §4 got iterated on afterward, in-browser, based on direct feedback on screenshots. This section is the diff summary so a builder doesn't have to re-read the whole file to find out what moved. Everything below is **frontend-only, hardcoded/mock data, no backend behind it** — same "pre-code" status as the rest of this repo.

**Portfolio Board:**
- Added a **Portfolio Overview panel** above the table: 3 stat tiles (Positions / needs-check data points / average coverage), a tab strip (one tab per position — clicking jumps to and briefly highlights that row, auto-expanding the collapsed group if needed), and a single status donut + legend (count of positions per status, not AUM-weighted — kept deliberately plain per feedback, no toggle).
- Added a **"Claim Checking" module** — a `check {n}%` badge per row, intentionally a *separate axis* from thesis-health status (it answers "how much of this position's evidence is still unverified," not "is the thesis still true"). Implemented as one isolated `ClaimChecking.compute()` function reading a `POSITIONS` data array, specifically so the blend formula can be retuned later without touching markup.
- Replaced the **"Load-bearing claim" column** with **"Sector vs Industry (QoQ)"** — country, industry, and a company-vs-industry-sector QoQ delta (pp) — for every row.
- Removed the ERAA freshness-confirm nudge row, and removed the two long explanatory callout blocks (traffic-light logic + "quiet by default" P1 note) — judged as clutter for a first-glance home page.
- Subhead replaced with a tagline: **"Thesis Ledger, Identify What Matters Faster with AI-Powered Research."**
- **CPALL's status changed from INTACT to NEED_REVIEW** (mock judgment call, not backed by new evidence) so the board shows 2-needs-attention instead of 1; BBCA/MAP/HPG are unchanged.

**Thesis Card → renamed "Thesis Evidence":**
- The ticker/exchange/sector meta line under the stock name was replaced with a plain **"Thesis evidence"** section-bar label.
- The old numeric Claims list (claim vs. threshold comparisons) was **removed entirely** and replaced with **3 "reason cards"** — the qualitative story for why the position was bought, each with Overview / Why we bought / Sell-or-take-profit criteria, collapsed by default (click to expand). Content is invented/plausible, pending a real thesis note (see B1 in §7 — this makes that test more urgent, not less, since the UI now assumes richer qualitative thesis content exists).
- Each reason card has: a status + conviction-star badge, a "?" tooltip explaining what the badge/stars mean, an Edit button (visual toggle only, no persistence), and a per-evidence "→ Reference source" link.
- Added a **Priority & filter toolbar** (filter by load-bearing / needs-review, sort by priority/confidence/conviction) above the reason cards.
- Added a **"Show more thesis support"** affordance below the 3 cards — expands to a thin "No data available" since there's no 4th reason yet; this is the intended empty-state pattern once real thesis extraction produces a variable number of reasons.

**Terminology rename (applies file-wide, all 4 screens, round 1):** the red status label changed **BROKEN → AT_RISK**, and the amber status label changed **AT_RISK → NEED_REVIEW**. Colors and underlying logic are untouched — this was purely a copy/wording change requested during iteration. **This was superseded again in the very next round — see §12.** Don't build against `AT_RISK`/`NEED_REVIEW`; see the ⚠️ callout in §6 for the current name.

## 12. Second iteration pass — Thesis Evidence support-point redesign + final terminology

This is a second, later editing session on top of §11 — same file, same "frontend-only, mock data" caveat applies.

**Thesis Evidence — the 3 reason cards were restructured again**, replacing the flat Overview/Why-we-bought/Sell-criteria layout from §11 with **numbered "support points"** (e.g. `1.1`, `1.2` under reason 1; a reason can have one or several). Each support point is now a two-column comparison:
- **Existing** (`ข้อมูลเดิม`, green) — the original evidence/reasoning, with its own status pill (a support point's status can differ from its parent reason card's).
- **Upcoming data** (`ข้อมูลที่ขัดแย้งกัน / ข้อมูลใหม่`, red) — new or conflicting evidence that challenges the existing reasoning, or "รอข้อมูลเพิ่ม" (waiting on more data) if there isn't any yet.
- An **"◆ adversarial disagreement"** tag when the two sides genuinely conflict.
- Both columns get their own **"◇ Source finder"** — reference links now styled as clickable blue chips (solid `accent-tint` background, rounded, bold) specifically so they read as clickable, not as plain text.
- A **"+ Add new data"** button per support point (demo-only affordance — flashes a note, no real upload).

**Data reviewer moved from once-per-reason-card to once-per-support-point**, sitting directly under that support point's "+ Add new data" (so every support point — not just the reason as a whole — gets its own review checkpoint). Went through two design passes:
1. First pass: reviewer header text was `Data reviewer · คนที่มารีวิว`, turning amber with a `⚠ ต้อง review` flag when that support point had an adversarial-disagreement tag.
2. **Final pass (current state):** simplified to a plain **"Data reviewer"** label, and **all** reviewer headers — regardless of disagreement — use one uniform **soft light-purple background** (`#EAE3F5` bg / `#4C3E78` text, hardcoded — no design-token equivalent exists yet, see note below) that reads as clearly softer than the black "Support point" header bar above it. The per-item amber/flag distinction was dropped entirely; don't reintroduce it without asking — it was a deliberate simplification, not an oversight.

Each reviewer block: reviewer name (pre-filled `ปวริศ (PM)`, with an "auto-filled จากบัญชีที่ login อยู่" note — realistic stand-in for "don't make the user retype who they are," not literally an IP-based lookup), a department field, three status buttons — **Accept Existing / Accept Upcoming / Edit manually** — that stamp an append-only timestamp on click (`decided_by` is always a person, consistent with the guardrail in §6/§9), and a **"+ Add reviewer"** to add more rows. All of this is generated from one JS function per support point (`.sp` element + its `data-sp` id), not hand-duplicated markup — retune the template once, every support point updates.

**Edit button → dropdown**, not a toggle: clicking it now opens a small menu — **"📎 Add file / picture"** or **"✏️ Edit text manually"** — instead of flipping a single editing flag.

**"?" tooltip now has real copy**, not a placeholder — one shared legend (built once in JS, reused on every reason card) defining what each of the four statuses means and routes to (INTACT / RISK / REVIEW / STALE) plus what the ★ rating means (team's conviction *at the time the thesis was set*, not a live number).

**Load-bearing tag removed** from the reason-card header (still tracked as data for the "Load-bearing" filter chip, just not rendered as a visible pill anymore) — judged as something a non-specialist reader wouldn't parse at a glance.

**Terminology rename, round 2 (final, current — supersedes §11's round 1, file-wide, all 4 screens):** the red label changed **AT_RISK → RISK**, and the amber label changed **NEED_REVIEW → REVIEW**. Current status set is **`INTACT / RISK / REVIEW / STALE`**. Same as round 1: colors, thresholds, and underlying logic are untouched, purely a copy change. See the ⚠️ callout in §6 — treat this as the name going forward.

> ⚠️ **Loose end for whoever builds the design-token file next:** the new purple reviewer-header color (`#EAE3F5` / `#4C3E78`) was added directly as a hardcoded hex in `04-product-prototype.html` — `03-design-system.html`'s token set has no purple ramp. If "Data reviewer" stays purple in the real build, add it as a proper token there instead of copying the hex.
