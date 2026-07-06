# Amazon EBS — Getting Started Notes

## 1. Introduction to Amazon EBS

**What it is:** Block-level storage volumes for use with EC2 instances — attach high-performance virtual drives to cloud servers.

- EBS volume = appears to EC2 as a **raw block device** (like a new physical hard drive).
- You format it with a file system, then use it like any block device.
- Volumes exist **independently** of the EC2 instance — data can persist through stop/terminate (config-dependent).
- Analogy: like pulling a hard drive out and reading it on another computer.

### Problems EBS Solves
| Problem | How EBS Helps |
|---|---|
| **Data persistence** | Non-root volumes survive instance termination by default |
| **Storage management** | Increase capacity with no application downtime |
| **Performance consistency** | Predictable performance; multiple volume types for different workloads |
| **Data protection** | Snapshot-based safeguards against data loss |

### Benefits
- High availability & data durability
- On-demand scalable capacity
- Performance optimized per workload
- Persistence independent of EC2 instance
- Snapshots for backup/recovery
- Encryption for security

### Pricing
- **Pay-as-you-go** — pay only for provisioned capacity, no upfront commitment/minimum fees.
- Volume type pricing tiers:
  - **General Purpose SSD (gp)** — balanced price/performance
  - **Provisioned IOPS SSD (io)** — higher performance, higher cost
  - **Throughput Optimized HDD / Cold HDD** — economical, specific use cases
- **Snapshots**: billed per GiB-month stored; **incremental** — only changed blocks are charged after the first snapshot.
- Extra charges may apply for **cross-Region data transfer**.

> 🧠 Knowledge check takeaway: EBS = *"block-level storage volumes that persist independently of EC2 instances."*

---

## 2. Architecture and Use Cases

### AWS Services That Integrate with EBS
- **CloudWatch** — monitoring
- **CloudFormation** — infrastructure as code
- **AWS Backup** — automated backup management
- **AWS KMS** — encryption key management
- **IAM** — access control

> Integration turns EBS from "basic storage" into part of automated, secure, highly available architectures.

### Core Technical Concepts
- **IOPS** (Input/Output Operations Per Second)
- **Throughput**
- **Snapshots**
- **Encryption**
- **EBS-optimized instances** (dedicated bandwidth for EBS traffic)
- **Elastic Volumes** (resize/change type without downtime)

### Typical Use Cases
| Use Case | Why EBS Fits |
|---|---|
| **Enterprise database hosting** | io1/io2 Block Express → low-latency, high-IOPS for Oracle, SQL Server, MySQL, PostgreSQL |
| **High Performance Computing (HPC)** | Scalable, high-throughput block storage |
| **Boot volumes for EC2** | Root volume for instance OS |
| **Containerized application storage** | Persistent volumes for containers |

> Databases: scale storage independently of compute; use snapshots for point-in-time disaster recovery.

### Additional Considerations (Not Handled by EBS Itself)
- Performance monitoring & optimization
- Backup & disaster recovery strategy
- Time-based snapshot copy
- Volume initialization rate (provisioned rate)
- Operations management & cost optimization
- Security & access management
- Data protection & recovery

> These are **governance practices** you must build around EBS — not automatic features.

> 🧠 Knowledge check takeaway: EBS = *"a block-level storage service that provides virtual hard drives for EC2 instances."* Directly attaches to **EC2 instances**.

---

## 3. How to Use Amazon EBS (Demonstrations)

| # | Demo | Purpose |
|---|---|---|
| 1 | **Launch an EC2 instance** with 2 EBS volumes | Core skill — attaching storage at launch |
| 2 | **Create an EBS snapshot** | Point-in-time backup for DR |
| 3 | **Encrypt an EBS volume** | Protect sensitive data at rest |
| 4 | **Modify EBS volume size** | Increase capacity live, no downtime |
| 5 | **Create a volume from a snapshot** | Clone/restore data |
| 6 | **Clean up resources** | Avoid ongoing charges after the lab |

### ⚠️ Volume Modification Constraints
- After modifying a volume, **wait ≥ 6 hours** and confirm state is `in-use` or `available` before modifying again.
- Modification duration: minutes to hours (not instant).
- A **1 TiB** volume modification typically takes **up to 6 hours** — but can take **24+ hours** depending on system/network load.
- Time does **not scale linearly** — larger volumes can finish faster than smaller ones in some cases.
- After resizing, you must **manually extend the file system** on the OS side — resizing the volume alone isn't enough.

---

## Quick Reference

**EBS in one line:** Persistent, resizable, block-level virtual hard drives for EC2 — billed per GiB provisioned, with snapshot-based backup and encryption built in.

**Key exam-style facts:**
- Non-root volumes persist after termination **by default**.
- Snapshots are **incremental** (only changed blocks billed after the first).
- Volume type tiers: **gp (SSD) / io (Provisioned IOPS SSD) / Throughput Optimized HDD / Cold HDD**.
- Must wait **6 hours** between volume modifications.
- File system extension is a **separate manual step** after volume resize.