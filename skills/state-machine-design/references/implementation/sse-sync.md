## 10. Real-time Sync — SSE Pattern

**Backend = Source of Truth. Frontend = Passive View.**

The backend machine processes events and streams state snapshots to the frontend.

```typescript
// Backend — Express SSE endpoint
app.get('/api/machine/:id/stream', async (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  const sendSnapshot = (state: string, context: object) => {
    res.write(`data: ${JSON.stringify({ state, context })}\n\n`);
  };

  // Send current state immediately
  const current = await loadMachineFromRedis(req.params.id);
  if (current) sendSnapshot(current.state, current.context);

  // Subscribe to Redis Pub/Sub for future changes
  const sub = redis.duplicate();
  await sub.subscribe(`machine:events:${req.params.id}`, (message) => {
    res.write(`data: ${message}\n\n`);
  });

  req.on('close', () => sub.quit());
});

// Frontend — connect and render
const es = new EventSource(`/api/machine/${machineId}/stream`);
es.onmessage = (e) => {
  const { state, context } = JSON.parse(e.data);
  setUIState(state, context);  // pure render — no logic
};

// Frontend sends events to the backend (NOT to a local machine)
async function sendEvent(type: string, payload: object) {
  await fetch(`/api/machine/${machineId}/event`, {
    method: 'POST',
    body: JSON.stringify({ type, payload, requestId: crypto.randomUUID() }),
  });
  // Don't update local state — wait for SSE to push the new snapshot
}
```

---
