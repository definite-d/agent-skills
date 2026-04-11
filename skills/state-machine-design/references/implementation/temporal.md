## 8. Distributed / Long-running — Temporal

Temporal is ideal for state machines with steps measured in hours, days, or weeks.
The workflow IS the state machine — Temporal handles durability, retries, and timeouts natively.

```typescript
import { proxyActivities, sleep, condition, defineSignal, setHandler } from '@temporalio/workflow';

const cancelSignal = defineSignal('CANCEL');
const approveSignal = defineSignal<[{ approvedBy: string }]>('APPROVE');

export async function orderWorkflow(orderId: string): Promise<string> {
  let cancelled = false;
  let approved = false;
  let approvedBy = '';

  setHandler(cancelSignal, () => { cancelled = true; });
  setHandler(approveSignal, ({ approvedBy: by }) => { approved = true; approvedBy = by; });

  const { processPayment, shipOrder, sendConfirmation } = proxyActivities({
    startToCloseTimeout: '30 seconds',
    retry: { maximumAttempts: 3 },
  });

  // State: awaiting_approval — wait up to 48 hours
  const resolved = await condition(() => approved || cancelled, '48 hours');
  if (!resolved || cancelled) return 'cancelled';  // timeout or cancel

  // State: processing_payment
  await processPayment(orderId);

  // State: shipping
  await shipOrder(orderId);

  // State: complete
  await sendConfirmation(orderId, approvedBy);
  return 'complete';
}
```

Key Temporal advantages over manual FSMs:
- **Automatic durability** — workflow state survives worker crashes.
- **Built-in retries** — activities retry with configurable backoff.
- **Timeouts at every level** — workflow, activity, and schedule timeouts.
- **Signals = events** — external systems send signals to advance the workflow.

---
