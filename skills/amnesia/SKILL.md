---
name: amnesia
description: |-
  Make long autonomous work survive context-window wipes and heavy compactions by externalizing
  agent state to disk (PROMPT.md / PLAN.md / HISTORY.md). Use proactively when starting a
  multi-step task likely to span sessions, resets, or budget exhaustion: long builds, research
  harnesses, multi-commit features, overnight/background work, or when the user says "work
  autonomously" / "I'll leave you to it".

  Examples:
  - user: "Build a test harness and run it autonomously, I'm leaving" → init state files, record plan, update after every step
  - user: "You'll get context resets, manage your own state" → create PROMPT.md/PLAN.md/HISTORY.md before starting
  - user: "Continue the fulgur-test work" → resume protocol: read PROMPT.md → PLAN.md → HISTORY.md before acting
  - user: "Work in this directory across multiple sessions" → set up the state triad as the single source of truth
---
# Amnesia — context-wipe-resistant autonomous work

## The problem

Context windows get reset or compacted mid-task. Everything held only in working memory is lost:
decisions, partial plans, learned facts, the "what do I do next" state. `amnesia` fixes this by
treating the **filesystem as external memory** — state lives in three files on disk, and the agent
rehydrates from them on every resume.

## Core principle

> If it matters and is in your head only, it will be forgotten. Write it down before it matters.

Rule of thumb: **write state before starting a risky/long step and after every meaningful step.**
When in doubt, write.

## When to set up

Initialize the triad at the **start** of any task that is:
- multi-step (3+ steps) and autonomous ("I'll leave you to it", "work autonomously")
- likely to hit long compiles/builds, web fetches, or deep research (context heavy)
- explicitly flagged for resets ("your context will be reset", "maximize your context window")
- single-shot per-message work does NOT need this — skip for quick Q&A or one-command fixes

## The state triad

Place the three files at the task root (the directory the agent owns). Use the templates in
`assets/` for the initial structure.

| File | Purpose | Written when |
|------|---------|--------------|
| `PROMPT.md` | Self-contained context brief — lets a **fresh, cold** agent resume without prior conversation | init + whenever key facts change |
| `PLAN.md` | Checkboxed task tracker with claimed states | every step completion |
| `HISTORY.md` | Append-only decision log: decisions, rationale, pitfalls, facts gathered | at every decision/finding |

Optionally add a `context/` subdir for research dumps or probe outputs if PROMPT.md would bloat.

### PROMPT.md rules
- Write it so a fresh context can **execute cold**: goal, constraints, exact commands, exact paths,
  key identifiers (function names, file:line, crate versions), current state, next actions.
- Include a short "Current state / next actions" section that is updated continuously.
- No conversational filler. Dense, terse, machine-parseable.
- Preserve exact file paths, commands, version pins, and identifiers verbatim — fuzzy memory of a
  command line or flag costs a full re-discovery cycle.

### PLAN.md rules
- Checkbox list: `[ ]` pending · `[x]` done · `[~]` in progress · `[!]` blocked (note why).
- Claim tasks explicitly when working concurrently: `- [~] 2.3 renderer.rs — OWNED, started 2026-07-31`.
- Tick items **immediately** after finishing, never in batches at the end.
- Open questions / risks section at the bottom — move resolved items into HISTORY.md.

### HISTORY.md rules
- Every decision gets: what, why, alternatives rejected, consequence. Timestamp it.
- Record pitfalls and environment facts discovered empirically (e.g. "cargo init without workdir
  pollutes the repo root — always set workdir").
- Append; never rewrite history. Prefix each session with a `## <date> — Session start` header.

## Resume protocol (on any suspicion of a wipe/compaction)

1. Read `PROMPT.md` first — re-establish goal, constraints, next actions.
2. Read `PLAN.md` — figure out exactly what is done and what is next. **Never redo a completed step.**
3. Read `HISTORY.md` — re-learn decisions and pitfalls so you don't repeat them.
4. Verify workspace state against PLAN (files exist? outputs present?) before acting — drift happens.
5. Resume from the first un-ticked item, claiming it in PLAN.md.

## Working disciplines

- **Preflight:** before a long build / fetch / risky edit, ensure PROMPT.md "next actions" and PLAN.md
  status reflect reality — if the step kills the context mid-way, a fresh agent must see where you were.
- **Postflight:** after each meaningful step, update PLAN.md (tick) and HISTORY.md (findings/decisions).
- **Low-budget drill:** when context is tight, STOP producing narrative and write the state files —
  that is the only output that survives.
- **Concurrent agents:** always read a file before writing it (state may have drifted); claim tasks in
  PLAN.md to avoid clashing edits.
- **Terminal output:** if a tool result is truncated or a command times out, record the partial state
  (e.g. "deps compiled, target/ = 1.5G, 902 artifacts") in HISTORY.md — a resume agent needs to know.

## Anti-patterns (never do these)

- Keeping plan/decisions only in working memory "because it's short."
- Leaving PLAN.md stale while continuing to work.
- Overwriting PROMPT.md without reading the current one first.
- Writing secrets, tokens, or credentials into state files.
- One giant 10k-line PROMPT.md — segment: split research into `context/`, keep PROMPT.md scannable.
- Re-running completed expensive steps after a reset because PLAN.md wasn't consulted.

## Files

- `assets/PROMPT.md.tmpl` — cold-resume brief template
- `assets/PLAN.md.tmpl` — task tracker template
- `assets/HISTORY.md.tmpl` — decision log template
