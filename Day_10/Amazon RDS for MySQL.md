# Amazon RDS for MySQL — Course Notes

## 1. Introduction to Amazon RDS for MySQL

**What it is**
Amazon RDS for MySQL is a **fully managed relational database service** that automates setup, operation, and scaling of MySQL databases in AWS. It combines full MySQL capabilities with AWS reliability, high availability, and disaster recovery.

**What it handles automatically**
- Hardware provisioning
- Database setup & software patching
- Automated backups
- High availability & disaster recovery (Multi-AZ deployments)
- Compatible with existing MySQL apps/tools — easy migration path

**Problems it solves**

| Challenge | How RDS Solves It |
|---|---|
| Manual database management | Eliminates manual hardware/software provisioning |
| Operational overhead | Automates patching, backups, replication |
| Resource constraints | Automated scaling for growing workloads |
| Expertise requirements | Fully managed — no need for in-house DBA depth |
| Business focus | Frees teams to focus on apps, not infrastructure |
| Availability/durability | Automated backups, replication, failover |

**Key benefits**
- Efficient management via automated operations
- Enhanced performance via optimized infrastructure
- Effortless scalability
- Increased reliability with built-in redundancy
- Advanced security
- Streamlined compliance support

**Pricing model**
- **Pay-as-you-go** — no upfront cost or long-term commitment
- Cost drivers: **instance type**, **storage** (General Purpose SSD / Provisioned IOPS SSD), **backup storage**
- Data transfer **within the same Region = free**; egress out of AWS = charged
- Extra costs: Multi-AZ deployments, read replicas
- Pricing varies **by Region** — use AWS Pricing Calculator for estimates

---

## 2. Architecture and Use Cases

**Typical architecture**
- App tier (e.g., EC2) sits in a **public subnet**
- RDS for MySQL instance sits in a **private subnet** (not internet-facing)

**AWS service integrations**

| Service | Purpose |
|---|---|
| **CloudWatch** | Monitor DB metrics, set alarms, track performance/health |
| **Lambda** | Event-driven workflows triggered by DB events (e.g., table updates) |
| **AWS DMS** | Migrate databases from on-prem/other clouds with minimal downtime |
| **Amazon S3** | Backup storage integration |

**Core technical concepts**
- **DB instances** — isolated database environments (basic building block)
- **Instance classes** — determine compute/memory (affects performance & cost)
- **Storage types** — General Purpose SSD vs. Provisioned IOPS SSD
- **Multi-AZ deployments** — synchronous standby replica in a different AZ for HA
- **Read replicas** — read-only copies to offload read traffic
- **Security groups** — virtual firewalls controlling network access
- **Automated backups** — enables point-in-time recovery
- **DB snapshots** — user-initiated backups, retained until manually deleted

**Typical use cases**
- Web & mobile applications
- Ecommerce platforms (catalogs, customer data, orders)
- Content management systems (CMS)
- Data warehousing / BI
- Enterprise applications (financial, inventory, customer records)
- SaaS applications (multi-tenant with data isolation)

**Other considerations**
- **Security**: network isolation, encryption at rest & in transit, IAM integration
- **Monitoring/Logging**: Performance Insights, Enhanced Monitoring, CloudTrail auditing
- **Operations**: automated backups, patching, and scaling via built-in tooling

---

## 3. Creating an Amazon RDS for MySQL Database Instance

**Steps via AWS Management Console**
1. Search **RDS** in the Console → select **RDS**
2. Click **Create database**
3. Choose **MySQL** as the engine
4. Select **DB instance class** (performance/cost tradeoff)
5. Configure **Storage** — type (General Purpose SSD / Provisioned IOPS) + allocated capacity
6. Set **DB cluster identifier**, **Master username**, **Master password**
7. Under **Connectivity**, choose **VPC** and **DB subnet group**
8. Review additional configuration options
9. Click **Create database**
10. Connect using the provided **endpoint** and credentials

> ⚠️ **Warning:** Creating an instance may incur charges based on instance class/storage. Delete the instance after testing to avoid ongoing costs.

---

## 4. Connecting to an Amazon RDS for MySQL Instance

Connect using a MySQL client or application with:
- The RDS instance **endpoint**
- The **master username and password**
- Ensuring the **security group / VPC network path** permits the connection

---

## 5. Creating a Table in an Amazon RDS for MySQL Instance

**Workflow**: Connect via MySQL client → create database → select database → create table → verify table.

```sql
-- Create a new database
CREATE DATABASE mydatabase;

-- Select the database to use
USE mydatabase;

-- Create a table named "users"
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL
);

-- List tables in the database
SHOW TABLES;
```

---

### Quick Reference Summary
- RDS for MySQL = **fully managed**, pay-as-you-go relational database
- **Multi-AZ** = high availability (standby replica, different AZ)
- **Read replicas** = scale read performance, not for HA by themselves
- Best practice: place RDS instance in a **private subnet**
- Core integrations: **CloudWatch** (monitoring), **Lambda** (event-driven), **DMS** (migration), **S3** (backups)
- Table creation flow: `CREATE DATABASE` → `USE` → `CREATE TABLE` → `SHOW TABLES`