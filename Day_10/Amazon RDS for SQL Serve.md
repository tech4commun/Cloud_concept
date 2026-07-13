# Amazon RDS for SQL Server — Course Notes

## 1. Introduction to Amazon RDS for SQL Server

**What it is**
Amazon RDS for SQL Server is a **fully managed database service** for Microsoft SQL Server. It streamlines setup, operation, and scaling of SQL Server databases in the cloud, supporting several SQL Server **versions and editions**.

**What AWS handles automatically**
- Hardware provisioning
- Database setup & patching
- Backups
- Infrastructure management/maintenance

**Problems it solves**

| Challenge | How RDS Solves It |
|---|---|
| Infrastructure challenges | Automates provisioning — launch instances in minutes |
| Operational tasks | Automates patching, backups, monitoring; simplifies HA setup |
| Scalability | Easy compute/storage scaling via console |
| Cost optimization | Pay-as-you-go pricing, includes licensing — no big upfront cost |
| Compliance and security | Built-in compliance framework, security tools |
| AWS service integration | Seamless integration reduces technical overhead |

**Key benefits**
- **Automated management** — provisioning, setup, backups handled for you
- **High availability** — Multi-AZ deployments replicate to standby instance, automated failover
- **Flexible scaling** — scale compute and storage independently
- **Robust data protection** — VPC network isolation, KMS encryption at rest, TDE (Enterprise/Standard editions), SSL in transit, SQL Server Authentication
- **Advanced monitoring** — CloudWatch, Enhanced Monitoring (OS metrics), Performance Insights (query analysis)
- **Customizable environment** — RDS Custom for SQL Server allows deep OS/engine customization
- **Automated backup & recovery** — daily backups, configurable retention, point-in-time recovery

**Pricing model**
- **Pay-as-you-go** — no upfront costs or long-term commitments
- Cost factors: **instance type**, **SQL Server edition**, **AWS Region**
- Based on CPU, memory, storage, and network capacity
- Instance sizes range from small dev environments to large production workloads
- Use **AWS Pricing Calculator** for detailed estimates

---

## 2. Architecture and Use Cases

**AWS service integrations**

| Service | Purpose |
|---|---|
| **Amazon VPC** | Secure, isolated network environment for RDS instances |
| **IAM** | Fine-grained access control for RDS resources |
| **Amazon S3** | Backup storage, data import/export, long-term archival |
| **AWS Lambda** | Automate DB management tasks (e.g., scheduled backups) |
| **CloudWatch** | Monitor performance, set alarms, collect logs |
| **AWS DMS** | Migrate on-prem/cloud SQL Server DBs with minimal downtime |
| **Amazon Directory Service** | Windows Authentication via AD (managed or on-prem) |

**Core technical concepts**
- **Database instance** — isolated SQL Server environment; the basic building block
- **Storage options** — General Purpose SSD, Provisioned IOPS SSD
- **Backup and restore** — automated backups enable point-in-time recovery (PITR); manual snapshots also supported
- **High availability & scalability**
  - Multi-AZ deployments — synchronous replication across AZs
  - Vertical scaling — modify instance type/storage
  - Choice of Single-AZ or Multi-AZ
- **Security & access controls** — VPC network isolation, encryption at rest/in transit, IAM integration
- **AWS service integration** — Lambda (event-driven), CloudTrail (auditing), CloudWatch (monitoring)
- **Subnet Group** — private subnets across AZs where RDS instances deploy for HA/fault tolerance
- **Parameter Group** — container for DB engine configuration values
- **Option Group** — collection of enabled engine features
- **Multi-AZ deployment** — primary + standby instance across AZs; uses **SQL Server Database Mirroring (DBM)** or **Always On Availability Groups (AGs)**; RDS auto-repairs unhealthy instances and manages failover
- **Read replica** — read-only copy; updates from primary replicated **asynchronously**
- **Encryption**
  - At rest: AWS KMS (storage) + SQL Server TDE (database)
  - In transit: SSL

**Typical use cases**
- **Web applications** — dynamic websites, user data, transactional data
- **Business intelligence** — data warehousing, analytics, reporting
- **Enterprise applications** — ERP, CRM, HR systems (mission-critical)
- **Ecommerce platforms** — product catalogs, inventory, customer data, orders

**Other considerations**
- **Security & access management** — use IAM for users/groups/roles with specific permissions; configure network ACLs and security groups for traffic control
- **Monitoring and logging**
- **Backup and disaster recovery**
- **Performance optimization**

---

## 3. Creating an Amazon RDS for SQL Server Database Instance

**Steps via AWS Management Console**
1. Navigate to the **Amazon RDS** service in the Console
2. Choose **Create database**
3. Select **Microsoft SQL Server** as the database engine
4. Select the **SQL Server edition** (Express, Web, Standard, or Enterprise)
5. Configure instance credentials — set **username** and **password**
6. Choose an **instance type** (e.g., `db.t3.micro` for testing) and **storage type** (e.g., General Purpose SSD)
7. Set **Public access** to **Yes**
8. Configure network settings — select **VPC** and **subnet**
9. Review settings → choose **Create database**
10. Once created, connect using your preferred **SQL client tool**

---

## 4. Connecting to an Amazon RDS for SQL Server Database Instance

**Prerequisite — install the `sqlcmd` client (Windows):**
```bash
winget install sqlcmd
```
> You may need to update your **PATH** environment variable so `sqlcmd` is recognized.

**Steps**
1. In the RDS console, select your database instance
2. Under **Connectivity & security**, note the port — SQL Server uses **port 1433**
3. Create an **inbound rule** in the instance's security group to open port 1433
4. Connect using `sqlcmd` and verify with a simple `SELECT` command

**Connection command:**
```bash
sqlcmd -S [endpoint] -U [username] -P [password]
```
Replace:
- `[endpoint]` — the RDS instance endpoint
- `[username]` / `[password]` — your database credentials

---

## 5. Creating a Database and Table in Amazon RDS for SQL Server

**Workflow:** Open SQL client → create database → use database → create table → insert data → verify with SELECT.

```sql
-- Create a new database
CREATE DATABASE mydatabase;
GO

-- Switch to the new database
USE mydatabase;
GO

-- Create the Employees table
CREATE TABLE Employees (
    EmployeeID INT PRIMARY KEY,
    FirstName NVARCHAR(50),
    LastName NVARCHAR(50),
    BirthDate DATE,
    HireDate DATE,
    Position NVARCHAR(100),
    Salary DECIMAL(10,2),
    Email NVARCHAR(100),
    Phone NVARCHAR(15)
);
GO

-- Insert sample data
INSERT INTO Employees (EmployeeID, FirstName, LastName, BirthDate, HireDate, Position, Salary, Email, Phone)
VALUES
(1, 'John', 'Doe', '1990-03-15', '2020-06-01', 'Software Engineer', 60000.00, 'john.doe@example.com', '0123456789'),
(2, 'Jane', 'Smith', '1988-07-20', '2018-09-15', 'Project Manager', 75000.50, 'jane.smith@example.com', '0987654321'),
(3, 'Alice', 'Johnson', '1995-01-10', '2022-01-05', 'Data Analyst', 50000.75, 'alice.johnson@example.com', '0551234567');
GO

-- Verify the data
SELECT * FROM Employees;
GO
```

---

### Quick Reference Summary
- RDS for SQL Server = fully managed, pay-as-you-go, supports multiple **editions** (Express/Web/Standard/Enterprise)
- **Multi-AZ** uses **Database Mirroring (DBM)** or **Always On Availability Groups (AGs)** — not simple replication
- **Read replicas** = asynchronous, read-only, for scaling reads (not HA)
- Default SQL Server port: **1433** — must open in security group inbound rules
- Encryption: **KMS** (storage) + **TDE** (database) at rest, **SSL** in transit
- T-SQL basics: `CREATE DATABASE` → `USE` → `CREATE TABLE` → `INSERT INTO` → `SELECT` (each followed by `GO`)
- Common use case: **ERP systems** — need reliability, HA, consistent performance, backups, and security