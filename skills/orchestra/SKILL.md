---
name: orchestra
description: Coordinate parallel subagents to decompose, diagnose, fix, and report on a task.
---

# Orchestra

You are a conductor of subagents. This skill is a domain-agnostic playbook
for that role: decompose a task into focused parallel subagents, coordinate
them so they don't collide, loop diagnose, fix, and verify until the result
meets the bar, then report back. The example throughout (reviewing a PR) is
illustrative; apply the same shape to research, audits, refactors, data
cleanup, or anything else that decomposes.

## When to use this

Use this skill whenever a task can be broken into multiple independent or
semi-independent units of work and you can complete it faster or more
thoroughly with parallel subagents than by working through it serially
yourself: code review and fix-up, multi-file refactors, research synthesis
across sources, content or data audits, compliance sweeps, large migrations,
or any "check a bunch of things, fix what's broken, report back" job. Apply
it any time the user asks you to act as an orchestrator, coordinate
subagents or delegates, run something "in parallel," or describes a workflow
with distinct review/fix/report phases, even if they don't use the word
"orchestrator." Also apply it proactively when a task you're about to do
serially is clearly decomposable and would benefit from parallel specialized
subagents.

Use orchestration when the task has multiple checkable or fixable units that
are mostly independent of each other (files, sections, sources, modules,
claims, records). Don't bother decomposing a task that's inherently a single
linear thread of reasoning; that's slower with orchestration overhead, not
faster.

## The core loop

As conductor, you don't do the underlying work yourself; you decompose it,
dispatch it, and integrate the results. Run this loop:

1. **Decompose** the task into the smallest set of focused, non-overlapping
   scopes that each map to one subagent.
2. **Diagnose in parallel.** Dispatch read-only subagents to assess or review
   their scope and report findings. Don't let anything get changed yet.
3. **Fix in parallel, only where needed.** Dispatch a second batch of
   subagents (scoped to just the flagged issues) to resolve them. Skip this
   phase entirely for scopes with no findings.
4. **Verify** that fixes actually resolved the issues and didn't introduce
   new ones. Loop back to step 2/3 if not clean, but cap the number of loops
   (see Efficiency below) rather than iterating forever.
5. **Report** a single consolidated summary of what was checked, what was
   found, what was fixed, and the final state against the bar the user set
   (for example "ready to merge," "ready to publish," "ready to ship").

## Decomposition: more focused subagents beats fewer generalized ones

Split by natural unit boundaries in the task (files, modules, sources,
sections, record batches) rather than by arbitrary chunking. Give each
subagent:

- A single, clearly bounded scope (one file, one module, one claim to
  fact-check, one data shard), not "review everything."
- Enough standing context to do the job without re-deriving it: relevant
  style guides, required tools/skills, acceptance criteria, and what "done"
  looks like for their scope.
- An explicit output format so you can merge results mechanically rather
  than re-interpret them (for example, a list of issues as
  {location, severity, description}).

Don't over-fragment trivial work. If a scope is small enough that spawning
an agent costs more than doing the check inline, skip the subagent.

## Coordination: don't let your subagents trip over each other

This is the part most likely to go wrong silently, so treat it as a hard
requirement, not a nice-to-have:

- **Partition before you dispatch.** Give each subagent a scope that does
  not overlap another subagent's scope. If two scopes must touch the same
  resource, merge them into one subagent's job rather than parallelizing.
- **Serialize shared-state mutations.** Anything that mutates a shared
  resource (committing to a shared repo, writing to a shared file, appending
  to a shared doc, calling a rate-limited shared API) must happen one at a
  time under your control, even if the analysis leading up to it was
  parallel. Have fix subagents propose changes; apply them yourself, or
  through one serialized subagent at a time.
- **Don't let conflicting fixes land silently.** If two fix subagents'
  changes would interact (for example, they touch the same file or the same
  downstream consumer), resolve the order yourself or merge the fix into a
  single subagent before dispatching.
- **Track subagent identity and scope in your state log** (see below) so
  that if you're resumed or reprompted, you know exactly who owns what.

## Tool-use protocol

If the task specifies required tools or skills, make sure your subagents
use them. Don't let a subagent freelance a different approach because it's
faster. Define an explicit fallback order: try the primary tool first, and
only fall back to a secondary method (for example a CLI instead of an MCP
tool) after the primary tool demonstrably fails, and only for the step that
failed, not preemptively across the board. Log which path was actually used
so your final report and any resumed work reflect reality.

## State management and resilience

Long-running multi-agent work is vulnerable to interruptions: dropped
connections, context compaction, or the user saying "carry on" after a
mishap. Guard against redoing or losing work:

- Maintain an explicit status list (todo, in progress, done per subagent
  scope) that you check and update at each phase transition, not just held
  in implicit conversation context.
- Before you resume after any interruption, reconstruct state from this
  list (and from artifacts already produced: diagnostic findings, fixes
  already applied) rather than restarting the whole loop from scratch.
- Make each subagent's output self-contained enough (scope, findings, what
  was changed) that it's still useful if the conversation is compacted
  before you finish reading it.
- If a mishap happens mid-fix (for example, a shared-state mutation half
  applied), check actual current state before retrying; don't assume the
  last known state is still accurate.

## Efficiency

- Parallelize everything that's genuinely independent; don't serialize out
  of caution once scopes are properly partitioned.
- Don't re-run a diagnostic subagent whose scope had no findings.
- Cap verification loops. If the same issue survives two fix attempts,
  surface it to the user rather than looping indefinitely.
- Batch related lightweight checks into one subagent rather than spawning
  one subagent per trivial item.

## The report

End with a single consolidated report, not a scattering of subagent
transcripts. Include:

- What you checked (scopes/subagents dispatched)
- What you found, by severity and category
- What you fixed, and by which method, including any tool fallbacks used
- Anything left unresolved and why
- An explicit statement of whether the final state meets the bar the user
  set, so they know if it's truly ready for them to take the next step
  (merge, publish, ship, sign off) themselves.

## Worked example: PR review orchestration

To ground the abstract loop above, here's how it maps onto reviewing a pull
request:

- **Decompose**: dispatch one subagent per changed file or logical module,
  not one subagent for the whole diff.
- **Diagnose**: have each review subagent check its file(s) using the
  required review skills/checklists, reporting issues without editing
  anything.
- **Coordinate**: make sure no two subagents edit the same file
  simultaneously in the fix phase; if a fix in one file affects another
  (for example a shared interface), handle it with one subagent or sequence
  it explicitly.
- **Fix**: only dispatch a fix subagent for files with findings; have each
  use the required tools, falling back to a secondary method only on
  demonstrated tool failure.
- **Verify**: re-review changed files to confirm issues are resolved and no
  regressions were introduced, looping once more only if needed.
- **Report**: summarize what was reviewed, what was wrong, what was fixed,
  and confirm the branch is in a mergeable state, so the user can merge it
  themselves.
