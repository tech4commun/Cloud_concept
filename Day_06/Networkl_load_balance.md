# Network Load Balancer — Getting Started Notes

## What It Is

Network Load Balancer (NLB) is an **Elastic Load Balancing** type that distributes incoming traffic across targets (EC2 instances, containers, IP addresses). Operates at **Layer 4 (transport layer)** of the OSI model.

- Handles **millions of requests/second** at **ultra-low latency**.
- Routes based on: **IP protocol data, TCP/UDP port, client IP address** — not request content.
- Supports **TCP and UDP simultaneously**.
- Provides **static IP addresses** for the load balancer endpoint.
- Preserves **source IP address** — important for traffic patterns needing client identity.
- Offers **direct connectivity to targets** using private IP addresses.

> Best fit: non-HTTP/HTTPS protocols, or apps needing consistent/static network addressing.

---

## Core Functionality

| Function | Description |
|---|---|
| **Connections** | Operates at connection level — routes based on IP data, not request content → faster processing |
| **Health checks** | Confirms targets are responsive before/while sending traffic |
| **Traffic distribution** | Spreads millions of req/s across targets with ultra-low latency |

---

## Technical Concepts

| Concept | What It Means |
|---|---|
| **Listeners and ports** | Define which protocol/port NLB listens on for incoming traffic |
| **Target groups** | Logical grouping of targets (instances/IPs/containers) that receive routed traffic |
| **Connection draining** | Gracefully completes in-flight requests before removing a target |
| **Cross-zone load balancing** | Distributes traffic evenly across targets in **all enabled AZs**, not just the AZ traffic entered through |
| **Client IP preservation** | Original client IP is passed through to the target — no NAT masking |
| **Flow stickiness** | Routes connections from a specific client to the **same target** consistently |
| **Zonal isolation** | Keeps traffic within a single AZ when needed, for fault isolation |

---

## Key Features & Capabilities

- **High performance** — millions of req/s, ultra-low latency
- **Protocol support** — TCP, UDP, and TLS
- **Static IP addresses** — fixed IP per AZ for the load balancer's lifetime (helpful for firewall allowlisting)

---

## Practical Business Applications

| Use Case | Why NLB Fits |
|---|---|
| **High-performance web apps** (finance, ecommerce, streaming) | Handles millions of req/s with ultra-low latency; absorbs traffic spikes (flash sales, breaking news) without pre-warming |
| **Gaming infrastructure** | UDP support for real-time gameplay; client IP preservation aids matchmaking/anti-cheat; static IPs simplify firewall allowlisting |
| **IoT device management** | Scales to massive connection pools (TCP + UDP); supports smart home, industrial monitoring, connected vehicles |
| **Voice & video communications (VoIP, video conferencing)** | Single NLB handles TCP (signaling) + UDP (media) together; low latency + IP preservation aid call quality monitoring |

---

## Architecture

- Distributed system of **load balancer nodes** deployed across **multiple AZs** in a Region.
- Traffic enters via nodes → routed to registered targets based on **routing algorithm + health check status**.
- Each node/component contributes to high availability and performance.

---

## Integrations

| AWS Service | Role |
|---|---|
| **Amazon EC2** | Standard compute targets |
| **Amazon ECS / Amazon EKS** | Container-based targets |
| **AWS Global Accelerator** | Improves global application performance (routes over AWS backbone) |
| **AWS Certificate Manager** | Manages TLS certificates for NLB TLS listeners |
| **Amazon CloudWatch** | Metrics/monitoring for NLB and target health |

### Integration Considerations
- **Traffic routing strategies**: design routing algorithms to match app/traffic patterns; use **target group weights** for blue/green deployments & canary releases; use **flow stickiness** (based on source/destination IP + port) for connection persistence.
- **Health check configurations**: tune to target application behavior.
- **Security controls**: apply appropriate access/security policies at each integration point.

---

## Demonstrations (Hands-On Workflow)

### Prerequisites
- Requires infra set up via **AWS CloudFormation** (YAML template provided: `nlb-demo-prereqs-cf-template.yaml`, ~4.6 KB).
- Template provisions:
  - VPC with **2 public subnets** across different AZs
  - Internet gateway + routing
  - Security group allowing **HTTP (80)** and **SSH (22)**
  - **2× t3.micro EC2 instances** with Apache installed
  - Stack outputs: VPC ID, subnet IDs, instance IPs

### Demo Steps
1. **Create a Network Load Balancer** — set up via AWS Console
2. **Configure health checks** — define check parameters for targets
3. **Test your Network Load Balancer** — validate routing/connectivity
4. **Delete NLB resources** — clean up via Console to avoid ongoing charges

> ⚠️ Cost warning: demos can incur AWS charges — clean up all resources (NLB, target groups, EC2 instances, CloudFormation stack) after practicing.

---

## Quick Reference

- **NLB = Layer 4** (vs. ALB = Layer 7, GWLB = Layer 3).
- Routes by **IP protocol/port/client IP**, not request content → very fast.
- Supports **TCP + UDP simultaneously**, plus TLS.
- **Static IP per AZ** — useful for firewall allowlisting.
- **Client IP preservation** — no NAT masking of source IP.
- **Flow stickiness** = same client → same target, based on IP/port hashing.
- Best for: extreme-performance apps, gaming, IoT, VoIP/video — anything needing raw throughput or non-HTTP protocols.