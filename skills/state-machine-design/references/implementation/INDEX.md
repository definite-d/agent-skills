# Implementation Reference — Index

Each file in this directory covers one stack or concern. Load only the file(s) relevant
to the user's technology choices — do not load the entire folder.

| File | Stack / Concern |
|------|----------------|
| `xstate-v5.md` | XState v5 — full machine template, React integration, actor spawning, DB hydration |
| `redux.md` | Redux reducer as a Mealy machine; limitations; middleware for side effects |
| `plain-typescript.md` | Hand-rolled FSM class (~40 lines); no library dependency |
| `python-transitions.md` | `transitions` library (pip); timeout pattern |
| `python-manual.md` | Manual FSM class in Python; no library dependency |
| `postgres-persistence.md` | PostgreSQL JSONB persistence; optimistic locking; stall-recovery cron |
| `redis-persistence.md` | Redis/Valkey hash persistence; distributed locking; Pub/Sub for SSE |
| `temporal.md` | Temporal Workflows — durable long-running machines; signals as events |
| `aws-step-functions.md` | AWS Step Functions States Language; Lambda tasks; retries/catches |
| `sse-sync.md` | Backend = Source of Truth pattern; SSE streaming; passive frontend |
| `websocket-sync.md` | WebSocket event delivery; server-side machine; heartbeat detection |
| `testing.md` | Unit tests; model-based path enumeration; persistence/recovery integration tests |

## Selection guide

- **Frontend-only app, JS/TS**: `xstate-v5.md`
- **Frontend-only app, React + Redux**: `redux.md`
- **No external dependencies, TS**: `plain-typescript.md`
- **Python service**: `python-transitions.md` or `python-manual.md`
- **Needs to survive restarts, Node.js + SQL**: `postgres-persistence.md` + `xstate-v5.md`
- **High-throughput, needs locking**: `redis-persistence.md`
- **Steps measured in hours/days**: `temporal.md`
- **AWS-native architecture**: `aws-step-functions.md`
- **Real-time UI sync from backend machine**: `sse-sync.md` or `websocket-sync.md`
- **Writing tests**: `testing.md`
