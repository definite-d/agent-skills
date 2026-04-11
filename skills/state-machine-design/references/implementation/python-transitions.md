## 4. Python — `transitions` library

`transitions` is the most popular Python FSM library (pip install transitions).

```python
from transitions import Machine
from transitions.extensions import HierarchicalMachine  # for HSM
import time

class OrderProcessor:
    def on_enter_processing(self):
        print("Starting processing...")
        self.started_at = time.time()

    def on_enter_failure(self):
        print(f"Failed after {time.time() - self.started_at:.1f}s")

    def is_retriable(self):
        return self.retry_count < 3

    def __init__(self):
        self.retry_count = 0
        self.started_at = 0.0

states = ['idle', 'processing', 'success', 'failure', 'gave_up']

transitions = [
    {'trigger': 'submit',   'source': 'idle',       'dest': 'processing'},
    {'trigger': 'succeed',  'source': 'processing',  'dest': 'success'},
    {'trigger': 'fail',     'source': 'processing',  'dest': 'failure'},
    {
        'trigger': 'retry',
        'source':  'failure',
        'dest':    'processing',
        'conditions': 'is_retriable',
        'before':  lambda self: setattr(self, 'retry_count', self.retry_count + 1),
    },
    {'trigger': 'retry',   'source': 'failure',     'dest': 'gave_up'},
]

processor = OrderProcessor()
machine = Machine(
    model=processor,
    states=states,
    transitions=transitions,
    initial='idle',
    auto_transitions=False,  # disables wildcard any→state transitions
)

processor.submit()    # idle → processing
processor.fail()      # processing → failure
processor.retry()     # failure → processing (retry_count=1)
print(processor.state)  # 'processing'
```

### Timeout pattern in Python

```python
import threading

class TimedMachine:
    def __init__(self):
        self._timeout_handle = None

    def on_enter_loading(self):
        self._timeout_handle = threading.Timer(15.0, self._on_timeout)
        self._timeout_handle.start()

    def on_exit_loading(self):
        if self._timeout_handle:
            self._timeout_handle.cancel()

    def _on_timeout(self):
        self.timeout()  # trigger the 'timeout' event
```

---
