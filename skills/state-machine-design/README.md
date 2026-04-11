# State Machine Design

Design robust software systems using the state machine (statechart) pattern. This skill provides a systematic approach to modeling application logic, UI flows, async operations, protocols, workflows, and business processes as state machines.

## Why State Machines?

State machines excel when systems have:

- **3+ distinct behavioral modes** (states)
- **2+ events** that trigger transitions between modes
- **Complex conditional logic** that's hard to follow in ad-hoc if/else chains
- **Async operations** with loading, success, and error paths
- **Multi-step workflows** like wizards, checkouts, or onboarding flows

### Benefits

| Aspect          | Before (Imperative)                        | After (State Machine)                     |
| --------------- | ------------------------------------------ | ----------------------------------------- |
| Predictability  | Hidden state scattered across variables    | Explicit states with defined transitions  |
| Testability     | Path explosion through nested conditionals | Each state tested in isolation            |
| Maintainability | Logic changes ripple unpredictably         | Changes localized to specific transitions |
| Documentation   | Out-of-date comments                       | Self-documenting statecharts              |

## The Design Process

The skill guides you through a 4-phase systematic approach:

### Phase 1: Requirements Capture

Understand the domain, actors, events, invariants, and error paths before drawing any states. Key questions:

- What real-world entity are we modeling?
- Who triggers events (user, timer, server)?
- What must never happen simultaneously?
- What are the success and terminal conditions?

### Phase 2: State Enumeration

Apply the "states-first" technique:

1. List all distinct behavioral conditions
2. Challenge each: "Does the system respond differently to events here?"
3. Identify initial and terminal states
4. Document invariants and impossible combinations

### Phase 3: Transition Mapping

Define the δ function: `(source) + event [guard?] / action? → (target)`

Build a transition table and enforce:

- Every `(state, event)` pair has an explicit outcome
- Guards are mutually exclusive when multiple transitions share the same trigger
- Self-transitions for actions that don't change mode

### Phase 4: Context Design & Persistence

Determine what data the machine carries and where it lives:

- Guards and actions reference context fields
- Minimize context — display-only data may belong outside the machine
- Declare a persistence tier (memory-only, localStorage, database, etc.)
