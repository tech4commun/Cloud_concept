# Deep Dive: Planning & Designing Your Amazon EBS Architecture — Notes

## Course Objectives
- Plan/design an EBS architecture matching performance + data availability needs
- Identify app/org requirements before EBS deployment
- Match workload types to EBS
- Pick the right EBS volume type for a performance profile
- Identify repeatable, scalable deployment strategies

---

## 1. AWS Block Storage Services Review

### Instance Store (recap)
- Ephemeral, physically attached to host — **direct attached disk** architecture.
- NVMe-backed → sub-millisecond latency; HDD-backed → throughput-based performance.
- Only certain EC2 instance types support it; size/count varies by type.
- **Best for:** buffers, caches, scratch data, fleet-replicated data (e.g. load-balanced web servers).
- **Not recommended** for most workloads — not replicated/spread for durability.

**Data loss triggers:** disk failure, instance stop, hibernate, terminate.
**Data survives:** reboot only.

### Amazon EBS (recap)
- High-performance block storage; behaves as raw, unformatted block device.
- Persists **independently** of instance lifecycle.
- **Dynamically reconfigurable** (size/performance) — unlike fixed physical disks.
- Single volume: up to **64 TiB**; multiple volumes → **petabyte** scale.
- Replicated **within an Availability Zone** for durability.
- Snapshots + lifecycle policies → backup to Amazon S3, geographic protection.
- Pay only for what you provision.

### 9 EBS Feature Areas
1. Persistent
2. Built-in encryption
3. Multiple volume type options
4. High availability & durability
5. Elastic Volumes
6. Multi-Attach
7. Volume monitoring
8. Snapshots
9. Backups

---

## 2. EBS Volume Types — Comparison

**5 types:** General Purpose SSD, Provisioned IOPS SSD, Throughput Optimized HDD, Cold HDD, Magnetic (legacy — not recommended for new volumes).

**Constraints:**
- Account has a **limit** on volume count/total storage (increasable on request).
- Volume + instance **must be in the same AZ**.
- **Multi-Attach**: available on select volume types + instance types only.

### General Purpose SSD: gp3 vs gp2
| | gp3 | gp2 |
|---|---|---|
| Durability | 99.8–99.9% (0.1–0.2% AFR) | same |
| Volume size | 1 GiB–16 TiB | 1 GiB–16 TiB |
| IOPS range | 3,000–16,000 | 100–16,000 |
| Provisioned IOPS? | ✅ Yes | ❌ No (must grow volume size) |
| Max throughput | 1,000 MB/s | 250 MB/s* |
| Provisioned throughput? | ✅ Yes | ❌ No |
| Boot volume | ✅ | ✅ |
| Multi-Attach | ❌ | ❌ |

\* gp2 throughput: 128 MiB/s (≤170 GiB) → up to 250 MiB/s (170–334 GiB, burst-dependent) → 250 MiB/s guaranteed (≥334 GiB).

> **Key takeaway:** gp3 decouples performance from size — cheaper, more flexible than gp2.

### Provisioned IOPS SSD: io1 vs io2 vs io2 Block Express
| | io1 | io2 | io2 Block Express |
|---|---|---|---|
| Durability | 99.8–99.9% | 99.999% (0.001% AFR) | 99.999% |
| Volume size | 4 GiB–16 TiB | 4 GiB–16 TiB | 4 GiB–64 TiB |
| IOPS range | 100–64,000 | 100–64,000* | 100–256,000** |
| Max throughput | 1,000 MB/s | 1,000 MB/s* | 4,000 MB/s** |
| Boot volume | ✅ | ✅ | ✅ |
| Multi-Attach | ✅ | ✅ | ✅ |

\* Full IOPS/throughput only guaranteed on Nitro instances provisioned >32,000 IOPS; otherwise capped at 32,000 IOPS / 500 MB/s.
\** Requires an instance type supporting io2 Block Express — otherwise falls back to io2 limits.

### HDD Volumes: Throughput Optimized vs Cold
| | Throughput Optimized HDD | Cold HDD |
|---|---|---|
| Volume size | 125 GiB–16 TiB | 125 GiB–16 TiB |
| Throughput range | 5–500 MB/s | 2–192 MB/s |
| Max burst throughput | 500 MB/s | 250 MB/s |
| Boot volume | ❌ | ❌ |
| Multi-Attach | ❌ | ❌ |

---

## 3. Common Workloads for EBS

- **Lift and shift / rehosting**: fastest path to cloud — minimal changes to app/data, just recreate volumes and attach to compute.
- Use **Elastic Volumes** to tune size/performance as you migrate.

### Common Use Cases
- Enterprise applications
- Relational databases
- NoSQL databases
- Big data analytics engines
- File systems & media workflows

### Business Continuity
- Regularly back up data/logs across Regions → minimize data loss + recovery time.
- Two backup options:
  - **AWS Backup** — fully managed, integrates with EC2/EBS, broad backup toolset.
  - **EBS Snapshots** — built-in, low-cost, **incremental** (only changed blocks).
    - Stored in S3 → **11 nines (99.999999999%)** durability.
    - Enables fast volume restore + cross-AZ copy within a Region.

---

## 4. Identifying Application Requirements

**Sizing** = matching resource type/characteristics (capacity, media type, performance) to real requirements.

### Consequences of Incorrect Sizing
| | Risk |
|---|---|
| **Over-sizing** | Wasted spend — over-provisioned capacity/volumes, unneeded high-performance volume type, excess IOPS/throughput |
| **Under-sizing** | Performance/usability problems, potential customer impact |
| **Optimal** | Balanced cost vs. performance |

### Questions to Ask About Your App/Workflow
- What data types exist, and how much capacity does each need?
- How should data be split across volumes (min/ideal volume count)?
- What block size / cache size configuration is needed?
- Access pattern: random vs. sequential?
- Performance needs: latency tolerance, IOPS, throughput — per data type?
- Concurrency: number of users/operations, and impact on performance?
- Persistence/durability needs — any data that's OK to lose?
- Does data need to be shared? With what, and how?

### Available Planning Resources
- Published reference architectures
- Vendor-provided application guidelines
- Examining actual workload traffic
- AWS Solutions Architect advice
- AWS Professional Services

---

## 5. Building a Test Environment

**AWS advantage over on-prem:** spin up test resources in minutes, no procurement wait; pay only for what's used; tear down when done.

- **Best practice:** use a **separate AWS account** for testing → cleaner cost tracking, avoids accidental prod impact.

### Block I/O Simulation Tools
- Flexible I/O (**fio**)
- Jetstress
- Iometer
- sysbench
- Oracle Orion

---

## 6. Budgetary Constraints & Cost Optimization

**Common pattern:** scale down at launch to fit budget → scale up once proven.

**Cost model:** consumption-based — pay only for what's used.

### Ways to Lower/Manage EBS Costs
- Start with **smaller volumes**, expand later via **Elastic Volumes** (note: shrinking requires a manual process — growing is easy, shrinking is not).
- Default to **gp3** when unsure of IOPS/throughput needs — fits ~**80%** of workloads.
- Start with **lower provisioned IOPS** on io2, tune up/down as needed.
- Prefer **current-generation volume types** — e.g., gp3 allows provisioning IOPS/throughput independently of size, and costs less per GB/month than gp2.

### AWS Pricing Calculator — Key Facts
- Estimates cost per **volume type**, one type at a time (can model multiples of same type).
- Based on **hours/month**; can reduce hours to model partial months.
- Uses **730 hours** as a standard billable month (not 720 = actual 30-day month).
- Snapshot pricing uses your **provisioned volume size** as the initial full snapshot size.
- Volume size input is **not validated** against type min/max — check EBS volume type limits separately.
- **"Show calculations"** section (bottom of each panel) reveals the math behind the estimate.
- Found under: EBS Pricing page → *Additional pricing resources*, or accessed directly.

---

## 7. Repeatable & Scalable Deployment Strategies

### AWS Well-Architected Framework — 5 Pillars
1. Operational excellence
2. Security
3. Reliability
4. Performance efficiency
5. Cost optimization

> Repeatable/scalable deployment strategy = **Operational Excellence** pillar in action.

### Operational Excellence — 5 Design Principles
1. Perform operations as code
2. Make frequent, small, reversible changes
3. Refine operations procedures frequently
4. Anticipate failure
5. Learn from all operational failures

### Infrastructure as Code (IaC) — Benefits
- Cost reduction (less manual ops staffing)
- Speed (faster repeat deployments)
- Reduced risk (removes human error)
- Testability (test IaC templates in prod-like environments early)
- Stable & scalable environments (consistent, no config drift)
- Accountability (version-controlled like source code)
- Configuration consistency
- Documentation (the code *is* the doc)
- Enhanced security (consistent security posture every deploy)

---

## 8. AWS Infrastructure Automation Tools

| Tool | What it is | Underlying engine |
|---|---|---|
| **AWS CloudFormation** | Models/provisions related AWS + 3rd-party resources as a "stack" from a template | — (foundational service) |
| **AWS Elastic Beanstalk** | PaaS layer — upload app, it handles provisioning/scaling/monitoring | CloudFormation |
| **AWS CDK** | Define infra in code (TypeScript, JS, Python, Java, C#/.NET) → constructs → stacks/apps | CloudFormation |

### CloudFormation
- Template = desired resources + dependencies → deployed together as a **stack**.
- Create/update/delete the whole stack as **one unit**.
- Works across multiple accounts and Regions.
- Requires the caller to have the **IAM permissions** for whatever actions the template performs (e.g., need EC2 create permission to provision instances via CFN).

### Elastic Beanstalk
- Supports: Go, Java, .NET, Node.js, PHP, Python, Ruby.
- You upload an app source bundle (e.g., `.war`) → EB provisions & configures resources (e.g., EC2) automatically.
- Manage via console, AWS CLI, or the dedicated `eb` CLI.

### AWS CDK
- Define infra using real programming languages + OOP patterns.
- Supports logic (if/for), reusable/shareable constructs, modular projects, standard testing & code review workflows.
- Compiles down to CloudFormation for actual provisioning.

---

## Quick Reference

- **Instance store:** ephemeral, host-attached, no durability guarantees — cache/buffer/scratch only.
- **EBS:** persistent, resizable, AZ-bound, up to 64 TiB/volume.
- **gp3** fits ~80% of workloads — default choice when unsure.
- **io2 Block Express**: highest performance tier (256,000 IOPS / 4,000 MB/s) — needs supporting instance type.
- **EBS Snapshots**: incremental, stored in S3, 11 nines durability.
- **Pricing calculator**: uses 730 hrs/month; one volume type per estimate.
- **IaC tools**: CloudFormation (core) → Elastic Beanstalk (PaaS simplicity) / CDK (code-first flexibility).