# Orchestra

Coordinate parallel subagents to decompose, diagnose, fix, and report on a task. A domain-agnostic playbook for conducting specialized workers without letting them collide.

## When to Use

- The task decomposes into multiple independent or semi-independent units (files, modules, sources, sections, records).
- User asks you to orchestrate, coordinate subagents/delegates, or run something "in parallel."
- Workflows with distinct review/fix/report phases, even if nobody says "orchestrator."
- Proactive use when a task you were about to do serially is clearly decomposable.

Don't bother when the task is one linear thread of reasoning; orchestration overhead just makes it slower.

## The Core Loop

1. **Decompose** the task into the smallest set of focused, non-overlapping scopes, one per subagent.
2. **Diagnose in parallel.** Read-only subagents assess their scope and report findings. Nothing gets changed yet.
3. **Fix in parallel, only where needed.** Dispatch a fix batch scoped to flagged issues. Skip scopes with no findings.
4. **Verify** fixes resolved the issues and added no regressions. Loop back if unclean, but cap the loops.
5. **Report** a single consolidated summary against the bar the user set (merge, publish, ship).

## Coordination Rules

- Partition before you dispatch; no two subagents share a scope or resource.
- Serialize shared-state mutations. Parallel analysis is fine, but commits, writes, and rate-limited calls happen one at a time under your control.
- Track who owns what in a state log so resumption after compaction or interruption doesn't redo work.

Refer to `SKILL.md` for the full details, including the tool-use fallback protocol, state-resilience rules, efficiency caps, and a worked PR review example.

## Structure

- `SKILL.md` - Complete orchestration methodology with worked example