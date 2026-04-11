## 3. Plain TypeScript — No Library

When a dependency isn't justified, a hand-rolled FSM is ~40 lines:

```typescript
type Transition<S extends string, E extends string, C> = {
  target: S;
  guard?: (ctx: C, event: { type: E; payload?: any }) => boolean;
  action?: (ctx: C, event: { type: E; payload?: any }) => Partial<C>;
};

class FSM<S extends string, E extends string, C> {
  private state: S;
  private context: C;
  private table: Record<S, Partial<Record<E, Transition<S, E, C>[]>>>;

  constructor(
    initial: S,
    context: C,
    table: Record<S, Partial<Record<E, Transition<S, E, C>[]>>>
  ) {
    this.state = initial;
    this.context = context;
    this.table = table;
  }

  send(event: { type: E; payload?: any }): void {
    const candidates = this.table[this.state]?.[event.type] ?? [];
    const transition = candidates.find(t => !t.guard || t.guard(this.context, event));
    if (!transition) return; // forbidden / unhandled
    if (transition.action) {
      this.context = { ...this.context, ...transition.action(this.context, event) };
    }
    this.state = transition.target;
  }

  getState() { return this.state; }
  getContext() { return this.context; }
}
```

---
