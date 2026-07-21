# Design Patterns for AWS Step Functions

> Course notes — Modules 1–3 (Introduction → Common Design Patterns → Best Practices)

---

## Module 1: Introduction

### What Step Functions workflows can do
- Invoke **AWS Lambda** functions
- Run business logic in **containers**
- Update databases (e.g., **DynamoDB**)
- Publish messages to a **queue** after a step / workflow completes

### Why design patterns matter (online retailer example)
Order processing needs to:
1. Verify inventory & reserve item
2. Verify payment
3. Verify delivery info
4. Notify fulfillment to ship
5. Notify/update customer at multiple stages

**Problems without patterns:**
- All steps run sequentially even when independent → slow
- No handling for multiple items in one order
- No rollback if a step fails (e.g., inventory actually out of stock)
- Monolithic workflow → higher cost, hard to maintain

**Five core patterns covered:** Parallelism · Dynamic Parallelism · Nested · Orchestration & Choreography · Saga

---

## Module 2: Common Design Patterns

### 1. Parallelism — `Parallel` state

Run a **fixed** number of branches concurrently on the **same input**.

**Use when:** steps don't depend on each other (e.g., look up address + phone at same time).

**Key fields:**
| Field | Required | Purpose |
|---|---|---|
| `Branches` | ✅ | Array of sub-state-machines (each needs `StartAt`, `States`) |
| `ResultPath` | ❌ | Controls how input + results combine into output |
| `ResultSelector` | ❌ | Manipulate results *before* `ResultPath` |
| `Catch` | ❌ | Fallback state on defined error |
| `Retry` | ❌ | Auto-retry after caught error |

Waits until **all** branches terminate before continuing.

```json
{
  "LookupCustomerInfo": {
    "Type": "Parallel",
    "End": true,
    "Branches": [
      { "StartAt": "LookupAddress",
        "States": { "LookupAddress": { "Type": "Task", "Resource": "...:AddressFinder", "End": true } } },
      { "StartAt": "LookupPhone",
        "States": { "LookupPhone": { "Type": "Task", "Resource": "...:PhoneFinder", "End": true } } }
    ]
  }
}
```

**Order-processing use:** verify payment + reserve inventory in parallel. Parallel states can be nested.

---

### 2. Dynamic Parallelism — `Map` state

Run the **same** steps in parallel across **many different inputs** (array).

**Two messaging shapes:**
- **Fan-out** — one array of messages → each item to its own Lambda (e.g., SQS batch)
- **Scatter-gather** — broadcast, then aggregate results (e.g., transcode 10×500MB → join into 5GB)

**Key fields:**
| Field | Required | Purpose |
|---|---|---|
| `Iterator` | ✅ | Sub-workflow run per array item |
| `ItemsPath` | ❌ | JSON path to the input array |
| `MaxConcurrency` | ❌ | Cap on parallel iterations (`0` = unlimited, `1` = serial) |
| `ResultPath` / `ResultSelector` / `Catch` / `Retry` | ❌ | Same as Parallel |

**Order-processing use:** per-item availability check → prepare for delivery → start delivery, all inside a `Map`.

```json
"ProcessAllItems": {
  "Type": "Map",
  "InputPath": "$.detail",
  "ItemsPath": "$.items",
  "MaxConcurrency": 3,
  "Iterator": {
    "StartAt": "CheckAvailability",
    "States": {
      "CheckAvailability": { "Type": "Task", "Resource": "...", "Next": "PrepareForDelivery",
        "Retry": [ { "ErrorEquals": ["TimeOut"], "IntervalSeconds": 1, "BackoffRate": 2, "MaxAttempts": 3 } ] },
      "PrepareForDelivery": { "Type": "Task", "Resource": "...", "Next": "StartDelivery" },
      "StartDelivery": { "Type": "Task", "Resource": "...", "End": true }
    }
  },
  "ResultPath": "$.detail.processedItems",
  "Next": "SendOrderSummary"
}
```

**Parallel vs Map:** Parallel = fixed branches / same input. Map = same branch / many inputs.

---

### 3. Nested workflows

Launch another state machine from a `Task` via `states:startExecution`.

**Benefits:**
- Separate high-level vs task-specific workflows
- Reuse via a library of modular workflows
- Reduce complexity / easier troubleshooting
- Cost optimization by mixing Standard + Express

**Standard vs Express Workflows:**
| | Standard | Express |
|---|---|---|
| Duration | Up to 1 year | Up to 5 min |
| History | Full history in console | CloudWatch Logs |
| Rate | >2,000/sec | >100,000/sec |
| Pricing | Per state transition | Executions × duration × memory |
| Best for | Long-running, auditable (payments, EMR) | High-volume events (IoT, streaming, mobile backends) |

**Starting a nested execution:**
```json
{
  "Type": "Task",
  "Resource": "arn:aws:states:::states:startExecution",
  "Parameters": {
    "StateMachineArn": "arn:aws:states:us-east-1:123456789012:stateMachine:HelloWorld",
    "Input": {
      "Comment": "Hello world!",
      "AWS_STEP_FUNCTIONS_STARTED_BY_EXECUTION_ID.$": "$$.Execution.Id"
    }
  },
  "End": true
}
```

- Use `.$` + `$$.Execution.Id` to link child execution to parent
- Special key `AWS_STEP_FUNCTIONS_STARTED_BY_EXECUTION_ID` adds parent/child links in the Step Functions console

---

### 4. Orchestration & Choreography

- **Orchestration** — Step Functions as central coordinator inside **one bounded context**
- **Choreography** — services react to events on an **EventBridge** bus; no central controller

**Better together:** orchestrate within a domain, publish results to EventBridge, let subscribers in other domains react.

**Bounded contexts** (DDD) for the retailer example:
- **Retail** — orders, inventory, payments
- **Fulfillment** — shipping/processing
- **Customer support** — returns, refunds

**Integration in ASL:** resource `arn:aws:states:::events:putEvents`

**Two integration patterns:**
| Pattern | Behavior | Workflow types |
|---|---|---|
| **Request-Response** | Continues immediately after HTTP response | Standard + Express |
| **Wait-for-callback** | Passes `TaskToken`, waits until returned | Standard only |

```yaml
# Request-Response
Send an EventBridge custom event:
  Type: Task
  Resource: 'arn:aws:states:::events:putEvents'
  Parameters:
    Entries:
      - Detail: { Message: 'Hello from Step Functions!' }
        DetailType: MyDetailType
        EventBusName: MyEventBusName
        Source: MySource
  Next: NEXT_STATE
```

```yaml
# Wait-for-callback
  Resource: 'arn:aws:states:::events:putEvents.waitForTaskToken'
  Parameters:
    Entries:
      - Detail:
          Message: 'Hello from Step Functions!'
          TaskToken.$: $$.Task.Token
```

**Order example:** one workflow emits event → EventBridge rules route to *direct-to-customer shipping* OR *in-store pickup* sub-workflow depending on order type. Consumers no longer need `DescribeExecution`; they just subscribe.

---

### 5. Saga — failure management for distributed transactions

Microservice transactions can't use ACID → risk of partial commits. The **saga pattern** runs **compensating transactions** to undo prior successful steps when a later step fails.

**Example:** travel booking — flight seat + hotel room reserved during payment; if payment fails, release both.

**Two ways to implement in Step Functions:**

#### a) `Choice` state — route on result
```json
"ChoiceStateX": {
  "Type": "Choice",
  "Choices": [
    { "Not": { "Variable": "$.type", "StringEquals": "Private" }, "Next": "Public" },
    { "Variable": "$.value", "NumericEquals": 0, "Next": "ValueIsZero" },
    { "And": [
        { "Variable": "$.value", "NumericGreaterThanEquals": 20 },
        { "Variable": "$.value", "NumericLessThan": 30 }
      ], "Next": "ValueInTwenties" }
  ],
  "Default": "DefaultState"
}
```

#### b) `Catch` field — trap errors, route to compensation
```json
"X": {
  "Type": "Task",
  "Resource": "arn:aws:states:us-east-1:123456789012:task:X",
  "Next": "Y",
  "Retry": [
    { "ErrorEquals": ["ErrorA","ErrorB"], "IntervalSeconds": 1, "BackoffRate": 2.0, "MaxAttempts": 2 },
    { "ErrorEquals": ["ErrorC"], "IntervalSeconds": 5 }
  ],
  "Catch": [ { "ErrorEquals": ["States.ALL"], "Next": "Z" } ]
}
```

**Order-processing:** if *Inventory updated* fails → run *Revert inventory* (release DynamoDB stock) → *Remove order* → mark workflow Failed. Preserves data integrity.

---

## Module 2 (cont.): Best Practices

### Enable AWS X-Ray tracing
- Visualizes workflow performance, isolates root cause of errors/latency
- Especially useful for **Saga** and **Nested** patterns (hardest to trace)
- Toggle on in "Specify state machine settings" when creating or editing a state machine

---

## Quick cheat-sheet

| Pattern | State/Service | Use when |
|---|---|---|
| Parallelism | `Parallel` | Fixed independent branches, same input |
| Dynamic Parallelism | `Map` | Same steps across an input array (fan-out / scatter-gather) |
| Nested | `states:startExecution` | Reuse, isolation, mix Standard + Express for cost |
| Orchestration + Choreography | `events:putEvents` + EventBridge | Cross-domain, decoupled consumers |
| Saga | `Choice` or `Catch` + compensating tasks | Distributed transactions needing rollback |
