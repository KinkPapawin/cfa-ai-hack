---
name: clarify-before-build
description: Use this skill whenever asked to build, implement, scaffold, wire up, or "just start coding" any software system, service, pipeline, or non-trivial feature — before writing or generating any code. If the request, or the project's own docs (README, BRIEF.md, requirements.md, architecture.md, SKILL.md in .claude/skills/), leave a data model, scope boundary, tech choice, edge case, or acceptance criterion unstated or ambiguous, STOP and ask the user detailed clarifying questions instead of guessing or silently picking a default. Always consult this skill before beginning implementation work, even mid-project, even if earlier parts of the build were already clarified — new files/features can introduce new unknowns. Do not use this skill for trivial one-off scripts, pure bug fixes with an obvious cause, or requests that are already fully specified.
---

# Clarify Before Build

Software built on a guessed assumption is more expensive to fix later than a
question is to ask now. This skill makes "ask, don't assume" the default
behavior before any non-trivial build work starts.

## Step 1 — Check the project's own source of truth first

Before asking the user anything, check whether the answer already exists in
the project:

- Root-level docs: `README.md`, `BRIEF.md`, `HANDOFF.md`, or similar.
- `.claude/skills/*/SKILL.md` and any `references/*.md` inside those skill
  folders — these often hold the actual locked requirements.
- Existing code, config files (`package.json`, `.env.example`, migrations,
  schema files), and comments.
- If two docs conflict, prefer whichever one explicitly says it supersedes
  the other (e.g. "if X conflicts with Y, X wins"); if that's not stated,
  that conflict itself is something to ask the user about.

Only ask the user about things that are genuinely unanswered after this
check. Don't make the user re-answer something already written down —
that's a signal you didn't read the docs.

## Step 2 — Decide if this actually needs a question

Ask before building when any of these are true and unresolved:

- **Scope / boundaries**: what's in vs. explicitly out for this piece of work.
- **Data model**: shape of core entities, required vs. optional fields,
  relationships, storage choice.
- **Tech / framework choice**: anything the docs flag as "TBD" or "not yet
  decided," or that isn't implied by the existing stack.
- **Edge cases & failure modes**: what happens on bad input, missing data,
  partial failure, concurrent writes, empty states.
- **Non-functional constraints**: expected scale, latency, auth/permissions,
  compliance or licensing rules (e.g. data that must never leave the system).
- **Acceptance criteria**: what "done" looks like for this specific task —
  if you can't state a concrete way to verify it works, that's a gap.
- **Naming/behavior ambiguity**: a term used two different ways in the
  request or docs, or a word ("real-time", "secure", "scalable") with no
  concrete definition attached.

Don't ask when:

- The task is a trivial, easily-reversible script or one-off fix.
- The gap has only one reasonable interpretation and getting it wrong costs
  nothing (e.g. variable naming, formatting, which linter to run first).
- The user already gave a specific, unambiguous instruction for exactly this
  case.

When genuinely unsure whether something rises to the level of "ask" — ask.
A clarifying question costs the user a few seconds; a wrong guess costs a
rebuild.

## Step 3 — How to ask

- Stop before writing implementation code. Don't scaffold "something
  reasonable" and ask afterward — that produces throwaway work and anchors
  the user toward whatever you happened to guess.
- Batch related questions together in one message rather than trickling
  them out one at a time across multiple turns.
- Be specific and concrete — name the exact field, endpoint, file, or
  decision point. Prefer offering 2–4 concrete options over an open-ended
  "what do you want here?" when the option space is enumerable.
- State what you already found from the docs, so the user knows you did
  the legwork: "requirements.md doesn't say whether X handles concurrent
  writes — did you want optimistic locking, or is single-writer fine for
  this build?"
- If only *part* of the task is blocked, say which part can start now and
  which part is waiting on an answer, rather than blocking everything.

## Step 4 — Proceeding after the answer

- Record the answer's implications inline (a code comment, a short note in
  the relevant doc, or a TODO) so the next person — human or Claude — hits
  the same clarity instead of re-opening the question.
- If the user says "just use your best judgment," that's explicit
  permission to proceed — pick the most conservative/reversible option,
  state the assumption in one line, and continue. This is different from
  silently guessing without asking at all.
