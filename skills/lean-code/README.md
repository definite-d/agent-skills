# Lean Code

Keeps AI-generated code lean by catching and preventing AI slop: over-engineering, premature abstraction, defensive-programming overload, chat-oriented comments, and unnecessary tests.

## When to Use

- Writing, editing, or reviewing code.
- User asks to clean up, simplify, audit, or de-slop existing code.
- Before telling the user any coding task is done (run as a silent self-check).

## The Four Core Patterns

1. **Over-engineering / premature abstraction** - interfaces, config options, or extensibility hooks nobody actually uses.
2. **Defensive-programming overload** - try/except and validation for failures that can't happen at an internal boundary.
3. **Chat-oriented comments** - comments that reference the conversation instead of explaining the code on its own terms.
4. **Unnecessary or tautological tests** - tests for trivial code, tests that re-implement the logic they assert, or tests nobody asked for.

## The Test for Every Line

Would this line survive being questioned by a terse senior engineer who has to maintain it? If you can't name a concrete reason it's needed right now, for this requirement, it's a candidate for deletion.

## Underlying Principle

**The best code is no code.** The fastest way to make code better is usually to delete some of it, not add to it. The bar isn't "could this theoretically be useful"; it's "does this task need it, now."

## Structure

- `SKILL.md` - Full patterns with signals, fixes, self-check and audit procedures