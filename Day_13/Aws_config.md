# AWS Config — Getting Started Course Notes

## 1. Introduction to AWS Config

**What it is**
AWS Config provides a detailed view of the **configuration of your AWS resources**. It continuously **monitors and records** resource configurations, helping you audit configuration history and assess compliance with organizational policies.

- Captures **resource relationships** and configuration details
- Enables discovering existing resources, exporting a complete inventory, and determining how a resource was configured **at any point in time**
- Streamlines compliance auditing, security analysis, change management, and operational troubleshooting
- Evaluates resource configurations against **rules** (best practices/internal policies) and assesses the impact of configuration changes

### Core functionality

| Function | Description |
|---|---|
| **Configuration recording** | Continuously and automatically monitors/records configuration changes — attributes, relationships, current settings — building a full configuration history |
| **Configuration evaluation** | Compares recorded configurations against rules (AWS managed or custom) whenever a resource is created, modified, or deleted; flags non-compliant resources |
| **Configuration history and snapshots** | Maintains historical, point-in-time records per resource; generate full-environment **snapshots** for troubleshooting, audits, and understanding infrastructure evolution |

### Technical concepts

| Concept | Description |
|---|---|
| **Configuration items** | Fundamental building block — a point-in-time record of a resource's attributes, relationships, and configuration. A new item is created every time a tracked resource changes; stored to build a complete change history |
| **Configuration streams** | Near real-time notifications of resource changes, delivered via an **Amazon SNS** topic — can trigger automated responses or external system integrations |
| **Configuration rules** | Define desired configuration settings; AWS Config evaluates resources against them (on change or on a schedule) and reports compliant/non-compliant status |
| **Configuration recorder** | The component that actually captures configuration changes; must be **set up and started** before AWS Config can track resources; customizable scope (which resource types) and storage destination |
| **Configuration history** | A full chronological timeline of changes per resource (what changed, when, relationships affected) — accessible via console, API, or delivered as files to an **S3 bucket** |
| **Conformance packs** | Collections of AWS Config rules + remediation actions, deployable together as a single unit across multiple accounts/Regions — typically represent a compliance framework or internal policy set. AWS provides sample packs for common standards |

### Key features and capabilities
**Features:**
- Resource discovery and inventory
- Configuration snapshots and history
- Compliance monitoring and reporting

**Capabilities:**
- Multi-account, multi-Region aggregation
- Automated remediation
- Resource relationship tracking

### Practical business applications

| Application | Example |
|---|---|
| **Compliance management and reporting** | A financial services company verifies all S3 buckets with customer data are encrypted and not publicly accessible — automated compliance evidence reduces manual audit prep |
| **Security incident investigation** | Security teams review configuration history to see when security group rules changed, IAM permissions expanded, or encryption was disabled — helps pinpoint root cause of a breach |
| **Change management and tracking** | Ops teams review Config history during a service disruption to identify recent changes that may have caused the issue — reduces MTTR |
| **Resource optimization and cost management** | Finance/cloud ops teams find unattached EBS volumes, idle load balancers, or oversized/non-standard instances to cut costs |

---

## 2. Technical Overview — Architecture & Integrations

### Architecture
AWS Config's architecture centers on **capturing, storing, and evaluating** resource configurations through several interconnected components (recorder → configuration items → rules evaluation → history/snapshots → notifications), providing end-to-end visibility and governance.

### Key service integrations

| Service | Integration purpose |
|---|---|
| **Amazon S3** | Stores configuration history files, snapshots, and evaluation results — durable long-term storage; accessible directly for custom analysis; supports S3 lifecycle policies for retention management |
| **Amazon SNS** | Delivers notifications on configuration changes and compliance evaluations; subscribe endpoints like email, Lambda, or SQS to build automated workflows |
| **AWS CloudTrail** | Provides the "**who**" behind changes (API caller) to complement Config's record of "**what**" changed — together giving full accountability and traceability |
| **AWS Organizations** | Deploy configuration rules and conformance packs centrally across **multiple accounts** — define once, apply org-wide |
| **AWS Lambda** | Powers **custom rules** — you provide a Lambda function containing evaluation logic; Config invokes it on a schedule or on resource change |
| **AWS Systems Manager** | Automates **remediation** of non-compliant resources via Automation documents (e.g., auto-correcting an overly permissive security group) |
| **Amazon EventBridge** | Detects Config events and routes them to targets based on rules — enables event-driven workflows like ticket creation, alerts, or remediation scripts |

### Integration considerations
- **Security** — use IAM roles/policies to control service access; encrypt sensitive data; follow authentication best practices; regularly audit access patterns/permissions; consider VPC endpoints/private links to reduce public network exposure
- **Scalability** — plan for growth across accounts/Regions (see multi-account aggregation)
- **Monitoring** — track Config's own operation and the health of its integrations

---

## 3. AWS Config Demonstrations (Console Walkthrough)

> ⚠️ These demonstrations can incur AWS costs. Clean up all created resources afterward if performed in your own environment.

### Demo 1 — Setting Up AWS Config
1. Search **config** in the Console → open **AWS Config**
2. Choose **Get started**
3. On the **Settings** page:
   - **Recording strategy:** All resource types with customizable overrides
   - **Recording frequency:** Continuous recording
   - **Data governance → IAM role:** Create AWS Config service-linked role
   - **Delivery channel → S3 bucket:** Create a bucket → enter a unique bucket name
4. Choose **Next**
5. On the **Rules** page → **Next** (no rules added yet)
6. On the **Review** page → review settings → **Confirm**
7. The AWS Config dashboard appears once setup completes

### Demo 2 — Configuring AWS Config Rules
1. Open **AWS Config** → navigation menu → **Rules**
2. Choose **Add rule**
3. **Select rule type:** Add AWS managed rule
4. Search **S3** → select rule: **`s3-bucket-versioning-enabled`** → **Next**
5. **Trigger type:** evaluates on configuration changes (default)
6. **Scope of changes:** keep default → **Next**
7. Review rule details → **Save**
8. Confirmation banner appears — rule successfully added

### Demo 3 — Exploring the Dashboard and Findings
1. Open **AWS Config** → **Dashboard** → review **Compliance status** section
2. Navigation menu → **Resources**
3. On **Resource Inventory**, search resource type: `s3` → select **AWS S3 Bucket** → **Apply**
4. Under **Resource identifier**, choose one of your S3 buckets
5. On the bucket details page → **Resource Timeline**
6. Under **Events**, note the resource is **Noncompliant** (bucket versioning not enabled)

**Fix the compliance issue:**
7. Search **s3** → open **Amazon S3** console
8. Select the bucket created during Config setup → **Properties** tab
9. **Bucket Versioning** section → **Edit** → **Enable** → **Save changes**
10. Confirmation banner: bucket versioning enabled

**Verify:**
11. Return to **AWS Config** console → **Dashboard** → **Compliance status** now shows the resource as **Compliant**

### Demo 4 — Deleting AWS Config Resources (Cleanup)
1. Open **AWS Config** → **Rules**
2. Select `s3-bucket-versioning-enabled` → **Actions** → **Delete rule**
3. In the confirmation dialog, type `confirm` → **Delete** (deletion can take several minutes)
4. Once deleted, the rule no longer appears on the Rules page

**Stop the recorder:**
5. Navigation menu → **Settings**
6. **Recorder** section → **Stop recording** → confirm in the dialog
7. Recorder status now shows **off**

**Delete the S3 bucket:**
8. Search **s3** → open **Amazon S3** console
9. Select the bucket → **Empty**
10. Type `permanently delete` → **Empty** → confirm banner → **Exit**
11. Select the bucket again → **Delete**
12. Type the bucket name to confirm → **Delete bucket**
13. Confirmation banner: bucket successfully deleted

---

### Quick Reference Summary
- **Configuration item** = point-in-time record of a resource's state; created on every tracked change
- **Configuration recorder** must be set up and started before Config tracks anything
- **Configuration rules** = compliance checks; AWS managed or custom (via Lambda)
- **Conformance packs** = bundled rules + remediation, deployable across accounts/Regions
- Config records **what** changed; **CloudTrail** records **who** made the change — used together for full accountability
- Key integrations: **S3** (storage), **SNS** (notifications), **CloudTrail** (attribution), **Organizations** (multi-account), **Lambda** (custom rules), **Systems Manager** (auto-remediation), **EventBridge** (event-driven workflows)
- Typical setup flow: enable recorder + S3 bucket → add managed/custom rules → monitor dashboard for compliance → remediate non-compliant resources → clean up (delete rules → stop recorder → empty & delete S3 bucket)