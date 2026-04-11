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

| Aspect | Before (Imperative) | After (State Machine) |
|--------|---------------------|----------------------|
| Predictability | Hidden state scattered across variables | Explicit states with defined transitions |
| Testability | Path explosion through nested conditionals | Each state tested in isolation |
| Maintainability | Logic changes ripple unpredictably | Changes localized to specific transitions |
| Documentation | Out-of-date comments | Self-documenting statecharts |
