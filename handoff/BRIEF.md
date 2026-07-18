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
| **`04-product-prototype.html`** | The actual 4-screen prototype: **Portfolio Board**, **Drop & Trace**, **Thesis Card**, **Verification Queue** — built using the design system above, with realistic MWG/CPALL data in the layout. | This is the closest thing to "what the finished demo should look like." Treat it as the visual spec for frontend implementation — build the real screens to match this, don't redesign from scratch. |

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
