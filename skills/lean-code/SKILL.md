---
name: lean-code
description: >-
  Use when writing, editing, reviewing, or simplifying code.
---

# Lean Code

Coding agents (including you) are trained to be thorough, careful, and to show their work. Those instincts make for good conversations but bad codebases: they produce code that's padded with speculative flexibility, defensive checks against impossible failures, comments that talk to the user instead of documenting the code, and tests that exist to look complete rather than to catch real bugs. None of that padding is free; every extra branch, guard, and file is something a human now has to read, maintain, and trust.

This skill has two modes:
- **Self-check**: run silently before you consider any code-writing or code-editing task finished.
- **Audit**: run explicitly when asked to review, clean up, simplify, or find "AI slop" in existing code (yours or someone else's).

In both modes, the underlying test for every line is the same: **would this line survive being questioned by a terse senior engineer who has to maintain it?** If you can't articulate a concrete reason it's needed *right now*, for *this* requirement, it's a candidate for deletion. It's always cheaper to add code later when a real need shows up than to carry speculative code today.

## The four core patterns

### 1. Over-engineering / premature abstraction

**Why it happens:** it feels responsible to anticipate future needs, and abstraction feels like "good engineering."

**Signals:**
- An interface, abstract base class, or plugin/strategy pattern with exactly one real implementation
- Config options, feature flags, or parameters that nothing in the codebase actually varies
- A generic/configurable solution built for what is actually a one-off script or single call site
- Extensibility hooks, registries, or "for future use" arguments nobody asked for
- A factory or builder for two concrete cases

**Fix:** solve the problem that's actually in front of you, as directly as possible. Add a seam only when a *second* concrete need for it actually appears, not when you can imagine one. Match the code's complexity to the problem's real complexity, not its imagined future complexity.

### 2. Defensive-programming overload

**Why it happens:** catching errors feels safer than not catching them, so agents wrap everything defensively rather than trusting boundaries they control.

**Signals:**
- `try/except` around code that cannot realistically raise, or that only re-raises/logs and continues
- A bare or broad `except Exception` that swallows the error and returns `None`/a default, hiding real bugs instead of surfacing them
- Re-validating types, null-ness, or ranges on values the function's own caller already guarantees (internal boundaries you control)
- Guard clauses for states that the code above already makes impossible
- Retry loops, timeouts, or fallback paths for failure modes that were never in scope

**Fix:** validate at the true boundary (user input, network responses, file contents, third-party APIs) and trust the code you control past that boundary. Let bugs in your own code fail loudly with a real stack trace instead of being caught and quietly papered over; a silent `except: return None` is often worse than no handling at all, because it turns a bug into a mystery for someone else to debug later.

### 3. Chat-oriented / spec-narrating comments

**Why it happens:** the agent is in a conversation and the comment-writing instinct bleeds the conversation into the code.

**Signals:**
- Comments that reference the request or session: `// as requested`, `// per your instructions`, `// updated per feedback`
- Comments describing a diff instead of the current code: `// changed from X to Y`, `// removed the old validation here`
- Comments that just restate what the next line obviously does (`// increment counter` above `count += 1`)
- Commit-message-style narration embedded in the body of the code

**Fix:** write every comment as if the reader has no access to the conversation that produced the code, because in a week, they won't. A comment should explain something about the code *in its own terms*: a non-obvious "why," a tricky invariant, a reason a simpler approach was rejected. If a comment would only make sense to someone who read the chat, delete it or rewrite it to stand alone. If the comment just repeats what the code already says, delete it.

### 4. Unnecessary or tautological tests

**Why it happens:** generating tests feels like diligence and a visible signal of thoroughness, so agents produce a full suite whether or not it was asked for or adds coverage.

**Signals:**
- Tests for trivial getters/setters or pass-through code with no logic
- Tests that just re-implement the function's logic to assert it equals itself (tautological; they'll pass even if the logic is wrong, as long as it's consistently wrong)
- Tests that exercise a library's or framework's behavior rather than your code
- A full test scaffold, fixtures, or CI config generated for a task that never asked for tests
- Tests written to hit a coverage number rather than to protect specific, plausible failure modes

**Fix:** write a test when it protects behavior that could plausibly break, documents a real edge case, or was explicitly requested, not by default. If tests weren't requested and the code isn't especially risky, it's fine to ship without them; offer to add tests rather than assuming. When you do write a test, make sure it could actually fail if the logic were wrong; if you can't describe a bug it would catch, it's not pulling its weight.

## Other gotchas worth the same scrutiny

These show up less often but are worth a glance in the same pass:

- **Needless indirection**: wrapper functions/classes/modules that just forward to something else with no added logic
- **Dead code kept "just in case"**: unused parameters, branches, or exports with no current caller
- **Backward-compatibility shims** for an interface that never actually shipped or was never used elsewhere
- **Restating docstrings** that just repeat the function signature in prose, adding no information
- **Unsolicited scaffolding**: README files, config templates, logging setup, or CLI flags nobody asked for
- **Vague naming that hides intent**: `data`, `result`, `handleThing`, `doStuff` where a specific name would tell the reader what's actually happening
- **Reinventing the standard library**: hand-rolled versions of things the language or an already-used dependency provides

## How to run the self-check

Before telling the user a coding task is done, scan your own diff (not the whole file, just what you touched) against the six categories above. This should take seconds, not a separate elaborate pass:

1. For every new abstraction, ask: *is there a second real caller/case today, or am I guessing at the future?* If guessing, inline it.
2. For every `try/except` and every validation check, ask: *what specific failure am I defending against, and can it actually happen at this boundary?* If you can't name one, remove the guard.
3. For every comment, ask: *would this make sense to someone with zero knowledge of our conversation?* If not, rewrite or delete it.
4. For every test, ask: *what real bug would this catch?* If none, and tests weren't requested, drop it.
5. Skim for the "other gotchas" list above.

Do this quietly; don't narrate the checklist to the user or explain that you're "checking for AI slop." Just deliver leaner code.

## How to run an audit

When asked to review, clean up, simplify, or de-slop existing code:

1. Read the relevant files in full before judging anything; a pattern that looks unnecessary in isolation might be load-bearing.
2. Walk the code against the four core patterns plus the "other gotchas" list. For each finding, note: file/location, which category it falls under, and a one-line reason it's a problem (not just "this is over-engineered").
3. Present findings as a concise list, grouped by category, before making changes; this is a judgment call in places, and the user may know context you don't (e.g., that extensibility point is used by another team). Don't silently rewrite a large codebase in one pass.
4. Once the user confirms scope, apply the fixes. Prefer deletion and simplification over rewriting for style.
5. If the codebase turns out to be lean already, say so; don't manufacture findings to justify the audit.

## The underlying principle

**The best code is no code.** Every line you write is a line someone else now has to read, maintain, and trust; the fastest way to make code better is usually to delete some of it, not add to it. Before writing anything, ask whether the task can be done with less: fewer files, fewer abstractions, fewer branches, fewer words.

None of this means "never handle errors" or "never write tests" or "never comment code"; it means each of those should be a deliberate response to a real, current need, not a reflexive default. The bar isn't "could this theoretically be useful"; almost anything could theoretically be useful. The bar is "does *this* task need it, *now*." When in doubt, write the smaller, more direct version, and let the next real requirement, not speculation about it, justify anything more.
