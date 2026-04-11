## 5. Python — Manual FSM Class

For full control without a library:

```python
from dataclasses import dataclass, field
from typing import Callable, Dict, Optional, Any
from enum import Enum

class State(str, Enum):
    IDLE       = 'idle'
    LOADING    = 'loading'
    SUCCESS    = 'success'
    FAILURE    = 'failure'

@dataclass
class Context:
    data: Any = None
    error: str = None
    retry_count: int = 0
    request_id: str = None

@dataclass
class Transition:
    target: State
    guard: Optional[Callable] = None
    action: Optional[Callable] = None

class FSM:
    table: Dict[State, Dict[str, list[Transition]]] = {
        State.IDLE: {
            'SUBMIT': [Transition(target=State.LOADING)],
        },
        State.LOADING: {
            'SUCCESS': [Transition(target=State.SUCCESS, action=lambda ctx, e: setattr(ctx, 'data', e.get('data')))],
            'ERROR':   [Transition(target=State.FAILURE, action=lambda ctx, e: setattr(ctx, 'error', e.get('message')))],
        },
        State.FAILURE: {
            'RETRY': [
                Transition(
                    target=State.LOADING,
                    guard=lambda ctx, e: ctx.retry_count < 3,
                    action=lambda ctx, e: setattr(ctx, 'retry_count', ctx.retry_count + 1),
                ),
            ],
        },
    }

    def __init__(self):
        self.state = State.IDLE
        self.context = Context()

    def send(self, event_type: str, payload: dict = {}):
        candidates = self.table.get(self.state, {}).get(event_type, [])
        for t in candidates:
            if t.guard is None or t.guard(self.context, payload):
                if t.action:
                    t.action(self.context, payload)
                self.state = t.target
                return
        # No matching transition — event ignored
```

---
