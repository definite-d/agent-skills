## 12. Testing State Machines

### 12a. Unit testing transitions (framework-agnostic)

```typescript
describe('machine transitions', () => {
  it('enters loading on valid SUBMIT', () => {
    const actor = createActor(machine);
    actor.start();
    actor.send({ type: 'SUBMIT', requestId: 'req-1' });
    expect(actor.getSnapshot().value).toBe('loading');
  });

  it('ignores duplicate SUBMIT with same requestId (idempotency)', () => {
    const actor = createActor(machine);
    actor.start();
    actor.send({ type: 'SUBMIT', requestId: 'req-1' });
    expect(actor.getSnapshot().value).toBe('loading');
    actor.send({ type: 'SUBMIT', requestId: 'req-1' }); // duplicate
    expect(actor.getSnapshot().value).toBe('loading');   // still loading, not restarted
  });

  it('transitions to failure on timeout', async () => {
    vi.useFakeTimers();
    const actor = createActor(machine);
    actor.start();
    actor.send({ type: 'SUBMIT', requestId: 'req-1' });
    vi.advanceTimersByTime(15_001);
    expect(actor.getSnapshot().value).toBe('failure');
    expect(actor.getSnapshot().context.error?.message).toBe('Request timed out');
    vi.useRealTimers();
  });
});
```

### 12b. Model-based testing (enumerate all paths)

XState provides a `@xstate/graph` package to enumerate all reachable state paths
and generate tests automatically:

```typescript
import { createTestModel } from '@xstate/test';
import { getSimplePaths } from '@xstate/graph';

const model = createTestModel(machine);
const paths = model.getPaths();

paths.forEach(path => {
  it(path.description, async () => {
    await path.test({
      states: {
        idle:    async ({ page }) => expect(await page.locator('[data-state]').getAttribute('data-state')).toBe('idle'),
        loading: async ({ page }) => expect(page.locator('.spinner')).toBeVisible(),
        success: async ({ page }) => expect(page.locator('.success-message')).toBeVisible(),
        failure: async ({ page }) => expect(page.locator('.error-message')).toBeVisible(),
      },
      events: {
        SUBMIT: async ({ page }) => page.locator('button[type=submit]').click(),
        RETRY:  async ({ page }) => page.locator('button.retry').click(),
      },
    });
  });
});
```

### 12c. Testing persistence / recovery

```typescript
it('recovers stalled machine to failure state after timeout', async () => {
  // Simulate a machine that was persisted in `loading` state 10 minutes ago
  await db.query(
    `INSERT INTO machine_states (entity_id, state, context, updated_at)
     VALUES ($1, 'loading', '{}', NOW() - INTERVAL '10 minutes')`,
    [testEntityId]
  );

  await recoverStalledMachines();

  const row = await db.query('SELECT state FROM machine_states WHERE entity_id = $1', [testEntityId]);
  expect(row.rows[0].state).toBe('failure');
});
```
