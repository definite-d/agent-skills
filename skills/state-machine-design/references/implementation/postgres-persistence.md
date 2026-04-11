## 6. Backend Persistence — PostgreSQL + Node.js

Use a `JSONB` column to persist the machine snapshot. Add `version` for optimistic locking
and migration safety.

```sql
CREATE TABLE machine_states (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  entity_id   UUID NOT NULL UNIQUE,       -- the business entity this machine belongs to
  state       TEXT NOT NULL,
  context     JSONB NOT NULL DEFAULT '{}',
  version     INTEGER NOT NULL DEFAULT 1,
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Index for stall detection (cron job finds machines stuck > threshold)
CREATE INDEX machine_states_state_updated
  ON machine_states (state, updated_at)
  WHERE state IN ('loading', 'processing');
```

```typescript
// Optimistic-lock save — prevents concurrent writes from corrupting state
async function saveMachineState(
  entityId: string,
  state: string,
  context: object,
  expectedVersion: number
) {
  const result = await db.query(
    `UPDATE machine_states
     SET state = $1, context = $2, version = version + 1, updated_at = NOW()
     WHERE entity_id = $3 AND version = $4
     RETURNING version`,
    [state, JSON.stringify(context), entityId, expectedVersion]
  );
  if (result.rowCount === 0) throw new Error('Optimistic lock conflict — state was modified concurrently');
  return result.rows[0].version;
}

// Crash-recovery: find stalled machines and advance them
async function recoverStalledMachines() {
  const stalled = await db.query(
    `SELECT entity_id, state, context, version FROM machine_states
     WHERE state IN ('loading', 'processing')
       AND updated_at < NOW() - INTERVAL '5 minutes'`
  );
  for (const row of stalled.rows) {
    const actor = createActor(machine, {
      snapshot: { value: row.state, context: row.context }
    });
    actor.start();
    actor.send({ type: 'TIMEOUT' });
    await saveMachineState(row.entity_id, actor.getSnapshot().value, actor.getSnapshot().context, row.version);
  }
}
```

---
