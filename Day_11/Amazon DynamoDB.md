# Amazon DynamoDB — Getting Started Course Notes

## 1. Introduction to Amazon DynamoDB

**What it is**
DynamoDB is a **fully managed, serverless NoSQL database service** delivering consistent, **single-digit millisecond performance at any scale**. It supports both **key-value** and **document** data models with a flexible (schemaless, except for the primary key) structure. Data is automatically distributed across multiple Availability Zones for durability and high availability.

**Problems it solves**

| Challenge | How DynamoDB Solves It |
|---|---|
| Unpredictable scaling requirements | Handles traffic spikes and data growth without admin overhead |
| Performance inconsistency | Delivers predictable millisecond-level response times regardless of data size/load |
| Infrastructure management burden | Fully managed — no servers, patches, or manual scaling |
| Complex data distribution | Simplifies multi-Region distribution while maintaining consistency and availability |

**Key benefits**
- **Automatic scaling** — handles hundreds to millions of requests without manual intervention
- **Fully managed** — no hardware provisioning, patching, or cluster scaling
- **Built-in high availability** — global tables replicate across Regions; **99.999%** availability rate; fast local read/write in each Region
- **Flexible data model** — schemaless except for required primary key; each item can have different attributes
- **Comprehensive security** — encryption at rest & in transit, IAM-based fine-grained access control, **attribute-based access control (ABAC)** using tags, automatic encryption with AWS-managed keys
- **Automatic backups** — point-in-time recovery (PITR) to any point in the last **35 days**
- **AWS service integration** — Lambda, API Gateway, Step Functions; zero-ETL integration with Redshift and OpenSearch Service
- **Event-driven capabilities** — **DynamoDB Streams** capture table changes in real time to trigger workflows
- **Transaction support** — full **ACID** compliance across multiple items and tables

**Pricing model**
- **Pay-for-what-you-use** — no minimum fees or mandatory usage
- Two capacity modes:
  - **On-demand** — pay per read/write request; best for unpredictable/intermittent traffic
  - **Provisioned** — reserve capacity in advance; best for predictable, consistent traffic
- Cost factors: read/write requests, storage used (**Standard** and **Standard-Infrequent Access** table classes), data transfer out, global tables (multi-Region), backup/recovery, streaming/export, S3 import/export, warm throughput capacity for infrequent workloads
- Free tier available for exploration before production use

---

## 2. Architecture and Use Cases

**AWS service integrations**

| Service | Purpose |
|---|---|
| **Lambda** | Process table data and respond to data changes |
| **API Gateway** | Create secure web/mobile interfaces to DynamoDB data |
| **IAM** | Control read/write access to tables |
| **CloudWatch** | Monitor performance, usage, and alert on issues |
| **AWS AppSync** | Extend GraphQL APIs; keeps mobile/web data consistent, even offline |
| **Amazon S3** | Store large file references in DynamoDB while keeping actual files in cost-effective S3 storage |

**Core technical concepts**
- **Tables and items** — tables hold items (≈ rows) and attributes (≈ columns); schemaless except for required primary key
- **Primary key structure**
  - **Simple primary key** — partition key only
  - **Composite primary key** — partition key + sort key
- **Data distribution** — partition keys distribute data across storage partitions, enabling horizontal scaling and consistent performance at growth
- **Secondary indexes**
  - **Global Secondary Index (GSI)** — can have different partition and sort keys than the base table
  - **Local Secondary Index (LSI)** — must share the same partition key as the base table
- **Data operations** — Query, Scan, and Transactions to retrieve/add/update/delete items
- **Capacity modes** — On-demand (auto-handles traffic) vs. Provisioned (specify expected capacity)
- **Global tables** — automatic multi-Region replication with built-in write conflict detection/resolution for active-active deployments
- **Backup and recovery** — point-in-time recovery + on-demand backups, restorable to any point in the last 35 days
- **Warm throughput capacity** — cost-effective mode keeping tables ready for infrequent/unpredictable workloads (good for dev/test and low-traffic production tables)
- **Security and protection** — encryption at rest & in transit; fine-grained IAM access control

**Typical use cases**
- **Gaming applications** — player profiles, leaderboards, game state (single-digit ms latency at scale)
- **Session management** — web/mobile session data with auto-scaling for millions of concurrent users
- **Digital commerce** — shopping carts, product catalogs, user preferences
- **Real-time analytics** — IoT data, user activity tracking, real-time bidding
- **Media metadata** — flexible schema for video/image/article metadata
- **Mobile backend** — user profiles, app settings, cross-device data sync

**Additional considerations**
- **Security measures**: configure access policies; enable encryption at rest (AWS KMS) and in transit; set up VPC endpoints for network security/privacy
- **Monitoring**: use CloudWatch to track performance and usage
- **Backup strategies**: enable PITR and on-demand backups for data protection

---

## 3. How to Use Amazon DynamoDB (Console Walkthrough)

### Demo 1 — Creating a Table and a Global Secondary Index
1. Search **DynamoDB** in the AWS Console → open it
2. Navigation pane → **Tables** → **Create table**
3. Set **Table name**: `GameScores`
4. Set **Partition key**: `UserId` (String)
5. Set **Sort key**: `GameTitle` (String)
6. Use **Default settings** → **Create table**
7. Wait for creation to complete (banner shows progress; refresh to check status)

**Create a GSI:**
1. Open the `GameScores` table → **Actions** → **Create index**
2. Set **Partition key**: `GameTitle` (String)
3. Set **Sort key**: `TopScore` (Number)
4. Set **Index name**: `TopScores-GSI`
5. Leave other settings default → **Create index**
6. Wait until status shows **Active**

### Demo 2 — Querying and Scanning Operations

**Add items (form view):**
1. Navigation pane → **Explore items** → select `GameScores` → **Create item**
2. Set `UserId` = `123` (partition key), `GameTitle` = `Comet Quest` (sort key)
3. **Add new attribute** → Number → `TopScore` = `0`
4. **Add new attribute** → Number → `Wins` = `0`
5. **Add new attribute** → Number → `Losses` = `1`
6. **Create item**

**Add items (JSON view):**
1. **Create item** → switch to **JSON view**
2. Paste JSON (see example below) → **Create item**

```json
{
  "UserId": { "S": "201" },
  "GameTitle": { "S": "Comet Quest" },
  "TopScore": { "N": "200" },
  "Wins": { "N": "3" },
  "Losses": { "N": "1" }
}
```

**Query operation** (requires at least the partition key):
1. Select **Query** → enter `UserId` = `123` → **Run**
2. Returns only items matching that partition key — efficient, searches within one partition

**Scan operation** (no partition key required):
1. Select **Scan** → optionally **Add filter** (e.g., `TopScore` Number **Greater than** `100`) → **Run**
2. Without a filter, Scan returns **all items** in the table (less efficient than Query)

**Edit an item:**
1. Select the item's checkbox → **Actions** → **Edit item**
2. Update attribute values (e.g., `TopScore` = `399`, `Wins` = `1`) → **Save and close**

**PartiQL (SQL-compatible query language for DynamoDB):**
1. Navigation pane → **PartiQL editor** → select table → **Query table**

```sql
-- Query the base table
SELECT * FROM "GameScores" WHERE "UserId" = '201' AND "GameTitle" = 'Comet Quest'
```

```sql
-- Query the Global Secondary Index
SELECT "GameTitle", "TopScore", "UserId"
FROM "GameScores"."TopScores-GSI"
WHERE "GameTitle" = 'Comet Quest'
ORDER BY "TopScore" DESC
```

### Demo 3 — Enabling Point-in-Time Recovery (PITR)
1. Open the `GameScores` table → **Backups** tab
2. **Edit** → check **Turn on point-in-time recovery**
3. Leave default retention period of **35 days**
4. **Save changes**
5. Verify **Backups** tab shows PITR status = **On**

### Demo 4 — Cleaning Up Resources
1. Navigation pane → **Tables** → select `GameScores`
2. **Actions** → **Delete table**
3. Type `confirm` in the confirmation dialog → **Delete**
4. Wait for status to change to **Deleting**, then verify the table no longer appears in the list

---

### Quick Reference Summary
- DynamoDB = fully managed, serverless **NoSQL** database with single-digit ms latency at any scale
- **Primary key** = simple (partition key) or composite (partition key + sort key)
- **GSI** = different partition/sort keys allowed; **LSI** = must share base table's partition key
- **Query** = efficient, needs partition key; **Scan** = reads entire table (or filtered), no key required
- **PartiQL** = SQL-compatible query language for tables and indexes
- PITR = continuous backup, restorable to any point in the last **35 days**
- Pricing: **On-demand** (pay per request, unpredictable traffic) vs. **Provisioned** (reserved capacity, predictable traffic)
- **Global tables** = multi-Region, active-active replication with automatic conflict resolution
- Integrates with Lambda, API Gateway, IAM, CloudWatch, AppSync, and S3