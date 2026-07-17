# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status: pre-code planning stage

This repo is currently a **hackathon planning repo, not a scaffolded application**. There is no package.json, pyproject.toml, Makefile, CI config, source directory, build tooling, linter, or test suite — none of that exists yet. Do not guess at commands or invent a project structure. Once the app is actually scaffolded (see planned stack below), update this file's Commands section with the real build/lint/test/dev commands.

## What this project is

**Thesis Ledger** — a CFA × AI hackathon product for the Alpha Fund (InnovestX) case. It monitors whether the *reasons* behind a stock purchase are still valid, not just whether the price moved. Research theses are converted into falsifiable claims (`metric + operator + threshold + horizon`), evidence is continuously collected against each claim, and each claim gets a status: **INTACT / AT_RISK / BROKEN / STALE**. The system is recommendation-only — every decision requires a human sign-off. It is explicitly positioned as a "thesis-health nowcast," not a price predictor or investment-advice engine (precedent: BlackRock Aladdin Copilot).

## Where the real spec lives

The full product spec is captured in a Claude Skill at `.claude/skills/thesis-ledger/`:
- **`SKILL.md`** — product summary, client context, locked decisions, guardrails, rehearsed Q&A. Read this first for any Thesis Ledger / Alpha Fund work.
- **`references/architecture.md`** — data model, pipeline stages, guardrail implementation, risks, build-plan phasing, regulatory references. Read before backend/data work.
- **`references/frontend-spec.md`** — the 4 screens, traffic-engine color logic, tech stack, 5-day build order. Read before UI/demo work.

`Fund_Manager_User_Journey_v2.md` (repo root, Thai) is the underlying user-research artifact — the fund manager's investment lifecycle and the AI-tooling gaps that motivated this product. Useful background for pitch/positioning work, not implementation detail.

## Locked decisions — do not re-open without new evidence

- Pitch scope is narrowly **"Thesis Ledger"**; a broader "Data Ledger" is a roadmap item, not part of this build.
- The unit of knowledge is a **structured Claim**, not a document — this is a Claim Store, not a document/data lake.
- Database is **Postgres + pgvector**, explicitly *not* Databricks (wrong scale, wrong bottleneck, licensing forbids bulk-storing purchased data, no data-eng team on hand).
- The demo **never touches licensed data** — avoids ToS risk entirely.
- Positioning is a **thesis-health nowcast**, never framed as a price predictor or investment advice.

## Guardrails — implementation-binding, not optional

These are scored on feasibility and are load-bearing for the pitch, not nice-to-haves:

- **Grounding is mandatory**: no `page_or_span` → the item does not count as evidence.
- **Abstention over guessing**: nothing found → `NO_DATA`, never a guess.
- **Adversarial check**: a second model argues against a claim's verdict; disagreement routes to the Verification Queue for a human.
- **Human veto everywhere**: every `decision.decided_by` is a person; the decision log is append-only.
- **Tier policy**: `tier=licensed` documents can never be sent to an external LLM or persisted raw.
- **`as_of` vs `knowledge_date`** are always kept separate, to prevent lookahead bias in backtests.
- **STALE is distinct from INTACT**: confidence decays over time without fresh evidence — the system must never read as "safe" when it just means "nobody checked."

## Planned tech stack (decided, not yet built)

- **Postgres + pgvector** for storage and multilingual embedding search.
- **PyMuPDF** for PDF parsing, tracking page + character offsets (needed for traceback/grounding).
- **Claude API** with structured JSON output for claim/assertion extraction.
- **bge-m3** embeddings for multilingual (th/vi/id/zh/en) search.
- **FastAPI** backend with background tasks, for streaming extraction results to the UI.
- **Slack webhook** for the alert demo.
- Office add-in / browser extension is explicitly cut from MVP scope.

## Core data model

`thesis` (per-stock reasoning) → `claim` (metric, operator, threshold, horizon, `is_load_bearing`) → `verdict` (status, `as_of`, `knowledge_date`, confidence) → `decision` (HOLD/ADD/TRIM/EXIT/REVISIT, human-signed, append-only, references the triggering verdict).

Support tables: `document` (`tier`: public/licensed/inhouse, `can_send_to_llm`), `chunk` (text + embedding + page), `price_daily`, `source_health` (freshness monitor), `entity_map` (multilingual ticker/company alias mapping, ~30 manual entries).

Pipeline: (1) thesis extraction (LLM + JSON schema + human confirmation UI) → (2) evidence harvester (filings/news/field notes → document/chunk) → (3) adjudicator (deterministic comparator for numeric claims; LLM + adversarial check for qualitative claims) → (4) decision matrix (thesis status × price → recommendation for a human).

## Planned frontend (4 screens)

- **Portfolio Board** — traffic-light overview (🔴🟡🟢⚪), sorted worst-first, unchanged names collapsed. Must stay quiet by default — false alarms kill the system's credibility.
- **Drop & Trace** ⭐ — the star demo screen: drop a PDF, watch assertions extract live, click one to see the PDF highlight side-by-side with the claim/evidence it matched.
- **Thesis Card** — per-stock drill-down: claim status, linked evidence, verdict history.
- **Verification Queue** — human adjudication for claims the adjudicator is unsure about or where the adversarial check disagreed.

Traffic-engine color logic: 🟢 requires evidence from 2 independent source tiers; 🔴 requires a full traceback to page + character offset; unfound is ⚪ (`NO_DATA`), never 🟢.

## Build plan phasing

Phase 0 (schema, seed DB, entity map, mock research papers) → Phase 1 (core pipeline stages 1–3 — must not cut) → Phase 2 (demo: dashboard + the money-shot scenario, a stock marked BROKEN before its price actually drops + freshness monitor) → Phase 3 (backtest narrative, eval numbers, guardrail proof). If time runs short, cut from the back: Phase 3 first, then Phase 2 — never cut Phase 1.

## Commands

None exist yet — no package manager, build tool, linter, test runner, or CI is configured. Add real commands here once the FastAPI backend and frontend are scaffolded.
