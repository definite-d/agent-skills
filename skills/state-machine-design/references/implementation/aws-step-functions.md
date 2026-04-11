## 9. Distributed / Long-running — AWS Step Functions

AWS Step Functions Express or Standard Workflows implement the States Language (Amazon's
FSM specification). Best for AWS-native architectures.

```json
{
  "Comment": "Order processing state machine",
  "StartAt": "ProcessPayment",
  "States": {
    "ProcessPayment": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123:function:ProcessPayment",
      "TimeoutSeconds": 30,
      "Retry": [{ "ErrorEquals": ["States.TaskFailed"], "MaxAttempts": 3, "IntervalSeconds": 2 }],
      "Catch": [{ "ErrorEquals": ["States.ALL"], "Next": "PaymentFailed" }],
      "Next": "ShipOrder"
    },
    "ShipOrder": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:us-east-1:123:function:ShipOrder",
      "Next": "OrderComplete"
    },
    "PaymentFailed": { "Type": "Fail", "Error": "PaymentFailed", "Cause": "Payment processing failed" },
    "OrderComplete":  { "Type": "Succeed" }
  }
}
```

---
