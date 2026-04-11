## 2. Redux Reducer as Mealy Machine

A Redux reducer is a pure Mealy machine: `(state, action) → state`. It lacks guards,
hierarchy, and side effects — but is sufficient for many UI state needs.

```typescript
type MachineState = 'idle' | 'loading' | 'success' | 'failure';

interface SliceState {
  machine: MachineState;
  data: Data | null;
  error: string | null;
  retryCount: number;
}

// Explicit transition table — no if/else chains
const transitions: Record<MachineState, Partial<Record<string, MachineState>>> = {
  idle:    { SUBMIT: 'loading' },
  loading: { SUCCESS: 'success', ERROR: 'failure', CANCEL: 'idle' },
  success: { RESET: 'idle' },
  failure: { RETRY: 'loading', RESET: 'idle' },
};

function reducer(state: SliceState = initialState, action: Action): SliceState {
  const nextMachineState = transitions[state.machine]?.[action.type];

  // Unknown transition — ignore the action
  if (!nextMachineState) return state;

  // Apply state-specific context updates
  switch (`${state.machine}→${nextMachineState}`) {
    case 'loading→success':
      return { ...state, machine: nextMachineState, data: action.payload, error: null };
    case 'loading→failure':
      return { ...state, machine: nextMachineState, error: action.payload.message };
    case 'failure→loading':
      return { ...state, machine: nextMachineState, retryCount: state.retryCount + 1 };
    default:
      return { ...state, machine: nextMachineState };
  }
}
```

> **Limitation**: Redux doesn't model side effects (timeouts, invocations). Use
> `redux-saga`, `redux-observable`, or middleware to layer those on.

---
