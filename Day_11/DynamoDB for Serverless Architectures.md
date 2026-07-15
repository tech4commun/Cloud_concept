# Amazon DynamoDB for Serverless Architectures — Course Notes

## 1. Introduction to DynamoDB for Serverless Architectures

### Relational vs. NoSQL — the historical shift
- **Relational databases** were designed when storage was expensive — they minimize stored data volume and use CPU at query time to assemble relationships (via SQL).
- As storage got cheaper relative to CPU, **NoSQL databases** emerged: they pre-compute/store answers for a known set of access patterns, trading storage space and I/O for reduced CPU load and faster, simpler retrieval (typically via a simple API, not SQL).

### What is DynamoDB?
A **serverless, fully managed NoSQL** database service designed for **OLTP** (Online Transactional Processing) workloads.

- Fully managed — availability, durability, and scalability built-in; no servers to manage
- Manageable via Console, CLI, or SDK
- Built-in security controls, monitoring metrics, easy integration with other AWS services
- Available across AWS Regions globally

### DynamoDB vs. OLTP vs. OLAP
- **OLTP** — live/interactive operations, "hot" active data, smaller objects, known query patterns needing fast answers → DynamoDB's sweet spot, with very consistent low latency
- **OLAP** — complex, unpredictable analytical questions not needed instantly → pair DynamoDB with a purpose-built analytics service instead

### Where DynamoDB fits among AWS database services

| Service | Use case | SQL/NoSQL | Summary |
|---|---|---|---|
| **Amazon RDS** | Transactional | SQL | Relational DB service; engines: Aurora, PostgreSQL, MySQL, MariaDB, Oracle, SQL Server, IBM Db2 |
| **Amazon DynamoDB** | Transactional | NoSQL | Document/key-value store |
| **Amazon Redshift** | Data analytics/relationships | SQL | Fast, fully managed data warehouse |
| **Amazon Neptune** | Data analytics/relationships | NoSQL | Fully managed graph database for highly connected datasets |
| **Amazon ElastiCache** | In-memory data store/cache | NoSQL | Serverless, fully managed cache (Valkey-, Memcached-, Redis OSS-compatible) |
| **Amazon S3 + Athena** | Object storage + serverless SQL | — | S3 as a serverless key-based store; Athena for ad hoc SQL over S3 data |

**Key DynamoDB characteristics:**
- Flexible schema — JSON document or key-value structures
- Supports event-driven programming
- Horizontal scaling
- Fine-grained access control
- Deep AWS service integration

---

## 2. How Amazon DynamoDB Works

### Tables, Items, Partitions
- Data lives in **tables**; tables hold **items** (≈ rows); items have **attributes** (≈ columns)
- Every item requires at least a **partition key**; an optional **sort key** can also be required
- **Partition key + sort key (if defined) = primary key**, which must be unique
- Non-key attributes are **flexible** — they can vary from item to item (a core NoSQL advantage)
- DynamoDB **automates sharding** — the partition key's value is hashed to distribute items across partitions, enabling horizontal scaling with no admin effort
- Partitioning is managed internally but understanding it explains design best practices

### Data types for attributes
- **Scalar:** number, string, binary (base64), boolean, null
- **Sets:** number set, string set, binary set — entries must be the same type and unique; sets do **not** preserve order
- **Document types:**
  - **Map** — nested, unordered attributes of mixed type (like a JSON object)
  - **List** — ordered values of any type (like a JSON array)
  - Can nest up to **32 levels deep**
- **Primary key attributes must be** string, number, or binary — Map/List entries need to be exposed as separate attributes to be used as keys

### Primary key types
| Type | Description | Example use |
|---|---|---|
| **Partition key only (simple)** | Single attribute uniquely identifies each item | Sensor table: `SensorId` as partition key, storing current reading |
| **Partition key + sort key (composite)** | Partition key distributes load; sort key orders items within a partition key | Sensor history: `SensorId` (partition) + `Timestamp` (sort) — efficiently query recent readings for a sensor |

### Durability and Availability
- Data is automatically replicated across multiple SSDs in separate, fault-isolated servers/data centers (Availability Zones)
- A write does not return successfully until data is redundantly stored (**at least 2 copies**); other copies converge within ~1 second
- Failed servers are removed from the replication set within seconds and auto-replaced
- SLA: **99.99%** (four nines) availability

### Consistency
- **Read-after-write consistency** — writes are always consistent; **reads** can be **strongly consistent (SC)** or **eventually consistent (EC)**
- **EC is the default**; SC is opt-in per read request
- With SC reads, DynamoDB routes the request to a copy known to have the latest data — always returns the up-to-date value
- With EC reads, the request may return stale or current data depending on which replica is hit
- **SC reads cost more** (concentrated on fewer replicas) and may be briefly unavailable during failure scenarios
- SC reads are **not a locking mechanism** — data can still change between your read and a subsequent write
- **Best practice:** design around EC reads wherever possible

### Request Throughput
- You (optionally) specify provisioned throughput; DynamoDB distributes data across enough partitions to meet it
- **RCU** — 1 read capacity unit = 1 strongly consistent read/sec for an item ≤4KB (EC reads cost **half** as much — two 4KB EC reads = 1 RCU)
- **WCU** — 1 write capacity unit = 1 write/sec for an item ≤1KB
- Updating a single attribute requires rewriting the **entire item** (full WCU cost)
- Throughput is generally divided evenly across partitions — design for evenly distributed request patterns
- **Burst capacity** — unused throughput "saved up" like carry-over minutes
- **Adaptive Capacity** — intelligently borrows unused throughput from less-active keys to help busier keys
- **Hard limits per item:** max **3,000 RCU** or **1,000 WCU** (or linear combination) — regardless of table-level provisioning
- Exceeding provisioned/allowed throughput → **throttling** (client must retry)

### Basic Item Requests

| Operation | Purpose |
|---|---|
| `PutItem` | Insert a new item or overwrite an existing one by primary key |
| `UpdateItem` | Change specific attributes — but consumes WCUs for the entire item |
| `DeleteItem` | Delete by primary key — costs the same WCUs as creating the item |
| `GetItem` | Retrieve a single item by primary key |
| `BatchWriteItem` / `BatchGetItem` | Send multiple puts/gets/deletes in one call — more efficient than many individual requests |
| `Scan` | Reads the **entire table** — inefficient, use sparingly and rate-limit |
| `Query` | Requires a composite (partition + sort key) table; specify partition key + a sort key condition (equal, less than, greater than, begins with, etc.) — efficient, and RCUs are calculated on the **summed size of the result set** (rounded to nearest 4KB), not per-item |

- All reads can be EC (default) or SC
- You can request only specific attributes or filter results — reduces network transfer and simplifies your app, but **does not reduce throughput cost** (still billed on the full result set)

### Secondary Indexes
Allow querying from a different primary key perspective, or fetching a subset of items/attributes.

| Type | Local Secondary Index (LSI) | Global Secondary Index (GSI) |
|---|---|---|
| Partition key | Same as base table | Can be different from base table |
| Sort key | Different from base table | Any scalar attribute |
| Storage | Same partition as base table; shares base table's provisioned capacity | Separate "table" DynamoDB replicates to; own provisioned throughput |
| Size limit | ~**10GB** per partition key (base table + LSI combined) | No such limit |
| Consistency | Supports both EC and SC reads | **EC reads only** |
| Lifecycle | Must be defined **at table creation**; cannot be added/removed later | Can be created or deleted **anytime** |
| Uniqueness | — | Key values **do not** need to be unique |

**LSI example** — PatientSurvey table (`RepId` partition, `PatientId` sort, `ContactDate` attribute) → LSI `PatientSurveyByRepAndDate` (`RepId` partition, `ContactDate` sort) enables querying patients surveyed by a rep within a date range.

**GSI example** — Infections table (`PatientId` partition key) → GSI `InfectionsByCity` (`City` partition, `Date` sort) enables querying infection reports by city and date — a completely different access pattern than the base table.

### DynamoDB Streams
- Records all table writes as an ordered changelog (compatible with the **Kinesis Client Library**)
- Strictly ordered; shard count grows with the table
- Configurable level of detail; **durable for 24 hours**
- Example: user updates a preference → the change is captured in the stream for downstream processing (e.g., an ad server reacting to preference changes)

---

## 3. Operating Amazon DynamoDB

### Resilient Client Behavior
- **400-level errors** — often need a fix before retrying (e.g., missing required parameters); but throttling (`ProvisionedThroughputExceeded`) is retryable
- **500-level errors** — DynamoDB-side issues; safe to retry until success

### Tuning Retries
- AWS SDKs include built-in retry logic with sensible defaults — tune for your use case: max retry attempts, timeouts, exponential backoff (with jitter)
- Choose between "fail fast + fallback" vs. "wait and keep trying" based on your app's needs
- For throttling, aim to "slide into the next second" of capacity
- **Avoid the "thundering herd"** — don't let many retries hit at the same moment after a temporary issue resolves (causes renewed throttling)

### Handling Errors in Batch Operations
- `BatchGetItem`/`BatchWriteItem` are wrappers around `GetItem`/`PutItem`/`DeleteItem`
- Unprocessed items are returned to you (`UnprocessedKeys` for reads, `UnprocessedItems` for writes)
- Implement your own retry loop **with exponential backoff** around unprocessed items
- Most common failure cause: insufficient provisioned read/write capacity on the table

### DynamoDB Auto Scaling
- Uses **AWS Application Auto Scaling** to dynamically adjust provisioned RCU/WCU based on actual traffic
- RCU and WCU managed **separately** — set a minimum, maximum, and target utilization (%) for each
- **Enabled by default**; highly recommended everywhere
- Great for smoothing seasonal/predictable load patterns and providing insurance against unexpected spikes (reacts faster than manual alarm response)
- **Reactive, not instant** — takes time to recognize a pattern; can't fully prevent throttling during a sudden spike
- Set target utilization based on historical spikiness to maintain a buffer
- For known upcoming large events (product launch, sale, big ingestion job), **pre-emptively raise the minimum** capacity ahead of time, then scale back down after
- Auto Scaling parameters can be changed programmatically or scheduled (e.g., scale down nights/weekends)

### Global Tables
- Fully managed **multi-Region** replication — you specify target Regions and DynamoDB handles table creation and ongoing replication
- Enables an SLA of **five nines (99.999%)**
- Adds cost/complexity — evaluate whether the extra "9" is justified (tables are already highly available in-Region)
- **More compelling reason:** extremely low latency for globally distributed clients
- **Multi-master, last-write-wins conflict resolution** — cross-Region **strong consistency is not possible**
  - For strongly consistent read-after-write, direct clients to read from the same Region they write to
  - Useful for warm DR failover, or geo-routing clients to their nearest endpoint (e.g., user profiles rarely updated from multiple places at once)
- Each Region's replica needs enough **write capacity** to absorb all global write traffic (replicated writes count) — Auto Scaling strongly recommended
- Example: conflicting status updates from two Regions converge to whichever has the latest timestamp — stale updates are discarded even if they arrive "later"

### Expiring Items with TTL (Time to Live)
- Automatically deletes items past a configured expiry — **no WCUs consumed**
- Configure an attribute (containing an epoch timestamp) as the expiry flag
- Deletion typically happens within **a day or two** of the expiry time (not instant)
- If your app needs to immediately ignore expired-but-not-yet-deleted items, check the epoch value against current time client-side
- **Popular pattern:** move expiring items to "cold" storage (e.g., S3) — TTL deletes are written to the stream; a Lambda trigger reads them and writes to S3 (often via Kinesis Data Firehose)

### Access Control
- Tightly integrated with **IAM** — fine-grained control down to individual items/attributes
- **Best practice:** principle of least privilege
- **Best practice for VPC clients:** use a **VPC Endpoint** for DynamoDB to avoid traversing the public internet and improve isolation

### DynamoDB Accelerator (DAX)
- API-compatible **in-memory cache** for DynamoDB, accessed via a separate endpoint
- Highly available cluster of nodes, accessible only within your VPC
- **Write-through cache** — writes made through DAX are immediately available for subsequent EC reads
- **Strongly consistent reads are not cached**
- Reduces RCU consumption, smooths spiky/imbalanced read loads, and cuts latency from single-digit ms to **sub-millisecond**

### Backup and Restore
- **On-demand backups** — nearly instant, triggered manually
- **Point-in-time recovery (PITR)** — rolling 35-day window, restore to any second within that window
- Neither backup type consumes table capacity
- Restore creates a **new table** (or delete the original first) — commonly used to compare/repair data rather than overwrite in place
- Restore time varies with partition density but is typically well under 10 hours (parallelized, doesn't scale linearly with table size)
- **Recommendation:** PITR for most production tables, supplemented with on-demand backups for longer-term retention

### Monitoring and Troubleshooting Best Practices
- Log the AWS error code from every operation
- Enable **CloudTrail** to capture DynamoDB control operations (create/update table, etc.)
- Use **CloudWatch metrics** to monitor table performance
- Recommended alarms: `SuccessfulRequestLatency`, throttling events, capacity consumption, user errors, system errors

---

## 4. Design Considerations

### Uniform Workloads (avoiding hot partitions)
- Choose a partition key with **high cardinality** (many unique values) for even distribution
- Poor example: `CountryCode` when your app only serves 2 countries → only 2 partition values
- Better example: `UserId`
- Consider **appending a calculated/random value** to the partition key to further spread traffic
- A **hot key** overloads its partition; **Adaptive Capacity** helps some, but design for even distribution first
- **DAX** smooths hot *read* activity but not writes; for hot writes, consider **queue-based load leveling** (e.g., an SQS queue in front of the write path)

### Hot and Cold Data
- Separate frequently-accessed ("hot") data from rarely-accessed ("cold") data
- **Rolling table design** — e.g., one table per month/quarter; older tables get lower provisioned capacity
- Delete old tables when no longer needed (avoids paying WCUs for individual deletes)
- Consider moving cold data to a cheaper tier like **S3**

### Managing Large Attributes
- DynamoDB items are capped at **400KB**; optimal item size is **1–4KB**
- Large items localize "hot" activity to a single partition
- Strategies:
  - Store large objects in **S3**, keep only a reference/metadata in DynamoDB (DynamoDB as an index over S3)
  - **Compress** large objects and store as a binary attribute, separate from smaller metadata attributes
  - **Chunk** large objects into multiple smaller items — enables parallel read/write and spreads traffic across partitions (metadata item tracks chunk count; app reassembles chunks)
- For unavoidable hot keys (e.g., a viral voting item), use **write sharding / "scatter-gather"**: split the key across several shard-items, write to a random shard, and read+aggregate all shards for a summarized result

### Using Indexes Thoughtfully
- **Use indexes sparingly** — every index adds WCU cost for extra writes
- **Project attributes selectively** — only include what you frequently need; use sparse indexes to save throughput
- **LSI:** limited to ~10GB per partition key collection; fixed at table creation; can't be modified/deleted later
- **GSI:** more flexible (any LSI can be modeled as a GSI); no strong consistency; watch for **GSI back-pressure** — if the GSI can't keep up with writes, the base table gets throttled; GSIs need the same even-distribution design as base tables; GSI throttling is independent of the base table (useful for isolating heavy Scan activity)
- **Recommendation:** prefer GSIs unless you have a justified need for strong consistency

### Optimistic Locking with Version Number (Optimistic Concurrency Control)
Prevents lost updates when multiple clients might modify the same item concurrently:
1. Read the item and note its version number (`versionNum = 0`)
2. Validate and compute the new state in memory
3. Increment the version number (`versionNum = 1`)
4. Write back with a **conditional expression** — only succeeds if the version hasn't changed since your read
5. If the condition fails, **restart from step 1**

Also called the **read-modify-write** pattern. Works fine with EC reads.

### One-to-Many Tables (avoid oversized set attributes)
- If an item stores a large/growing set (e.g., forum replies as a string set), it risks exceeding item size limits and wastes throughput fetching unneeded data
- **Solution:** split the set into a **separate table** of individual items (e.g., a `Replies` table instead of a set attribute on the thread)
- Schema flexibility even allows mixing different item "types" in one table, using indexes for different query needs

### Varied Access Patterns (splitting rarely- vs. frequently-accessed attributes)
- If a table mixes large, static attributes (e.g., company info, mission statement, logo) with small, frequently-changing ones (e.g., stock price), split them into separate tables — or consider modeling the frequently-accessed data as a **GSI** with only the needed attribute projected

### Migrating Existing Data to DynamoDB
**If downtime is acceptable:** export → transform → import → cut over.

**If zero downtime is required — live migration steps:**
1. Create the DynamoDB table(s)
2. Modify the application to write to **both** the source and DynamoDB
3. Perform a back-fill (bulk copy of existing data)
4. Verify
5. Modify the application to **read** from DynamoDB
6. Modify the application to **write only** to DynamoDB
7. Shut down the deprecated data store

**Tools for export/transform/import (back-fill):** AWS Glue, Amazon EMR with Hive (DynamoDB connector)

**AWS Database Migration Service (DMS):**
- Moves data from sources like Cassandra, MongoDB, and various relational databases into DynamoDB
- Performs an initial full copy, then continues applying ongoing changes
- Cut over your app to the DynamoDB SDK once ready
- **Use migration as an opportunity to remodel data** for DynamoDB's strengths — especially important coming from a relational source

**Example: Shopping cart migration**
- Relational model: two normalized tables (carts, cart items)
- DynamoDB model: aggregate into a **single item** in a single table
- Update pattern: `GetItem` current cart → build new version with the added product + incremented version number → conditional write based on version (optimistic concurrency control) → on failure, retry from the read

---

## 5. Serverless Architecture Patterns

### AWS Serverless Platform components
Lambda, API Gateway, S3, DynamoDB, SNS/SQS, Step Functions, Kinesis, Athena, plus supporting tooling.

**Advantages:** no server management, flexible scaling, high availability, no idle capacity.

### DynamoDB Streams + Lambda
Streams characteristics:
- Stream of item changes, **exactly-once guaranteed delivery**, strictly ordered by key
- Durable, scalable, fully managed
- **24-hour retention**, sub-second latency
- Serves as an **event source for Lambda**

The stream is sharded to scale with table throughput; Lambda scales automatically to keep up. Any write can become a trigger — Lambda can filter and act on changes. Together, **DynamoDB Streams + Lambda = reliable "at least once" event delivery**, useful for pipelines feeding other services (e.g., a data lake).

### Querying in Microservice Architectures
**Challenge:** data is segregated across microservices' own databases, and the operational data view often doesn't fit querying/reporting needs.

**Solution approach:**
- **Separate operational and querying views**
- **Polyglot persistence** — use the right database for each job
- **CQRS (Command Query Responsibility Segregation)** — different materialized views exposed as their own microservices

**Typical pattern:** live traffic hits DynamoDB (for availability/durability/scale) → DynamoDB Streams trigger Lambda → Lambda pushes processed data to other purpose-built stores, e.g., a separate DynamoDB table for a near-real-time dashboard, or Redshift (via Kinesis Data Firehose + S3) for BI. Each query view is exposed as its own microservice via API Gateway and scales independently.

### Worked Example: Time-Series Sensor Data

**Requirement:** ingest readings from thousands of sensors; retrieve a given sensor/timestamp reading in <10ms; retain 30 days, then delete.

**Initial architecture:**
Sensors → Kinesis Data Stream → Lambda → DynamoDB table (with TTL for 30-day expiry) → expired items streamed via Lambda → Kinesis Data Firehose → S3 (long-term storage) → users query via a static S3-hosted website authenticated with Cognito → CloudWatch for operational metrics.

**Cost problem identified:**
- Data rate: 100,000 points/sec; ~2.5TB/month storage
- Estimated costs: Lambda $2K–5K/mo, Kinesis ~$5K/mo (100 shards), **DynamoDB ~$50K/mo (100,000 WCUs)**
- Root cause: each data point is only ~50 bytes, but 1 WCU = 1KB — so each tiny write only uses **4.88%** of a WCU, wasting the rest

**Revised (optimized) architecture — queue-based load leveling:**
- Batch multiple data points into a **single item** (using a List attribute) via Lambda batch size
- Use `BatchWriteItem` for connection efficiency
- Still relies on TTL for expiry; compression is an additional option
- **Result:** aggregating 10 data points per item saves **~90%** of DynamoDB WCU cost — over **$500K/year** in savings

**Scaling considerations:**
- More sensors → add Kinesis shards (Lambda auto-scales with shards; DynamoDB auto-scales space/throughput; use Auto Scaling for gradual growth, explicit provisioning for large spikes); TTL deletes expired data at no capacity cost
- More events per sensor → Lambda creates "denser" items, packing more data points per DynamoDB item

---

### Quick Reference Summary
- DynamoDB = serverless NoSQL, built for **OLTP**; pair with Redshift/Athena/Neptune for OLAP/analytics needs
- **Partition key** (required) + optional **sort key** = primary key; high-cardinality partition keys avoid hot spots
- **RCU** = 1 strong read/sec (or 2 eventual) per 4KB; **WCU** = 1 write/sec per 1KB; hard cap **3,000 RCU / 1,000 WCU per item**
- **EC reads** = default, cheaper, no locking guarantee; **SC reads** = costlier, always current, single-Region only
- **LSI** = same partition key, different sort key, fixed at table creation, ~10GB/partition-key limit; **GSI** = flexible keys, own throughput, EC-only, can be added/removed anytime
- **Auto Scaling** = reactive, good for gradual/seasonal load — not sudden spikes (pre-raise minimums for known events)
- **Global Tables** = multi-Region, multi-master, last-write-wins, no cross-Region strong consistency
- **TTL** = free automated deletion; pair with Streams + Lambda + Firehose to archive to S3
- **DAX** = write-through in-memory cache, sub-ms latency, EC reads only
- **Backup**: PITR (35-day rolling) + on-demand snapshots, neither consumes table capacity
- Design principles: uniform workloads, separate hot/cold data, keep items small (1–4KB ideal, 400KB max), use indexes sparingly, optimistic locking for concurrency, split oversized sets into separate tables
- Migration: live cutover (dual-write → backfill → verify → switch reads → switch writes → decommission) or use **AWS DMS**
- Serverless pattern: **DynamoDB Streams + Lambda = reliable at-least-once event delivery** for CQRS/polyglot persistence architectures
- Watch item size vs. WCU granularity — batching small records into fewer, denser items can cut costs dramatically