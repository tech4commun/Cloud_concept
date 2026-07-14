# Introduction to Building with AWS Databases — Course Notes

## MODULE 1: AWS Well-Architected Framework

### Overview
The AWS Well-Architected Framework is a set of architectural best practices to ensure workloads are **secure, reliable, efficient, cost-effective, and sustainable**. It provides a consistent way to evaluate architectures against best practices — it's a **conversation, not an audit**.

**Components (3):**
1. A series of questions
2. Six pillars
3. Design principles

**General design principles (6):**
- Stop guessing your capacity needs
- Test systems at production scale
- Automate to make architectural experimentation easier
- Allow for evolutionary architecture
- Drive architecture using data
- Improve through game days

**The Six Pillars** (not ordered/ranked — each considered equally):
1. Operational Excellence
2. Reliability
3. Performance Efficiency
4. Cost Optimization
5. Sustainability
6. Security

---

### Pillar 1: Operational Excellence
**Definition:** How your organization supports business objectives, runs workloads effectively, gains insight into operations, and continuously improves processes.

**5 Design principles:**
- Perform operations as code
- Make frequent, small, reversible changes
- Refine operations procedures frequently
- Anticipate failure
- Learn from all operational failures

**Database tooling:**
| Tool | Purpose |
|---|---|
| **CloudWatch Logs** | Centralize logs from all systems/apps/services; search, filter, query, visualize |
| **CloudWatch Dashboards** | Customizable single-view monitoring pages |
| **CloudWatch Alarms** | Monitor a metric, trigger notifications/actions on threshold breach; can pair with auto-scaling |
| **CloudWatch Contributor Insights (DynamoDB)** | Identifies most-accessed and most-throttled items/keys in a table |
| **RDS/Aurora Performance Insights** | Visualize DB load by waits, SQL statements, hosts, or users |

---

### Pillar 2: Reliability
**Definition:** Workloads performing their intended function and recovering quickly from failure. Key topics: distributed system design, recovery planning, adapting to changing requirements.

**5 Design principles:**
- Automatically recover from failure
- Test recovery procedures
- Scale horizontally to increase aggregate workload availability
- Stop guessing capacity
- Manage change in automation

**Two core concepts:**
1. **Availability & Resiliency** — resiliency = ability to recover from disruptions; availability = % of time a workload is usable (measured in "nines," e.g., 99.9999% = six nines)
2. **Disaster Recovery** — planning for and recovering from failure

**Database reliability techniques:**
- **Multi-AZ deployment** — primary + synchronous standby in a different AZ within the same Region; automatic failover on primary failure
- **Read replicas** — asynchronous copies for horizontal read scaling; route read traffic to replicas to offload primary
- **Cross-Region replicas** — supported for MariaDB, Aurora MySQL, MySQL, Oracle, PostgreSQL (NOT supported for SQL Server on RDS)
- **Global tables (DynamoDB)** — multi-Region, multi-active replica tables; automatic conflict-free replication; app stays available even if a Region degrades
- **Maintenance windows** — schedule OS/engine updates during low-traffic periods; stack windows across instances to avoid overlapping downtime

**Disaster recovery planning:**
- **RPO (Recovery Point Objective)** — how much data loss is acceptable
- **RTO (Recovery Time Objective)** — how quickly you must recover
- Disaster categories: natural disasters, technical failures, human actions
- **Availability ≠ Disaster Recovery** — availability focuses on workload components; DR focuses on discrete copies of the entire workload
- Business Continuity Plan (BCP) should define RTO/RPO upfront

**DR strategies (increasing cost/complexity):**
1. **Backup and restore** — periodic/continuous backups with point-in-time recovery (PITR); available via DynamoDB backup, RDS snapshot, Aurora DB snapshot, Redshift snapshot, Neptune snapshot
2. **Pilot light**
3. **Warm standby**
4. **Multi-site active/active**

---

### Pillar 3: Performance Efficiency
**Definition:** Structured, streamlined allocation of IT/computing resources — selecting resource types/sizes optimized for workload, monitoring performance, maintaining efficiency as needs evolve.

**5 Design principles:**
- Democratize advanced technologies
- Go global in minutes
- Use serverless architecture
- Experiment more often
- Consider mechanical sympathy

**Database approach:**
1. **Understand data characteristics** — is it transactional? How does it interact with data? Pick a purpose-built engine (relational, key-value, document, in-memory, graph, time-series, ledger) instead of one-size-fits-all
2. **Evaluate specific database offerings** — compute/memory, provisioned IOPS, scalability, caching options
3. **Collect and record performance metrics** — use CloudWatch Logs to measure transactions/sec, slow queries, latency
4. **Choose data storage based on access patterns** — e.g., relational DB for transactions; key-value store for high throughput/eventual consistency
5. **Optimize storage based on access patterns** — indexing, key distribution, data warehouse design, caching

---

### Pillar 4: Cost Optimization
**Definition:** Avoiding unnecessary costs — understanding spending over time, selecting right-sized resources, scaling to meet needs without overspending.

**5 Design principles:**
- Implement cloud financial management
- Adopt a consumption model
- Measure overall efficiency
- Stop spending money on undifferentiated heavy lifting
- Analyze and attribute expenditure

**Database techniques:**
- **AWS Auto Scaling** — works with Aurora and DynamoDB; automatically scales based on workload so you only pay for what's used
- **Read replicas + Auto Scaling** — scale replica count with read demand instead of over-provisioning fixed capacity

---

### Pillar 5: Sustainability
**Definition:** Minimizing environmental impact of cloud workloads. Includes shared responsibility for sustainability, understanding impact, maximizing utilization.

**6 Design principles:**
- Understand your impact
- Establish sustainability goals
- Maximize utilization
- Anticipate and adopt new, more efficient hardware/software
- Use managed services
- Reduce the downstream impact of your cloud workloads

**Database techniques:**
- **Minimize data movement across networks** — store data close to customers (e.g., cross-Region replicas with RDS/Aurora, DynamoDB global tables) to reduce latency and hardware energy use
- **Reduce and reuse required resources** — connection pooling via **Amazon RDS Proxy** to share/reuse DB connections, reducing CPU/network utilization

---

### Pillar 6: Security
**Definition:** Protecting information and systems — confidentiality/integrity of data, managing permissions, detecting security events.

**7 Design principles:**
- Implement a strong identity foundation
- Enable traceability
- Apply security at all layers
- Automate security best practices
- Protect data in transit and at rest
- Keep people away from data
- Prepare for security events

**AWS Shared Responsibility Model:**
- **AWS responsible for:** security *of* the cloud — Regions, AZs, hardware, software, physical data center security
- **Customer responsible for:** security *in* the cloud — data protection, IAM configuration, compliance validation, infrastructure security controls, security best practices

**IAM concepts:**
| Concept | Description |
|---|---|
| **IAM Users** | Identity representing a person/app; no permissions by default |
| **IAM Policies** | Define allowed actions |
| **IAM Groups** | Manage permissions for multiple users at once |
| **IAM Roles** | Assumable identities for temporary/delegated access |

**IAM best practices:**
- Create individual IAM users for each person; never use root credentials
- Apply principle of least privilege
- Use IAM groups to manage permissions at scale
- Rotate credentials regularly
- Use **AWS Secrets Manager** to create/rotate/access database credentials securely

**Data protection:**
- **Encryption at rest** — AWS KMS integrates with RDS, Aurora, DynamoDB, etc.; use AWS Config managed rules to verify encryption is enabled
- **Encryption in transit** — SSL/TLS, certificate-based encryption between client and DB endpoint

**Network security:**
- Place databases in **private subnets** within a VPC (segmented from public-facing resources)
- Control traffic via **Network ACLs** and **VPC security groups** (inbound/outbound)
- Limit network paths to only what's needed

---

### AWS Well-Architected Tool
A free console-based service providing a consistent process for reviewing architecture against AWS best practices.

**Benefits:**
- Architectural guidance
- Consistent workload reviews
- Identify and implement improvements
- Customizable via **custom lenses** (tailor questions to your tech/governance needs)

**How it works:**
1. Define Workload (a collection of AWS resources/code delivering business value)
2. Conduct Architectural Review (answer pillar-based questions)
3. Apply Best Practices (get a prioritized, risk-ranked improvement plan)

**Additional features:**
- **Milestones** — record workload state at a point in time to track change over time
- **Dashboard view** — summary of issues across your whole portfolio of workloads

---

## MODULE 2: Data Types

**Three data source types:**

| Type | Description | Storage | Example |
|---|---|---|---|
| **Structured** | Highly organized, easy to analyze, supports complex queries | Rows/columns in relational (or some nonrelational) DBs | Product catalog, order table |
| **Semi-structured** | Flexible structure, changeable schema, moderate query complexity | JSON files loaded into a DB engine | Customer records where some have multiple cards, some none |
| **Unstructured** | No predefined structure, needs special tools to catalog/query | Files in object/file stores (e.g., Amazon S3) | Images, videos, documents |

**Database type → data source type support:**

| Database Type | Supports |
|---|---|
| Relational | Structured |
| Key-value | Unstructured |
| Document store | Semi-structured & unstructured |
| In-memory | Structured & semi-structured |
| Graph | Structured, semi-structured, unstructured (any type) |
| Ledger | Structured & semi-structured |
| Wide-column | Structured |
| Time-series | Structured |

---

## MODULE 3: AWS Database Services Overview

AWS provides purpose-built database services spanning **relational** and **nonrelational** categories, plus data access/analysis tools:

- **Relational:** Amazon RDS, Amazon Aurora
- **Nonrelational:** DynamoDB, Keyspaces, DocumentDB, Neptune, Timestream, QLDB, ElastiCache, MemoryDB
- **Data access & analysis:** Amazon Redshift, Amazon Athena

---

## MODULE 4: Relational Databases

### Core Concepts
- Data organized into **tables** (entities); a **column** = field (attribute); a **row** = record (single instance)
- **Primary key** — uniquely identifies each row in a table
- **Foreign key** — references a primary key in another table to build relationships
- Queried using **SQL**
- **Indexing** speeds up queries by grouping data predictably on disk (e.g., an indexed `OrderDate` column turns a 12,000-row scan into a 50-row seek)

**OLTP vs. OLAP:**
- **OLTP (Online Transaction Processing)** — simple, short transactions (insert/update/delete); e.g., bank ATM
- **OLAP (Online Analytical Processing)** — stores historical data for multidimensional analysis; e.g., BI tools

### Amazon RDS
**What it is:** Managed relational database service that reduces infrastructure cost, automates admin tasks (provisioning, patching, backups), and speeds up app development.

**Supported engines:** Amazon Aurora, PostgreSQL, MySQL, MariaDB, Oracle Database, Microsoft SQL Server

**Key features:**
- **Multi-AZ deployments** for availability/durability — automatic failover to standby, DNS-based reconnection (no connection string updates needed)

**Billing (3 parts):**
1. **Instance** — On-Demand (hourly) or Reserved (1–3 yr term discount)
2. **Storage & I/O** — per GB/month + per million requests
3. **Data transfer** — out to internet/other Regions (same-Region transfer between AWS services is free)

### Amazon Aurora
**What it is:** MySQL- and PostgreSQL-compatible relational database built for the cloud — combines enterprise DB performance/availability with open-source simplicity/cost.

**Performance:** Up to **5x faster** than standard MySQL, **3x faster** than standard PostgreSQL — commercial-grade at ~1/10th the cost.

**Key features:**
- Fully managed by RDS (provisioning, patching, backups automated)
- Automatic backups to S3 with point-in-time recovery
- **6 copies of data across 3 AZs** automatically maintained; auto-recovers to a healthy AZ with no data loss
- Up to **15 read replicas** — serve read traffic and support failover
- Multiple security layers: isolation, encryption at rest & in transit
- **Aurora Serverless** — run without managing individual DB instances

**Billing (3 parts):**
1. Instance — On-Demand, Reserved, or Serverless (capacity-based)
2. Storage & I/O — per GB/month + per million requests
3. Data transfer out (same-Region service-to-service transfer is free)

---

## MODULE 5: Nonrelational (NoSQL) Databases

**Key distinction:** NoSQL = "not only SQL" — it describes data modeling, not query language capability.

**Relational vs. Nonrelational comparison:**

| Characteristic | Relational | Nonrelational |
|---|---|---|
| Representation | Multiple tables (rows/columns) | Single table collection (keys/values) |
| Data design | Normalized / dimensional warehouse | Denormalized document/wide-column/key-value |
| Optimization | Storage | Compute |
| Query style | SQL | Many languages; object querying |
| Scalability | Vertical | Horizontal |
| Implementation | OLTP business systems / OLAP | OLTP web/mobile apps |

**Trade-off:** Nonrelational DBs scale horizontally across distributed servers but often use **eventual consistency** — data may not update simultaneously everywhere. Not ideal if strict **ACID** compliance is required.

**Schema flexibility:** Adding a new attribute to a nonrelational DB just means writing new items with that attribute — no need to retroactively update existing records (unlike relational, where a schema change requires updating the table structure).

### Nonrelational Database Types

| Type | Storage Model | Strengths | Weaknesses | AWS Service |
|---|---|---|---|---|
| **Key-value** | Blob values tied to keys, no schema | Flexible, handles many data types, no joins/indexing needed, portable | Hard to do analytical queries; access patterns must be known upfront | **DynamoDB** |
| **Document** | JSON/BSON-like files/collections | Flexible, no upfront schema, scalable | Sacrifices ACID; can't query across files natively | **DocumentDB** (MongoDB-compatible) |
| **In-memory** | RAM-based cache/store | Sub-millisecond/microsecond latency, great for caching/gaming/sessions, scales without downtime | Not for rapidly changing/rarely accessed data; low tolerance for stale data | **ElastiCache**, **MemoryDB for Redis** |
| **Graph** | Nodes (entities) + edges (relationships) | Fast complex relationship retrieval, real-time recommendations | Poor for transactional data; requires learning new query languages | **Neptune** |
| **Ledger** | Immutable, cryptographically verifiable log | Immutable records, auditable change history, legally provable | Can't correct bad data (creates new record instead) | **QLDB** |
| **Time-series** | Time-indexed events | Great for tracking metric changes over intervals | Narrow use case, not flexible | **Timestream** |
| **Wide-column** | Columns/rows grouped into column families | High volume, scalable, fast writes | Difficult with changing requirements | **Keyspaces** (Apache Cassandra-compatible) |

### Amazon DynamoDB
Fully managed, serverless, key-value NoSQL database for high-performance apps at any scale. Features: built-in security, continuous backups, automated multi-Region replication, in-memory caching, data export tools.

### Amazon Keyspaces (for Apache Cassandra)
Fully managed, serverless, wide-column database compatible with Apache Cassandra (uses CQL — Cassandra Query Language). No servers to manage; auto-scales; 99.99% availability SLA; encrypted by default; replicated 3x across AZs; continuous backups with PITR up to 35 days.

### Amazon DocumentDB
Fast, fully managed, MongoDB-compatible document database. Stores data as documents in collections; flexible indexing and ad hoc queries. On-demand pricing (per-second billing); storage auto-scales 10 GB–64 TB; automatic continuous incremental backups with PITR (up to 100% of cluster storage free).

### Amazon Neptune
Fully managed graph database engine, optimized for billions of relationships with millisecond query latency. Supports **3 query languages**: Apache TinkerPop (Gremlin), openCypher, and RDF/SPARQL. Up to 15 read replicas, Multi-AZ support, encryption at rest.

**Billing (4 parts):** instance (on-demand), storage (per GB/month, first 50GB backup free), number of requests, data transfer out.

### Amazon Timestream
Fast, scalable, serverless time-series database for IoT and operational apps — up to 1,000x faster and 1/10th the cost of relational DBs for time-series workloads. Tiered storage (memory store for recent data, magnetic store for historical), automatic lifecycle management, built-in time-series analytics functions, always encrypted (KMS support for magnetic store).

### Amazon QLDB (Quantum Ledger Database)
Fully managed, purpose-built ledger database providing an immutable, cryptographically verifiable transaction history owned by a central trusted authority.

- Writes go first to an **immutable, append-only journal**, then materialize into queryable tables
- Supports **PartiQL** (SQL-like query language)
- Serverless — auto-scales, no capacity provisioning
- Common uses: finance, insurance claims, manufacturing defect tracking, HR/payroll

**Billing:** read/write IO requests, data transfer, journal storage, index storage (all per GB/month or per million requests; no advance provisioning needed).

### Amazon ElastiCache
Fully managed in-memory cache supporting **Redis** and **Memcached** engines.
- **Redis** — complex data types, replication, high availability; ideal for session caching, leaderboards, message queues
- **Memcached** — simpler, for small/static data (e.g., static HTML/CSS/JS)
- Typically sits between app and database to reduce DB load (can cut DB queries by up to ~95%)

**Billing:** On-Demand or Reserved nodes (up to 75% savings); 1 free snapshot, additional snapshots billed per GB/month; charged only for EC2 data transfer in/out (no ElastiCache-node transfer charge).

### Amazon MemoryDB for Redis
Redis-compatible, **durable**, in-memory **primary** database (not just a cache) — delivers microsecond read / single-digit millisecond write latency at high throughput (handles 13+ trillion requests/day, peaks of 160M requests/sec).
- Stores entire dataset in memory + uses a distributed transactional log for durability/consistency/fast recovery across multiple AZs
- Eliminates need to separately manage a cache + a durable DB
- Scales horizontally (sharding for writes, replicas for reads) or vertically; stays online during resizing

---

## MODULE 6: Data Access and Analysis

### Amazon Redshift
Fast, fully managed, cloud data warehouse for analytical data — used for complex queries, BI reporting, and ML, as opposed to storing individual transactional records (that's what relational DBs are for).

**Architecture:** Single **leader node** + multiple **compute nodes**. Client sends SQL to the leader node's endpoint; leader creates parallel jobs for compute nodes; compute nodes process and return results; leader aggregates and returns the final result.

**Key features:**
- Massively parallel, **columnar** architecture — indexed to match analytical query patterns
- **Concurrency scaling** — automatically adds/removes cluster capacity to handle concurrent query spikes
- **Redshift Spectrum** — query data directly in S3 without first loading it into Redshift; extends analytical reach beyond local storage; enables a "rich data platform" architecture without duplicating data

**Billing (4 parts):**
1. On-Demand cluster node pricing (hourly, per node)
2. Concurrency scaling — per-second on-demand rate
3. Reserved Instance pricing — up to 75% savings for 1–3 yr commitment
4. Redshift Spectrum — billed per bytes scanned in S3 (in addition to cluster pricing)
No charge for Redshift ↔ S3 transfer within the same Region for backup/restore/load/unload.

### Amazon Athena
Serverless, interactive query service for analyzing data **directly in Amazon S3** using standard SQL — no infrastructure to manage, no ETL required.

**Key features:**
- Point at S3 data, define schema, query — most results in seconds
- Built on **Presto** with ANSI SQL support; works with CSV, JSON, ORC, Avro, Parquet
- Handles simple one-time queries and complex analysis (joins, window functions, arrays)
- Runs queries in parallel automatically for fast performance
- **Athena Federated Query** — single SQL query across multiple data sources (on-prem or cloud) via Lambda-based data source connectors (open-sourced for DynamoDB, HBase, DocumentDB, Redshift, CloudWatch Logs/Metrics, JDBC sources like MySQL/PostgreSQL)

**Pricing:** **$5 per TB scanned**; no separate storage charge beyond S3 costs. Compressing/partitioning/converting to columnar formats can cut costs 30–90%.

---

## MODULE 7: Choosing the Right Database — Example Mappings

| Business Need | Likely AWS Service |
|---|---|
| Migrate on-prem inventory system to a relational cloud DB | Amazon RDS |
| Gaming website overwhelmed by rapid growth/scale | Amazon DynamoDB |
| Reduce PostgreSQL admin burden from small write I/O overhead | Amazon Aurora (PostgreSQL-compatible) |
| Rapidly gather/discard shopping cart data | Amazon DynamoDB (or ElastiCache for ephemeral session-like data) |
| Data warehouse with automated infra/admin, cost management | Amazon Redshift |
| Near real-time fraud pattern detection for e-commerce | Amazon Neptune (graph relationships) |
| Migrate existing MongoDB workload to a managed cloud service | Amazon DocumentDB |

---

## MODULE 8: AWS Database Migration Tools

### AWS Schema Conversion Tool (AWS SCT)
Used for **heterogeneous migrations** (different source/target engines, e.g., Oracle → Aurora). Automatically converts schema and most code objects (views, stored procedures, functions); flags anything it can't auto-convert for manual fixing.

- Provides an **executive summary** to help estimate migration effort across DBAs, developers, testers, stakeholders
- Can scan application source code for embedded SQL and convert it too (cloud-native code optimization)
- Never modifies the source database — only works on a cached copy

> Not needed for **homogeneous migrations** (e.g., Oracle → Oracle) — skip straight to AWS DMS.

### AWS Database Migration Service (AWS DMS)
Migrates/replicates the actual **data** to AWS. Source database stays **fully operational** during migration (minimal downtime).

- Supports both **homogeneous** (Oracle → Oracle) and **heterogeneous** (Oracle → Aurora) migrations
- Can **fan out** — migrate to multiple targets simultaneously (e.g., on-prem Oracle → Aurora + Redshift + S3, all at once, no downtime)
- Two migration modes:
  1. **Full-load migration** — stop the system, transfer all data at once
  2. **Ongoing replication / change data capture (CDC)** — initial transfer, then continuous replication of changes (enables running old and new systems in parallel)

**SCT + DMS together:** SCT converts the schema → DMS migrates the data. For homogeneous migrations, native database tools may still be needed to migrate remaining schema elements DMS doesn't create automatically.

---

## MODULE 9: Database Architecture — Server-Based vs. Serverless

### Server-Based Architecture
Traditional model — you manage servers, patch/update them, and scale by purchasing additional capacity. High availability/fault tolerance can be costly to implement.

**AWS services covered:** Amazon RDS, Amazon EC2 (self-hosted DB engine)

**Benefits — Developer perspective:**
- Predictable/constant compute workloads can be more cost-effective with server-based billing
- Easier debugging/testing — full visibility into backend processes; no need to replicate a serverless environment
- Fewer integration points than distributed serverless designs — simpler troubleshooting

**Scaling:** RDS offers built-in instance monitoring; vertically scale by resizing to a bigger instance (18+ size options across MySQL/PostgreSQL/MariaDB/Oracle/SQL Server) — app can generally stay online during resize since RDS manages the scaling.

**Best fit:** Large apps with fairly consistent/predictable workloads; legacy applications hard to re-architect; long-running computations/deep analysis.

### Serverless Architecture
Third-party-hosted compute model — no server management; automatic scaling; built-in high availability and fault tolerance.

**AWS services covered:** Amazon DynamoDB, Amazon Aurora Serverless

**Benefits — Developer perspective:**
- No backend infrastructure to manage — reduced liability, no sysadmin work
- Automatic scaling with traffic — no capacity provisioning needed
- Application flexibility — can migrate individual features/workloads to serverless on-demand, freeing production resources for critical tasks

**Scaling:** Event-driven. E.g., DynamoDB Auto Scaling uses CloudWatch to monitor read/write capacity metrics and automatically adjusts throughput as traffic changes.

**Best fit:** New applications made of short-running, single-purpose tasks — can save significant time/money vs. equivalent server-based design.

---

### Quick Reference Summary
- **Well-Architected Framework** = questions + six pillars + design principles (a review conversation, not an audit)
- **Reliability** = availability + resiliency (Multi-AZ, read replicas, cross-Region replicas, global tables) + disaster recovery (RPO/RTO, 4 DR strategies)
- **Relational** (RDS, Aurora) = structured data, ACID, SQL, vertical scaling. **Nonrelational** (DynamoDB, DocumentDB, etc.) = flexible schema, horizontal scaling, often eventual consistency
- Match database type to **data characteristics** and **access patterns** — there's no one-size-fits-all
- **SCT** converts schema (heterogeneous migrations only); **DMS** migrates data (any migration type) with minimal source downtime
- **Server-based** (RDS/EC2) = predictable workloads, full visibility, manual scaling. **Serverless** (DynamoDB, Aurora Serverless) = event-driven auto-scaling, no server management, ideal for spiky/short-task workloads