# State Machine Design

A skill for designing robust software systems using the state machine (statechart) pattern. Helps you model application logic, UI flows, async operations, protocols, workflows, and business processes as explicit, testable state machines.

## When to Use This Skill

Use this skill when you're designing systems that have:
- **3+ distinct behavioral modes** (idle, loading, success, error, etc.)
- **Complex state transitions** triggered by events (user actions, API responses, timers)
- **Async operations** with loading states, retry logic, and error handling
- **Multi-step workflows** like wizards, checkouts, or onboarding flows
- **Conditional logic** that's becoming hard to follow in nested if/else chains

### Common Use Cases

- Authentication flows (logged out → authenticating → logged in)
- Form submissions (idle → validating → submitting → success/error)
- Media players (idle → buffering → playing → paused)
- Shopping carts (browsing → checkout → payment → confirmation)
- Connection management (disconnected → connecting → connected → reconnecting)

## Why State Machines?

State machines make implicit state explicit. Instead of scattered boolean flags (`isLoading`, `hasError`, `isAuthenticated`), you define clear states with defined transitions.

### Benefits

| Traditional Approach | State Machine Approach |
|---------------------|----------------------|
| Hidden state in variables | Explicit states everyone can see |
| Bug-prone conditional chains | Provably complete transition tables |
| Hard to test all paths | Each state tested independently |
| Documentation drifts from code | Self-documenting statecharts |
| Race conditions and edge cases | All transitions explicitly defined |

## What You'll Get

This skill guides you through a systematic 4-phase design process:

1. **Requirements Capture** — Identify actors, events, invariants, and error scenarios
2. **State Enumeration** — List all distinct behavioral conditions your system can be in
3. **Transition Mapping** — Define exactly how events move the system between states
4. **Context & Persistence** — Determine what data the machine tracks and where it lives

The skill includes implementation guides for XState v5, Redux, Python, and backend patterns with persistence, real-time sync, and distributed state coordination.

## Quick Example

Here's a form submission state machine:

```typescript
// States: idle, loading, success, failure
// Events: SUBMIT, SUCCESS, ERROR, RETRY

idle → SUBMIT [valid] → loading → SUCCESS → success
idle → SUBMIT [invalid] → idle (show errors)
loading → ERROR → failure → RETRY [retries < 3] → loading
loading → TIMEOUT → failure
```

The skill helps you design this with proper timeout handling, idempotency guards, and retry logic.

## Structure

- `SKILL.md` — Complete design methodology and implementation references
- `references/theory.md` — State machine theory and formal definitions
- `references/implementation/` — Language-specific guides (XState, Redux, Python, etc.)
