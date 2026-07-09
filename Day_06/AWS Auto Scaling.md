# AWS Auto Scaling — Getting Started Notes

## What It Is

**AWS Auto Scaling** is a fully managed service that monitors applications and **automatically adjusts capacity** — scaling up under demand, down when quiet — to maintain steady performance at the **lowest possible cost**.

### AWS Auto Scaling vs. EC2 Auto Scaling
| | Scope |
|---|---|
| **EC2 Auto Scaling** | Manages **one resource type** — EC2 instances in a single Auto Scaling group |
| **AWS Auto Scaling** | **Unified, cross-service** coordination layer — EC2 ASGs, ECS services, DynamoDB tables/GSIs, Aurora read replicas, all from one place |

- A **scaling plan** = a collection of scaling instructions for related resources, applied together as one coordinated strategy.
- AWS Auto Scaling sits **above** EC2 Auto Scaling and Application Auto Scaling — it orchestrates them, doesn't replace them.
- Why it matters: if your EC2 fleet scales up, dependent resources (e.g. DynamoDB) may need to scale too — a scaling plan keeps them in sync.

### Two Scaling Methods
| Method | How it works |
|---|---|
| **Dynamic scaling** | Reacts in real time via **target tracking policies** |
| **Predictive scaling** | Uses **machine learning** to forecast traffic and **pre-scales** ahead of demand |

> Best practice: **combine both** — predictive for known/recurring patterns (e.g. daily 9 AM peak), dynamic for unexpected surges.

---

## Core Functionality

A **scaling plan** combines:
1. **Resource discovery** — finds which resources to scale (e.g. via tags or CloudFormation stack)
2. **Scaling strategies** — defines the goal (cost vs. availability balance)
3. **Scaling policies** — the actual rules triggering scale events

> Key idea: resources scale as a **coordinated group**, not in isolation.

### Resource Discovery Methods
- **Tag-based discovery** — find resources sharing a common tag (works regardless of how they were created — Console, CLI, third-party tools).
- **CloudFormation stack-based discovery** — find resources belonging to a specific stack.

### Scaling Strategies
- **Optimize for cost** — run resources at higher utilization, less spare capacity (good for non-critical, delay-tolerant workloads).
- **Balance availability and cost** — middle ground.
- **Optimize for availability** — keep more spare capacity for responsiveness.
- **Custom** — define your own target metric (e.g. 20% CPU target).

---

## Technical Concepts (IT Domains Behind Auto Scaling)

- Cloud computing fundamentals (on-demand resources)
- Capacity planning strategies
- Cost optimization practices
- High availability design
- Performance management
- Infrastructure as code
- Monitoring & observability (feeds the scaling decisions)

---

## Key Features & Capabilities

1. **Unified scaling interface** — one place to manage scaling across resource types
2. **Predictive scaling** — ML-based forecasting
3. **Dynamic scaling** — real-time target tracking
4. **Scaling plans** — coordinated blueprint across resources
5. **Multi-resource scaling** — EC2, ECS, DynamoDB, Aurora together
6. **Cost optimization recommendations**

---

## Practical Business Applications

| Scenario | How Auto Scaling Helps |
|---|---|
| **E-commerce traffic spikes** (flash sales, holidays) | Adds capacity before/during event, removes after — pay only for peak usage, not year-round worst-case |
| **SaaS variable daily load** | Predictive scaling pre-provisions before morning login surge |
| **Batch processing** | Spin up for the job, scale back to baseline after — avoids idle server cost |
| **Dev/test environments** | Scale down to zero/minimal overnight & weekends — big cost savings |
| **Live media streaming events** | Reacts to rapid concurrent-viewer spikes using metrics like concurrent connections/CPU |
| **Microservices with uneven demand** | Each service scales independently on its own metrics (e.g. payment service spikes at checkout, search stays flat) — prevents over-provisioning the whole stack |

---

## Architecture

**Scaling plan = the orchestrator.** It coordinates:
- Amazon EC2 Auto Scaling groups
- Amazon ECS services
- Amazon DynamoDB tables
- Amazon Aurora read replicas
- Driven by **Amazon CloudWatch metrics**
- Applies **dynamic + predictive scaling** together

### Example: Creating a Scaling Plan (AWS CLI)
```bash
aws autoscaling-plans create-scaling-plan \
  --scaling-plan-name my-web-app-plan \
  --application-source '{"TagFilters": [{"Key": "app", "Values": ["my-web-app"]}]}' \
  --scaling-instructions '[
    {
      "ServiceNamespace": "autoscaling",
      "ResourceId": "autoScalingGroup/my-asg",
      "ScalableDimension": "autoscaling:autoScalingGroup:DesiredCapacity",
      "MinCapacity": 2,
      "MaxCapacity": 10,
      "TargetTrackingConfigurations": [{
        "PredefinedScalingMetricSpecification": {"PredefinedScalingMetricType": "ASGAverageCPUUtilization"},
        "TargetValue": 70.0
      }],
      "PredictiveScalingMaxCapacityBehavior": "SetForecastCapacityToMaxCapacity"
    },
    {
      "ServiceNamespace": "dynamodb",
      "ResourceId": "table/my-orders-table",
      "ScalableDimension": "dynamodb:table:ReadCapacityUnits",
      "MinCapacity": 5,
      "MaxCapacity": 100,
      "TargetTrackingConfigurations": [{
        "PredefinedScalingMetricSpecification": {"PredefinedScalingMetricType": "DynamoDBReadCapacityUtilization"},
        "TargetValue": 70.0
      }]
    }
  ]'
```
> One scaling plan → coordinates an EC2 ASG **and** a DynamoDB table together, both target-tracking at 70%.

---

## Integrations

- **Amazon EC2 Auto Scaling**
- **Amazon ECS**
- **Amazon DynamoDB** (read/write capacity scaling)
- **Amazon Aurora** (read replica scaling)
- **Amazon CloudWatch** — metrics that trigger *every* scaling action, across *all* integrations
- **Elastic Load Balancing** — traffic distribution to scaled resources
- **AWS CloudFormation** — templates for resource discovery/deployment

> CloudWatch is the common dependency — no CloudWatch visibility, no scaling trigger.

---

## Integration Considerations (Production Readiness)

1. **Security & IAM permissions**
   - The Auto Scaling **service-linked role** needs correct policies to launch/terminate instances on your behalf.
   - Use **read-only IAM policies** for users who should view scaling activity but not modify plans.

2. **Scalability & capacity limits**
   - Check **EC2 service quotas** for the target instance type/Region — a scaling policy can trigger correctly but still fail if quota is hit.

3. **Monitoring & scaling visibility**
   - Confirm CloudWatch metrics are flowing and scaling actions actually complete (not just triggered).

### Common Troubleshooting Patterns
| Symptom | Likely Cause |
|---|---|
| Scaling policy triggers but instances don't launch | IAM service-linked role missing permissions, or EC2 quota reached |
| New instances up but no performance improvement (behind ALB) | Instances haven't passed **health checks** yet — LB isn't routing to them |
| DynamoDB not scaling while EC2 does | DynamoDB table **not included** in the same scaling plan |
| ASG launching/terminating every few minutes ("thrashing") | Scaling thresholds set **too close together** |

---

## Quick Reference

- **AWS Auto Scaling** = multi-resource orchestrator; **EC2 Auto Scaling** = single-resource-type scaler.
- **Scaling plan** = discovery + strategy + policies, applied as one coordinated unit.
- **Predictive** = ML forecast, pre-scales ahead of known patterns.
- **Dynamic** = real-time reaction via target tracking.
- Best practice for predictable + unpredictable demand: **use both together**.
- **CloudWatch** underpins every scaling decision across every integration.
- Core exam/quiz theme: coordinate related resources in **one scaling plan** rather than configuring each in isolation — and remember, **scaling down** is where the cost savings actually come from.