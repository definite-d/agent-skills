## 11. Real-time Sync — WebSocket Pattern

```typescript
// Backend — WebSocket handler
wss.on('connection', (ws, req) => {
  const machineId = parseMachineId(req.url);

  ws.on('message', async (raw) => {
    const event = JSON.parse(raw.toString());
    await withMachineLock(machineId, async () => {
      const actor = await hydrateMachine(machineId);
      actor.send(event);
      const snap = actor.getSnapshot();
      await saveMachineState(machineId, snap.value, snap.context);
      ws.send(JSON.stringify({ state: snap.value, context: snap.context }));
    });
  });

  // Heartbeat — detect zombie connections
  const heartbeat = setInterval(() => ws.ping(), 30_000);
  ws.on('close', () => clearInterval(heartbeat));
  ws.on('pong', () => { /* connection alive */ });
});
```

---
