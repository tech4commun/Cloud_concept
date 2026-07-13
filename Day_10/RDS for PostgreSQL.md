# Amazon RDS for PostgreSQL — Troubleshooting Course Notes

## 1. Overview of Amazon RDS for PostgreSQL

**What it is**
Amazon RDS makes it easier to set up, operate, and scale PostgreSQL deployments in the cloud, with cost-efficient, resizable hardware capacity. RDS manages complex administrative tasks: software installation/upgrades, storage management, replication for HA/read throughput, and backups.

**Benefits**
- **Managed deployments** — preconfigured parameters, seamless app integration, event notifications, automatic patching
- **Monitoring and metrics** — CloudWatch metrics (compute, memory, storage, I/O, connections) at no extra charge
- **Backup and recovery** — automated backups for PITR, configurable retention, user-initiated snapshots
- **Isolation and security** — KMS encryption, VPC network isolation, configurable firewall settings

**Key concepts and terminology**

| Term | Definition |
|---|---|
| **DB instance** | Isolated database environment in the cloud; basic building block of RDS; can hold multiple databases |
| **DB engine** | The relational database software (Aurora, PostgreSQL, MariaDB, MySQL, Oracle, SQL Server) |
| **DB instance class** | Determines compute/memory capacity (type + size, e.g., `db.m6g.2xlarge`) |
| **DB instance storage** | Uses Amazon EBS volumes; RDS auto-stripes across multiple volumes for performance |
| **Parameter group** | Container for engine configuration values applied to one or more instances |
| **DB endpoint** | Host name/DNS + port used by applications to connect |
| **DB subnet group** | Collection of (typically private) subnets in a VPC designated for DB instances |

---

## 2. Gathering Information with the AWS Management Console

**Multi-AZ deployment types**
- **Multi-AZ DB instance deployment** — single standby replica in a different AZ, synchronous replication, automatic failover
- **Multi-AZ DB cluster deployment** — one writer + two readable standby instances across 3 AZs; more read capacity and lower write latency than instance deployment

**Console workflow for gathering info**
1. Sign in → search **RDS** → select **RDS**
2. Select the correct **Region** (required when contacting AWS Support)
3. **Dashboard** — view resource counts vs. quotas; request limit increases if needed
4. **Databases** → select instance → **Summary** — engine type, class, role
5. **Connectivity & security** tab — endpoint & port, Availability Zone, VPC, subnets, security groups, public accessibility
6. **Monitoring** section — CPU utilization, connections, I/O activity graphs
7. **Logs & events** section — CloudWatch alarms, recent events
8. **Configuration** tab — detailed instance settings
9. **Replication** section (if configured) — replication state and configuration

---

## 3. Gathering Information Using the AWS CLI

CLI commands are faster and more automatable than the console for repetitive tasks.

| Purpose | Command |
|---|---|
| List provisioned instances | `aws rds describe-db-instances` |
| List PostgreSQL engine versions | `aws rds describe-db-engine-versions --engine postgres` |
| List DB parameter groups | `aws rds describe-db-parameter-groups` |
| List parameters in a group | `aws rds describe-db-parameters --db-parameter-group-name default.postgres14` |
| List DB security groups | `aws rds describe-db-security-groups` |
| Get DB snapshot info | `aws rds describe-db-snapshots --db-snapshot-identifier mydbsnapshot` |
| List automated backups (current/deleted) | `aws rds describe-db-instance-automated-backups --db-instance-identifier database-1` |
| List DB subnet groups | `aws rds describe-db-subnet-groups` |
| List pending maintenance actions | `aws rds describe-pending-maintenance-actions` |

---

## 4. Monitoring Amazon RDS for PostgreSQL

**Before monitoring, build a plan answering:**
- What are your monitoring goals?
- What resources will you monitor?
- How often?
- What tools will you use?
- Who performs the monitoring?
- Who gets notified on failure?

**Monitoring tools**

| Tool | Purpose |
|---|---|
| **RDS instance status** | Health check via console, CLI, or API |
| **RDS recommendations** | Best-practice guidance (outdated engine versions, non-default security, mismatched instance classes) |
| **Amazon CloudWatch** | Real-time metrics sent every minute; supports alarms on thresholds |
| **Performance Insights** | Visualize DB load, filter by waits/SQL/hosts/users; aggregates across multiple DBs on an instance |
| **Enhanced Monitoring** | Real-time OS-level metrics and process info |

**Performance Insights details**
- Turning on/off causes **no downtime, reboot, or failover**
- Dashboard sections: **Counter metrics**, **Database load**, **Top Load Items**
- Top Load Items shows top SQL queries (as digests), wait states, hosts, or users
- Sends 3 metrics to CloudWatch automatically:
  - `DBLoad` — active sessions (avg)
  - `DBLoadCPU` — active sessions waiting on CPU
  - `DBLoadNonCPU` — active sessions waiting on non-CPU resources (I/O, locks, buffers)

**Enhanced Monitoring setup**
1. RDS console → **Databases** → select instance → **Modify**
2. Scroll to **Monitoring** → enable **Enhanced monitoring**
3. Set **Granularity** (1, 5, 10, 15, 30, or 60 seconds)
4. Requires an **IAM role** (e.g., `rds-monitoring-role`) to send OS metrics to CloudWatch Logs
5. Continue → Apply modifications

**Default OS metrics shown:** Free Memory, Active Memory, CPU User, CPU Total, Used Filesystem, Load Avg 1 min

**OS process list views:**
- **OS processes** — kernel/system processes (minimal performance impact)
- **RDS processes** — RDS management agent, diagnostics, AWS support processes
- **RDS child processes** — processes supporting the DB instance (e.g., `postgres`)

---

## 5. General Problem Determination (Methodology)

1. **Identify the problem** — categorize by symptoms (error messages, user reports, error codes)
2. **Collect data** — gather logs/traces; turn on logging/monitoring features if not already active
3. **Analyze the data** — search logs/traces; use CloudWatch Logs for durable PostgreSQL log storage
4. **Review documentation** — check for known issues/solutions
5. **Try known solutions** — apply fixes one at a time and test
6. **If unresolved:**
   - Post to **AWS re:Post** (community Q&A)
   - Engage **AWS IQ** (paid expert help)
   - Open a case via an **AWS Support Plan**

---

## 6. Troubleshooting Replication Lag

**What it is:** Asynchronous replication between primary and read replica can cause the replica to fall behind, returning stale data.

| Step | Actions |
|---|---|
| **Confirm** | Check CloudWatch `ReplicaLag` metric for consistent increase; check PostgreSQL logs for "Streaming replication has stopped/terminated"; check replication state in console |
| **Collect data** | Review CloudWatch graphs on source & replica; enable query logging on source; check replica PostgreSQL logs; publish logs to CloudWatch Logs |
| **Analyze** | Identify when lag started via `ReplicaLag`; check source `WriteLatency`/`WriteIOPS`; compare instance class/storage between source & replica; check for long-running queries/locks; run `SELECT * FROM pg_stat_activity;`; check `wal_keep_segments` |
| **Apply solutions** | Distribute write workload into smaller transactions; scale up replica to match/exceed source; tune long-running queries; terminate blocking PIDs with `SELECT pg_terminate_backend(PID);`; increase `wal_keep_segments`; re-create replica if terminated (30+ days stopped) |
| **Verify** | `ReplicaLag` decreasing/near zero; log shows "Streaming replication has resumed"; replication state = "replicating" |

---

## 7. Troubleshooting Read Replica "Canceling Statement Due to Conflict with Recovery" Errors

**What it is:** Long-running read replica queries conflict with incoming WAL entries and get canceled so WAL can be applied.

| Step | Actions |
|---|---|
| **Confirm** | Look for `ERROR: Canceling statement due to conflict with recovery` in replica's PostgreSQL error log |
| **Collect data** | Enable query logging on source; examine replica PostgreSQL logs (queries being cancelled run there) |
| **Analyze** | Check frequency of cancellations; compare successful run durations |
| **Apply solutions** | Set `hot_standby_feedback` on replica (may cause source table bloat); tune `max_standby_streaming_delay` / `max_standby_archive_delay` (may cause replication lag); set `vacuum_defer_cleanup_age` on source (may cause source bloat) |
| **Verify** | No recent errors in replica log; confirm with users; increase delay parameters further if cancellations persist |

---

## 8. Troubleshooting Autovacuum Skipping Tables / Dead Tuples

**What it is:** Table/index bloat from unremoved dead tuples slows queries — often caused by long-running transactions blocking autovacuum.

| Step | Actions |
|---|---|
| **Confirm** | Rising `n_dead_tup` in `pg_stat_user_tables`; stale `last_autovacuum`/`last_autoanalyze`; high "dead but not yet removable" counts in autovacuum logs |
| **Collect data** | Query `pg_stat_user_tables` for live/dead tuples; set `log_autovacuum_min_duration = 0` to log all autovacuum activity |
| **Analyze** | Find long-running transactions (`pg_stat_activity`, sessions > 5 min); check locks via `pg_locks` |
| **Apply solutions** | Reduce long-running transactions via query tuning; clean up idle-in-transaction sessions promptly; note autovacuum needs a `SHARE UPDATE EXCLUSIVE` lock and will skip/cancel on conflict |
| **Verify** | `n_dead_tup` decreasing; `last_autovacuum`/`last_autoanalyze` updating; lower "dead but not yet removable" values |

**Key queries:**
```sql
-- Check dead tuples and autovacuum status
SELECT relname AS TableName, n_live_tup AS LiveTuples, n_dead_tup AS DeadTuples,
       last_autovacuum AS Autovacuum, last_autoanalyze AS Autoanalyze
FROM pg_stat_user_tables;

-- Find long-running transactions (>5 min)
SELECT now() - query_start AS Running_Since, pid, datname, usename,
       application_name, client_addr, left(query, 60)
FROM pg_stat_activity
WHERE state IN ('active', 'idle in transaction')
  AND (now() - query_start) > interval '5 minutes';

-- Check current locks
SELECT locktype, database, relation, virtualxid, transactionid,
       virtualtransaction, pid, mode, granted
FROM pg_locks;
```

---

## 9. Troubleshooting Increasing MaximumUsedTransactionID Metric

**Background:** PostgreSQL supports up to ~2 billion in-flight unvacuumed transactions. Approaching this limit forces the DB into **read-only mode** requiring an offline standalone vacuum (hours/days of downtime).

| Step | Actions |
|---|---|
| **Confirm** | `SELECT max(age(datfrozenxid)) FROM pg_database;` — rising value above `autovacuum_freeze_max_age` signals a problem |
| **Collect data** | Check active autovacuum workers (`pg_stat_activity` filtered on `VACUUM`); `SHOW autovacuum_max_workers;`; `SHOW maintenance_work_mem;`; `SHOW autovacuum_freeze_max_age;`; find aging databases/tables; check Read/Write IOPS; enable `rds.force_autovacuum_logging_level = log` |
| **Analyze** | Identify long-running autovacuum sessions; compare active workers vs. max; determine how many tables are nearing the freeze age threshold |
| **Apply solutions** | Increase `autovacuum_max_workers` (static param — requires reboot); increase `maintenance_work_mem` (max 1GB per VACUUM); manually run `VACUUM FREEZE` on aging tables |
| **Verify** | `MaximumUsedTransactionID` drops below `autovacuum_freeze_max_age`; re-check worker usage |

**Key queries:**
```sql
-- Current transaction ID age
SELECT max(age(datfrozenxid)) FROM pg_database;

-- Active vacuum operations
SELECT datname, usename, pid, current_timestamp - xact_start AS xact_runtime, query
FROM pg_stat_activity
WHERE upper(query) LIKE '%VACUUM%'
ORDER BY xact_start;

-- Aging databases
SELECT datname, age(datfrozenxid) FROM pg_database ORDER BY 2 DESC LIMIT 20;

-- Aging tables within a database
SELECT c.oid::regclass AS table_name,
       GREATEST(age(c.relfrozenxid), age(t.relfrozenxid)) AS age,
       pg_size_pretty(pg_table_size(c.oid)) AS table_size
FROM pg_class c
LEFT JOIN pg_class t ON c.reltoastrelid = t.oid
WHERE c.relkind = 'r'
ORDER BY 2 DESC LIMIT 20;
```

---

## 10. Troubleshooting Major Version Upgrade Failures

| Step | Actions |
|---|---|
| **Confirm failure** | Instance status returns to "Available" but **Engine Version stays at the pre-upgrade version** |
| **Collect data** | Download `error/pg_upgrade_precheck.log` from console or CLI |
| **Analyze** | Log reveals specific blocker, e.g., existing **logical replication slots** preventing upgrade |
| **Apply solutions** | If replication slots are unused, drop them: <br>`SELECT * FROM pg_replication_slots;` <br>`SELECT pg_drop_replication_slot(slot_name);` — **do not drop slots still in use** |
| **Verify** | Re-run upgrade; if prechecks pass, status returns to "Available" with target Engine Version; open AWS Support case if it keeps failing |

> Logical replication slots are commonly used for AWS DMS migrations or replicating to data lakes/BI tools — confirm purpose before dropping.

---

## 11. Getting Help / AWS Support

**Information to gather before requesting help**
- Clear problem description + error messages
- Date/time of occurrence (or recent instances)
- Expected outcome
- Steps to reproduce
- RDS instance name(s) and AWS Region
- Client info (EC2 instance/container details, Region, on-prem client info)
- Attached PostgreSQL logs from the time of the issue
- Relevant screenshots

**Opening a support case**
1. Console → **Support** (under Customer Enablement, or search bar) → **Support Center**
2. **Create case** → choose **Technical**
3. Select **AWS Service**, **Category**, and **Severity** (severity affects routing/response time)
4. Review suggested **Solutions to common questions**
5. Enter a clear **Subject** and detailed **Description** (include timestamps, logs, environment details)
6. Attach files (screenshots, logs)
7. Fill any service-specific fields (instance ID, timestamps)
8. Review **Solve now** self-service articles, or proceed via **Contact us**
9. Choose contact language/method and additional contacts → **Submit**

---

### Quick Reference Summary
- **Multi-AZ instance** = 1 standby; **Multi-AZ cluster** = 2 readable standbys across 3 AZs
- **Performance Insights** metrics in CloudWatch: `DBLoad`, `DBLoadCPU`, `DBLoadNonCPU`
- **Enhanced Monitoring** = OS-level real-time metrics (granularity: 1–60 sec)
- Replication lag fixes: manage write load, match replica sizing, tune queries, adjust `wal_keep_segments`
- Read replica cancellation fixes: `hot_standby_feedback`, `max_standby_streaming_delay`, `max_standby_archive_delay`, `vacuum_defer_cleanup_age`
- Transaction ID wraparound risk: monitor `MaximumUsedTransactionID` vs. `autovacuum_freeze_max_age`; fix via more workers/memory or manual `VACUUM FREEZE`
- Major upgrade failures often stem from **logical replication slots** — check `pg_upgrade_precheck.log`
- Troubleshooting method: **Identify → Collect → Analyze → Review docs → Try solutions → Escalate (re:Post / AWS IQ / Support case)**