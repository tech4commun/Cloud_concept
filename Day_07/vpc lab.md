# Introduction to Amazon VPC — Lab Guide

**Duration:** ~45 minutes
**Goal:** Create a VPC with the wizard, then explore its core components.

## Overview

Amazon VPC lets you provision a logically isolated section of AWS where you launch resources in a virtual network you define — your own IP range, subnets, route tables, and gateways. Supports both IPv4 and IPv6.

### Objectives
- Create a VPC using the VPC Wizard
- Explore VPC components:
  - Public and private subnets
  - Route tables and routes
  - NAT gateways
  - Network ACLs

---

## Task 1: Create an Amazon VPC

**Architecture you're building:** One VPC → one public subnet + one private subnet → Internet Gateway attached to the VPC → NAT Gateway launched in the public subnet.

1. Go to **VPC** in the AWS Console → **Create VPC**.
2. Under **VPC settings**, choose **VPC and more**.
3. **Name tag auto-generation:** Auto-generate → enter `Lab`
4. **Number of Availability Zones (AZs):** `1`
5. **Number of public subnets:** `1`
6. **Number of private subnets:** `1`
7. Expand **Customize subnets CIDR blocks:**
   - Public subnet CIDR: `10.0.25.0/24`
   - Private subnet CIDR: `10.0.50.0/24`
8. **NAT gateways ($):**
   - Select **Zonal** option
   - Then choose **In 1 AZ**
9. **VPC endpoints:** `None` (make sure S3 Gateway is **not** selected)
10. Click **Create VPC** → wait for confirmation (takes a few minutes).
11. Click **View VPC**.
   > 📋 Copy the **VPC ID** to a text editor — you'll need it later.

✅ **Done:** VPC created via the wizard.

---

## Task 2: Explore Your VPC

### VPC
- **VPC → Your VPCs** → find it named `lab-vpc`.

### Internet Gateway
- **VPC → Internet gateways** → shows the gateway attached to your VPC.
- Connects the VPC to the internet. Horizontally scaled, redundant, highly available — no bandwidth constraint.

### Public Subnet
- **VPC → Subnets** → select `Lab-subnet-public...`
- A subnet:
  - Belongs to one VPC
  - Lives in a single AZ
  - Has its own CIDR range
- CIDR `10.0.25.0/24` → IPs `10.0.25.0`–`10.0.25.255`
- Shows **250 available IPs** out of 256 (some reserved + 1 used by the NAT gateway).

**Route table tab:**
| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | local | Keeps traffic inside the VPC |
| `0.0.0.0/0` | Internet Gateway | Sends all other traffic to the internet |

> Routes are evaluated most-specific → least-specific (`0.0.0.0/0` last). Having a route to the **Internet Gateway** is what makes this subnet **public**.

**Network ACL tab:**
- Stateless firewall at the subnet level.
- Default rules: **Rule 100** allows all inbound and all outbound traffic.
- Catch-all rule (`*`) denies anything not explicitly matched — extra safety net.

### Private Subnet
- **VPC → Subnets** → select `Lab-subnet-private...`
- **Tags tab:** tagged `Name = Lab-subnet-private...`

**Route table tab:**
| Destination | Target | Purpose |
|---|---|---|
| `10.0.0.0/16` | local | Same as public subnet — internal VPC traffic |
| `0.0.0.0/0` | NAT gateway | Routes internet-bound traffic through the NAT gateway instead |

> No direct route to the Internet Gateway = this is what makes it **private**. Instances here can't be reached directly from the internet.

### NAT Gateway
- **VPC → NAT gateways** → shows the gateway in the public subnet.
- Lets private-subnet resources **initiate outbound** connections to the internet/other AWS services.
- **Outbound-only** — the internet cannot initiate a connection back in.
- Flow: private subnet → NAT gateway → Internet Gateway → internet.

### Security Groups
- **Security → Security groups** → find the one matching your VPC ID → **Inbound rules** tab.
- Security groups = **instance-level** virtual firewall (not subnet-level).
- Up to **5** security groups per EC2 instance.
- **Default security group** behavior: allows ALL traffic — but only when the source is *also* the default security group (self-referencing). Everything else is denied by default.
- You can create additional security groups for specific tiers (web, app, database).

> ⚠️ This lab does not include launching EC2 instances — don't attempt to launch one.

✅ **Done:** Explored all core VPC components created by the wizard.

---

## Summary

| Component | Role |
|---|---|
| VPC | Isolated virtual network, your own IP range |
| Internet Gateway | Connects VPC to the internet |
| Public subnet | Has a route to the Internet Gateway → internet-reachable |
| Private subnet | No direct route to Internet Gateway → not internet-reachable |
| Route table | Defines where outbound traffic goes, per subnet |
| NAT gateway | Lets private subnet resources reach the internet outbound-only |
| Network ACL | Stateless, subnet-level firewall |
| Security group | Stateful, instance-level firewall |

## Additional Resources
- [Amazon VPC](http://aws.amazon.com/vpc/)
- [AWS Training & Certification](http://aws.amazon.com/training/)



































































































