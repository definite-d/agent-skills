# Amnesia

A skill for making long autonomous work survive context-window wipes and heavy compactions by externalizing agent state to disk as a `PROMPT.md` / `PLAN.md` / `HISTORY.md` triad. The filesystem becomes external memory: a fresh, cold agent can resume exactly where a previous one left off.

## When to Use

Use this skill at the **start** of any task that is:

- Multi-step (3+ steps) and autonomous ("I'll leave you to it", "work autonomously")
- Likely to hit long compiles/builds, web fetches, or deep research (context heavy)
- Explicitly flagged for resets ("your context will be reset", "maximize your context window")

Skip it for quick Q&A or single-command fixes — single-shot work doesn't need state files.

## Core Principle

> If it matters and is in your head only, it will be forgotten. Write it down before it matters.

Rule of thumb: **write state before starting a risky/long step and after every meaningful step.** When in doubt, write.

## The State Triad

| File | Purpose | Written when |
|------|---------|--------------|
| `PROMPT.md` | Self-contained context brief — lets a fresh, cold agent resume without prior conversation | init + whenever key facts change |
| `PLAN.md` | Checkboxed task tracker with claimed states | every step completion |
| `HISTORY.md` | Append-only decision log: decisions, rationale, pitfalls, facts gathered | at every decision/finding |

Optionally add a `context/` subdir for research dumps or probe outputs if `PROMPT.md` would bloat.

## Resume Protocol

1. Read `PROMPT.md` first — re-establish goal, constraints, next actions.
2. Read `PLAN.md` — figure out exactly what is done and what is next. **Never redo a completed step.**
3. Read `HISTORY.md` — re-learn decisions and pitfalls so you don't repeat them.
4. Verify workspace state against PLAN (files exist? outputs present?) before acting.
5. Resume from the first un-ticked item, claiming it in PLAN.md.

## Quick Start

```bash
# Initialize the triad at the task root using the bundled templates
cp assets/PROMPT.md.tmpl PROMPT.md
cp assets/PLAN.md.tmpl PLAN.md
cp assets/HISTORY.md.tmpl HISTORY.md

# Fill in PROMPT.md so a cold agent can execute: goal, constraints,
# exact commands and paths, key identifiers, current state, next actions
```

## Working Disciplines

- **Preflight:** before a long build / fetch / risky edit, ensure PROMPT.md and PLAN.md reflect reality.
- **Postflight:** after each meaningful step, update PLAN.md (tick) and HISTORY.md (findings/decisions).
- **Low-budget drill:** when context is tight, stop producing narrative and write the state files — that is the only output that survives.
- **Concurrent agents:** always read a file before writing it; claim tasks in PLAN.md to avoid clashing edits.
- **Terminal output:** if a tool result is truncated or a command times out, record the partial state in HISTORY.md.

## Anti-Patterns (never do these)

- Keeping plan/decisions only in working memory "because it's short."
- Leaving PLAN.md stale while continuing to work.
- Overwriting PROMPT.md without reading the current one first.
- Writing secrets, tokens, or credentials into state files.
- One giant 10k-line PROMPT.md — split research into `context/`, keep PROMPT.md scannable.
- Re-running completed expensive steps after a reset because PLAN.md wasn't consulted.

## Contents

| File | Description |
|------|-------------|
| `SKILL.md` | Trigger conditions, procedures, resume protocol, working disciplines |
| `assets/PROMPT.md.tmpl` | Cold-resume brief template |
| `assets/PLAN.md.tmpl` | Task tracker template |
| `assets/HISTORY.md.tmpl` | Decision log template |
