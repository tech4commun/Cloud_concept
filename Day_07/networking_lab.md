# AWS Networking Basics — Notes

## AWS Global Infrastructure

AWS offers **27 networking services** for connectivity, security, and monitoring — all built on the AWS Global Infrastructure: a collection of smaller infrastructure groupings connected by a global high-speed network, designed for **resiliency and high availability** via geographically dispersed, redundant hardware.

### Infrastructure Building Blocks

| Component | What | Why |
|---|---|---|
| **Region** | Geographical area with **2+ Availability Zones** | Spanning multiple AZs lets you survive large-scale disasters; Regions are isolated from each other → fault tolerant |
| **Availability Zone (AZ)** | One or more interconnected data centers with redundant power/networking | Isolated from other AZs but linked via high-speed redundant networking; if one AZ fails, others keep running |
| **Local Zone** | Extension of a Region, places compute/storage/DB closer to large population centers | Supports latency-sensitive applications |
| **Edge Location** | Global CDN endpoint (via **Amazon CloudFront**) | Caches content close to users — faster delivery, no round-trip to origin DB. More edge locations exist than Regions. |

---

## Service Resilience Tiers

| Tier | How It Works | Failure Behavior |
|---|---|---|
| **Globally-resilient** | Single database, replicated across Regions | Survives a full Region failure |
| **Regionally-resilient** | One data set per Region, replicated across AZs in that Region | Survives an AZ failure; **fails if the whole Region fails** |
| **Zone-resilient** | Runs in a single AZ | Fails if that AZ fails |

### As Your Network Grows — Stay On Top Of:
- New services/tools to optimize design
- Security for current + new requirements (inspect, filter, detect threats)
- Monitoring to ensure scaling + operational excellence

---

## Amazon VPC — Foundational Networking Service

**Definition:** A software-defined virtual private network — a service for creating secure private networks in AWS to host apps/data. Can connect to on-premises → hybrid environment.

- A VPC is **regional** — lives in one Region, one AWS account.
- Each **subnet's IP range must be unique** and non-overlapping with other subnets.
- If an AZ fails, only the subnet(s) in that AZ fail — other subnets in other AZs keep running.

### Default vs. Non-default VPC
| | Default VPC | Non-default VPC |
|---|---|---|
| Created by | AWS automatically, at account creation | You, manually |
| Count per Region | Only **one** | Multiple allowed |
| Access | **Public by default** (the one exception) | **Private/isolated** until you explicitly grant public access |
| Config | Same standard config everywhere | Custom |

### Default VPC Details
- Comes with **one CIDR range** (Classless Inter-Domain Range) — defines the IP range everything inside the VPC uses.
- Configured with a **Class B subnet**.
- One subnet is auto-created **per AZ** in the Region.
- Default route table:

| Destination | Target |
|---|---|
| `172.31.0.0/16` | local |
| `0.0.0.0/0` | internet_gateway_id |

---

## Amazon VPC Components

### CIDR
Defines the VPC's overall IP address range — everything inside the VPC uses it; all inbound/outbound VPC traffic maps to this range.

### Subnets
- A **sub-network** of the VPC's CIDR range, created in **one AZ**.
- AZ-resilient feature: if that AZ fails, the subnet (and its services) fail too.
- **Best practice:** spread services across subnets in different AZs for high availability.
- Subnets within a VPC communicate via the **local route**.

### Amazon EC2
Compute instances that run inside subnets — the actual workloads the VPC hosts.

### Routing (VPC Router)
- Highly available, runs across **all AZs** the VPC uses.
- Has a network interface in every subnet, using the **Network +1** address.
- Fully managed by AWS — you don't configure the router itself, only its **route tables**.

### Route Tables
- Created at the **VPC level**, associated with subnets.
- Every VPC has a **main route table** — used automatically if you don't explicitly associate another one.
- A subnet can have **only one** route table at a time, but one route table can serve **many** subnets.

### Internet Gateway
Entry/exit point connecting the VPC to the public internet.

### Network ACL
Subnet-level, stateless firewall — control list for inbound/outbound subnet traffic.

### Security Groups
Instance-level, stateful firewall — controls traffic to/from individual EC2 instances.

> 💰 No charge for the VPC itself — you pay for optional usage-based capabilities (NAT gateways, data transfer, etc.)

---

## Network Gateways

**Definition:** A device/node connecting networks using different transmission protocols, performing protocol translation. Acts as the entry/exit point — all data must pass through the gateway to be routed. Can be hardware or software-defined.

AWS offers **7 networking gateways** total (each suited to different connectivity needs — Internet Gateway, NAT Gateway, Virtual Private Gateway, Transit Gateway, etc.)

> **Gateway vs. Router:** a router directs traffic between networks using the *same* protocol; a gateway translates *between* different protocols/network types.

---

## VPC Peering

- Links multiple VPCs together — **direct communication** between two isolated VPCs using **private IP addresses**.
- Can span **AWS accounts** and **Regions**.
- Data is **encrypted** across the AWS global infrastructure.

---

## AWS Transit Gateway

Solves the complexity of peering at scale — instead of maintaining individual route tables/peering connections between every VPC (and separate gateways for on-prem), Transit Gateway acts as a **central hub** connecting multiple VPCs and on-premises networks.

---

## AWS PrivateLink

- Provides **private connectivity** between VPCs, AWS services, and on-prem networks — **without traversing the public internet**.
- Keeps data off the internet → reduces exposure/compromise risk.
- Simplifies connecting services **across different accounts and VPCs**.
- Lets you use more AWS services confidently without trading off security.

---

## Core Networking Protocols (Reference)

| Protocol | Purpose |
|---|---|
| **IP (IPv4 / IPv6)** | How devices communicate in AWS and over the internet |
| **SSH** | Cryptographic protocol for securely operating network services over an unsecured network |
| **TCP** | Establishes/maintains a network conversation for reliable data exchange; works with IP |
| **UDP** | Low-latency, loss-tolerant protocol — sends data without waiting for receiver acknowledgment, trading reliability for speed |

---

## Quick Reference

- **Region** = 2+ AZs | **AZ** = isolated data center(s) | **Local Zone** = latency-optimized extension | **Edge Location** = CDN cache point (CloudFront).
- **VPC = regional service**; only **one default VPC** per Region, and it's the only VPC type that's public by default.
- **Route table**: 1 per subnet at a time, but reusable across many subnets; falls back to the **main route table** if none is explicitly associated.
- **NACL** = stateless, subnet-level | **Security Group** = stateful, instance-level.
- **VPC Peering** = direct, private, 1:1 VPC-to-VPC link. **Transit Gateway** = hub-and-spoke for many VPCs/on-prem at scale. **PrivateLink** = private service access without public internet exposure.