# AWS Block Storage — Notes

## 1. Block Storage Basics

- **Block storage** = raw storage presented as a disk/volume, formatted and attached to a compute system.
- Data is split into fixed-size **blocks**.
- Runs on HDD, SSD, or NVMe devices — can also run on SAN (Storage Area Network) systems.
- The **OS or application** manages/formats the volume once attached.

### AWS Block Storage Options
| Service | Type | Notes |
|---|---|---|
| **Amazon EBS** | Persistent | Primary block storage for EC2 |
| **EC2 Instance Store** | Ephemeral | Temporary, tied to instance lifecycle |
| **Amazon FSx for Windows File Server** | Block volumes over iSCSI | Less common |

---

## 2. EC2 Instance Store

**Definition:** Temporary (ephemeral) block storage physically attached to the host hardware running the EC2 instance.

### Key Facts
- Directly attached disk — behaves like a local drive, not network storage.
- **Not all instance types support it** — depends on instance type.
- Number/size/type of instance store volumes **varies by instance type**.
- **No snapshots or backups possible.**
- **Included in instance price** — no extra charge.

### Data Persistence Rules
| Event | Data survives? |
|---|---|
| Reboot | ✅ Yes |
| Stop | ❌ Lost |
| Hibernate | ❌ Lost |
| Terminate | ❌ Lost |
| Underlying disk failure | ❌ Lost |

> ⚠️ On restart, the instance may land on a **different host** — instance store is gone either way.

### Architecture
- Located on the **same physical host** as the EC2 instance → **sub-millisecond latency**.
- Storage type (SSD/HDD/NVMe) is fixed by instance type.
- Some instance types support **multiple** instance store volumes.

### Use Cases
- Buffers, caches, scratch/temp data
- Data replicated across a fleet (e.g., load-balanced web servers)

> AWS recommends **persistent storage** (EBS / EFS / FSx) for anything that can't afford to be lost.

---

## 3. Amazon EBS (Elastic Block Store)

**Definition:** High-performance, persistent block storage service for EC2.

### Core Traits
- Persists **independently of instance lifecycle** (survives stop/terminate — unlike instance store).
- Raw, unformatted block device → mount, format, and use like any disk.
- **Dynamically resizable/configurable** — unlike fixed-size physical disks.
- Multiple **volume types** for price/performance tradeoffs.
- Pay-only-for-what-you-provision pricing.

### Ideal For
- Databases (random reads/writes)
- Throughput-heavy sequential workloads (e.g., Hadoop)
- File systems
- Fine-grained, block-level access needs

### Performance Range
- Single-digit ms latency (e.g., SAP HANA)
- GB/s throughput (e.g., big data workloads)

---

## 4. EBS Architecture

- Access via: Internet, VPN, AWS PrivateLink, or AWS Direct Connect.
- Manage via: AWS Console, CLI, or API.

### Basic Architecture
- EBS volume(s) attached to an EC2 instance.
- Multiple volumes / volume types can attach to **one** instance.
- **Same Availability Zone required** — instance & volume must be in the same AZ.
- Access controlled via **IAM** (users/groups/roles need explicit permissions).

### With EBS Snapshots
- Snapshots = **incremental backups** of a volume.
- Stored in **Amazon S3** (AWS-managed, protected vault).
- Multiple snapshots possible per volume.
- Snapshots are **optional**, additive to basic architecture.

---

## 5. EBS Use Cases

- Enterprise application storage
- Relational databases
- NoSQL databases
- Big data analytics
- File systems & media workflows
- **Lift-and-shift migrations** from on-prem (via AWS MGN / AWS SMS)

> Lift-and-shift = move app + data to cloud with minimal changes → fastest path to production → optimize later.

---

## 6. EBS Features (9 Categories)

1. Persistent
2. Built-in encryption
3. High availability & durability
4. Multiple volume type options
5. Elastic Volumes (resize/change type live)
6. Multi-attach (one volume → multiple instances)
7. Volume monitoring
8. Snapshots
9. Backups

---

## 7. EBS Pricing

- Based on: **volume type**, **provisioned size**, **provisioned IOPS/throughput**.
- Price **varies by Availability Zone**.
- Snapshots billed by **actual storage used** (not provisioned size).

---

## Quick Comparison: Instance Store vs. EBS

| Feature | Instance Store | EBS |
|---|---|---|
| Persistence | Ephemeral | Persistent |
| Survives stop/terminate | ❌ No | ✅ Yes |
| Backups/snapshots | ❌ Not supported | ✅ Supported |
| Location | Same host as instance | Network-attached, same AZ |
| Cost | Included in instance price | Billed separately |
| Best for | Cache, buffers, temp/replicated data | Databases, file systems, persistent workloads |

---

## 8. Amazon FSx for NetApp ONTAP — Block Storage

**Definition:** Fully managed file storage service built on NetApp's ONTAP file system — also offers **block storage** via iSCSI.

### Key Facts
- Access protocol: **iSCSI** (Internet Small Computer Systems Interface).
- Block storage exposed via:
  - **LUNs** (Logical Unit Numbers) — logically allocated block volumes on shared SAN.
  - **igroups** (iSCSI initiator groups) — LUNs are mapped to these to expose storage to hosts.
- Accessible from **Linux and Windows** hosts, on-prem or in AWS.
- Underlying storage: **SSD**, sub-millisecond latency.
- ONTAP OS provides **locking/coordination** → enables **shared access** to the same volume (EBS requires the OS to handle this itself).

### Beyond Block Storage
- Also supports **file storage** via NFS and SMB — including **simultaneous multi-protocol access** (NFS + SMB on same data).
- Snapshot, clone, and replicate data with one click.
- Auto-tiers data to lower-cost storage (less manual capacity planning).
- Fully managed: no patching, no failover/failback management, no manual backups.
- Built-in **cross-Region disaster recovery** and backup support.
- Integrates with IAM, WorkSpaces, KMS, CloudTrail.

### Use Cases
- Already using NetApp ONTAP on-premises → smooth migration path.
- Simplifies **lift-and-shift** (less app refactoring needed).
- Teams familiar with ONTAP tools/commands.
- Need **shared/concurrent access** to block volumes across hosts.

### Tradeoffs vs. EBS
| Aspect | FSx for ONTAP | EBS |
|---|---|---|
| Protocol | iSCSI (network) | Direct block attach |
| Performance | Slightly lower (network overhead) | Higher (direct access) |
| Shared access | Built-in via ONTAP locking | Needs OS-level coordination |
| Best fit | NetApp shops, shared volumes | General-purpose EC2 storage |

---

## 9. Additional AWS Storage Resources

### Amazon EBS
- [EBS Overview](https://aws.amazon.com/ebs/)
- [EBS Features](https://aws.amazon.com/ebs/features/)
- [EBS Pricing](https://aws.amazon.com/ebs/pricing/)
- EC2 Linux/Windows User Guides + EBS API Reference (via [AWS Docs](https://docs.aws.amazon.com/))

### Related AWS Storage Services
- Amazon S3 (object storage)
- Amazon EFS (file storage)
- Amazon FSx family: Lustre, Windows File Server, NetApp ONTAP, OpenZFS
- AWS Storage Gateway (S3 File Gateway, Volume Gateway, Tape Gateway)
- AWS Outposts, Amazon File Cache
- AWS Snow Family, AWS DataSync, AWS Transfer Family
- AWS Backup, AWS Elastic Disaster Recovery (DRS)

### Related "Getting Started" Courses (AWS Skill Builder)
1. AWS Storage Services – Portfolio Introduction
2. **AWS Block Storage Services Getting Started** *(this course)*
3. AWS Object Storage Services Getting Started
4. AWS File Storage Services Getting Started
5. AWS Hybrid Storage Services Getting Started
6. AWS Edge Storage, Data Transfer & File Transfer Services Getting Started
7. AWS Storage Data Protection Services Getting Started