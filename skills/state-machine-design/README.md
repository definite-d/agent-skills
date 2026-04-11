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
