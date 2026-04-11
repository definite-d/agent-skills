## 1. XState v5

XState v5 is the reference implementation for statecharts in JavaScript/TypeScript.
It implements the full SCXML model: guards, actions, context, hierarchy, parallel states,
actor spawning, and `invoke`.

### 1a. Full machine template

```typescript
import { createMachine, assign, sendTo, fromPromise } from 'xstate';

// --- Types ---
interface Context {
  data: SomeData | null;
  error: Error | null;
  retryCount: number;
  requestId: string | null;  // idempotency key
}

type Event =
  | { type: 'SUBMIT'; requestId: string }
  | { type: 'CANCEL' }
  | { type: 'RETRY' };

// --- Machine ---
export const machine = createMachine(
  {
    id: 'myMachine',
    types: {} as { context: Context; events: Event },
    initial: 'idle',
    context: { data: null, error: null, retryCount: 0, requestId: null },

    states: {
      idle: {
        on: {
          SUBMIT: {
            // Idempotency: only proceed if requestId is new
            guard: ({ event, context }) => event.requestId !== context.requestId,
            target: 'loading',
            actions: assign({ requestId: ({ event }) => event.requestId }),
          },
        },
      },

      loading: {
        // Timeout: auto-exit after 15 seconds
        after: {
          15000: { target: 'failure', actions: assign({ error: () => new Error('Request timed out') }) },
        },
        invoke: {
          src: 'doWork',
          input: ({ context }) => ({ requestId: context.requestId }),
          onDone:  { target: 'success', actions: assign({ data: ({ event }) => event.output }) },
          onError: { target: 'failure', actions: assign({ error: ({ event }) => event.error }) },
        },
        on: { CANCEL: 'idle' },  // invoke is automatically aborted
      },

      failure: {
        on: {
          RETRY: [
            {
              guard: ({ context }) => context.retryCount < 3,
              target: 'loading',
              actions: assign({ retryCount: ({ context }) => context.retryCount + 1 }),
            },
            { target: 'gaveUp' },
          ],
        },
      },

      success: { type: 'final' },
      gaveUp:  { type: 'final' },
    },
  },
  {
    actors: {
      // Wrap any async function as an actor
      doWork: fromPromise(async ({ input }) => {
        const response = await fetch('/api/work', {
          method: 'POST',
          body: JSON.stringify(input),
        });
        if (!response.ok) throw new Error(`HTTP ${response.status}`);
        return response.json();
      }),
    },
  }
);
```

### 1b. React integration

```typescript
import { useMachine } from '@xstate/react';

function MyComponent() {
  const [snapshot, send] = useMachine(machine);

  return (
    <div>
      <p>State: {snapshot.value}</p>
      {snapshot.matches('failure') && <p>Error: {snapshot.context.error?.message}</p>}
      <button
        onClick={() => send({ type: 'SUBMIT', requestId: crypto.randomUUID() })}
        disabled={!snapshot.matches('idle')}
      >
        Submit
      </button>
    </div>
  );
}
```

### 1c. Actor spawning pattern

```typescript
import { createMachine, assign, spawn, sendTo } from 'xstate';

const parentMachine = createMachine({
  context: { children: {} as Record<string, ActorRef<any>> },
  states: {
    active: {
      on: {
        ADD_CHILD: {
          actions: assign({
            children: ({ context, event, spawn }) => ({
              ...context.children,
              [event.id]: spawn(childMachine, { id: event.id, input: event.data }),
            }),
          }),
        },
        REMOVE_CHILD: {
          actions: [
            // Stop the actor first to prevent leaks
            ({ context, event }) => context.children[event.id]?.stop(),
            assign({
              children: ({ context, event }) => {
                const next = { ...context.children };
                delete next[event.id];
                return next;
              },
            }),
          ],
        },
        // Bubble events from children
        CHILD_DONE: { actions: 'handleChildDone' },
      },
    },
  },
});
```

### 1d. Durable state — hydration from DB

```typescript
// On server startup or page load, rehydrate from persisted state
const persistedSnapshot = await db.getMachineState(entityId);

const actor = createActor(machine, {
  // XState v5: pass snapshot to restore
  snapshot: persistedSnapshot ?? undefined,
});

// Persist on every transition
actor.subscribe(snapshot => {
  if (snapshot.status === 'active') {
    db.saveMachineState(entityId, actor.getPersistedSnapshot());
  }
});

actor.start();
```

---
