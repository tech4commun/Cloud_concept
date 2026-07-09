# Gateway Load Balancer — Getting Started Notes

## What It Is

Gateway Load Balancer (GWLB) deploys, scales, and manages **virtual security appliances** (firewalls, IDS, deep packet inspection). It acts as a **transparent network gateway** — intercepts traffic, routes it to third-party security appliances, then continues traffic to its destination.

### Key Positioning
- **ALB** → Layer 7 (application)
- **NLB** → Layer 4 (transport)
- **GWLB** → **Layer 3 (network)** — handles **all IP packets**, regardless of protocol (TCP, UDP, ICMP, etc.)

> Layer 3 operation = works with **any** protocol, ideal for appliances that must inspect *everything*, not just HTTP/TCP.

### Core Value
- Combines **transparent gateway** + **load balancing** → distributes traffic across a fleet of security appliances.
- Preserves **original source/destination IPs** — critical for accurate security analysis.
- Delivers high availability + scalability for security infrastructure.

---

## Core Functionality
1. **Traffic interception and routing** — intercepts, inspects context, routes to appliances
2. **Load balancing security appliances** — spreads traffic across multiple appliance instances
3. **High availability and scalability** — maintains uptime as appliance fleet/traffic scale

---

## Technical Concepts

| Concept | What It Does |
|---|---|
| **GENEVE protocol** | Encapsulation protocol — wraps original packet in a header, without altering contents, so appliances see real src/dst IPs (not the LB's IP) |
| **Target groups** | Collection of security appliances receiving traffic; also define health check rules; unhealthy targets are automatically routed around |
| **VPC endpoint services** | Expose GWLB security services to other VPCs/accounts privately — enables "security-as-a-service"; traffic never leaves the AWS network |
| **Elastic Network Interfaces (ENIs)** | Entry/exit points for traffic in the VPC; separate ingress/egress flows for cleaner routing + troubleshooting |
| **Flow symmetry** | Ensures both directions of a connection (request + response) hit the **same** appliance — vital for stateful appliances (e.g., firewalls tracking connection state); achieved via consistent hashing |
| **Health checks** | Periodic probes to appliances; unhealthy targets are marked and traffic is rerouted automatically |

---

## Key Features & Capabilities

### Features
- **Transparent network gateway** — intercepts/routes without modifying packet data
- **Layer 3 operation** — inspects all IP-based protocols (TCP/UDP/ICMP/etc.), not just app-layer traffic
- **Cross-zone load balancing** — any GWLB node can send traffic to appliances in **any AZ**, balancing load and adding zone-failure redundancy

### Capabilities
- **Automatic scaling** — adjusts capacity to traffic patterns transparently, no manual provisioning for peak load
- **Health monitoring** — continuous customizable checks, automatic failover away from unhealthy appliances
- **Private connectivity** — via VPC endpoints, keeps all inspection traffic off the public internet; enables cross-account security-service sharing

---

## Practical Business Applications

| Use Case | Value |
|---|---|
| **Enhanced security posture** | Makes inline appliance deployment practical at scale → supports defense-in-depth |
| **Centralized security services** | One security team/config protects many app teams ("security-as-a-service") → standardized controls, fewer config errors |
| **Regulatory compliance** | Eases traffic-inspection/monitoring mandates without breaking app connectivity |
| **Cost optimization** | Balanced traffic distribution avoids overprovisioning appliances; automation cuts manual ops overhead |

---

## Architecture & Integrations

| AWS Service | Role with GWLB |
|---|---|
| **Amazon VPC** | Networking foundation; GWLB deployed inside a VPC, uses VPC routing + endpoints to reach other VPCs |
| **AWS Transit Gateway** | Routes traffic to GWLB endpoints → hub-and-spoke architecture with centralized inspection across VPCs/on-prem/accounts |
| **AWS PrivateLink** | Powers private endpoint connectivity (no public internet transit); underpins the security-as-a-service sharing model |
| **Third-party appliances (AWS Marketplace)** | Plug-in NGFWs, IPS, DPI tools — vendor choice flexibility |
| **Amazon CloudWatch** | Metrics on GWLB + appliance health; alarms for issue detection |
| **AWS CloudTrail** | Logs API calls to GWLB → audit trail for compliance/security investigations |

### Integration Considerations
- **Traffic routing strategies**: match routing algorithm to app/traffic patterns; path/host-based rules; weighted target groups for blue/green & canary; sticky sessions for stateful apps (watch session timeout to avoid resource exhaustion).
- **Health check configuration**: tune thresholds appropriately for your appliances.
- **Security controls**: apply appropriate policies at each integration point.

---

## Demonstrations (Hands-On Workflow)

1. **Create your first Gateway Load Balancer** — basic setup via AWS Console
2. **Create & register target group instances** — attach EC2 instances as security appliance targets
3. **Create Gateway Load Balancer endpoints** — expose GWLB privately to consumer VPCs
4. **Delete resources** (cleanup, to avoid ongoing charges):
   - VPC Console → **Endpoints** → select GWLB endpoint(s) → Actions → Delete VPC endpoints → type `delete` to confirm
   - EC2 Console → **Load Balancers** → select GWLB → Actions → Delete load balancer → type `confirm`
   - EC2 Console → **Target Groups** → select group → Actions → Delete → confirm
   - EC2 Console → **Instances** → select all → Instance state → Terminate (delete) instance → confirm

> ⚠️ Demo cost warning: these exercises can incur AWS charges — clean up all resources after practicing.

---

## Quick Reference

- **GWLB = Layer 3**, ALB = Layer 7, NLB = Layer 4.
- **GENEVE** preserves original packet info during encapsulation to appliances.
- **Flow symmetry** = same appliance sees both directions of a connection (needed for stateful inspection).
- **Cross-zone load balancing** = any node can reach appliances in any AZ.
- Core AWS integrations: **VPC, Transit Gateway, PrivateLink, CloudWatch, CloudTrail**.
- Primary use case: insert firewalls/IDS/DPI transparently and at scale, without changing app configs.