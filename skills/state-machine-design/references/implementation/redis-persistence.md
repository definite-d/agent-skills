## 7. Backend Persistence — Redis/Valkey

Use Redis hashes for fast state reads in high-throughput systems. Combine with Pub/Sub
for real-time state change notifications.

```typescript
const MACHINE_KEY = (id: string) => `machine:${id}`;

async function saveMachineToRedis(id: string, state: string, context: object, ttlSeconds = 3600) {
  const key = MACHINE_KEY(id);
  await redis.hset(key, {
    state,
    context: JSON.stringify(context),
    updatedAt: Date.now().toString(),
  });
  await redis.expire(key, ttlSeconds);

  // Publish state change for SSE / WebSocket subscribers
  await redis.publish(`machine:events:${id}`, JSON.stringify({ state, context }));
}

async function loadMachineFromRedis(id: string) {
  const raw = await redis.hgetall(MACHINE_KEY(id));
  if (!raw?.state) return null;
  return { state: raw.state, context: JSON.parse(raw.context) };
}

// Distributed lock to prevent concurrent event processing on the same machine
async function withMachineLock<T>(id: string, fn: () => Promise<T>): Promise<T> {
  const lockKey = `lock:machine:${id}`;
  const token = crypto.randomUUID();
  const acquired = await redis.set(lockKey, token, { NX: true, EX: 5 });
  if (!acquired) throw new Error('Machine is being processed by another worker');
  try {
    return await fn();
  } finally {
    // Only release if we still own the lock
    const current = await redis.get(lockKey);
    if (current === token) await redis.del(lockKey);
  }
}
```

---
