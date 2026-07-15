# AWS Monitoring & Management — Study Reference

## Part 1: Core AWS Monitoring & Management Services

### Amazon CloudWatch
**Definition:** A monitoring and observability service that collects **metrics, logs, and events** from AWS resources and applications. Used to monitor performance and trigger **alarms** based on thresholds.

**Example:** Monitor EC2 CPU utilization and send an email notification when CPU usage exceeds 80%.

**Key use cases:** dashboards, alarms, log aggregation, custom metrics via the CloudWatch Agent.

---

### AWS CloudTrail
**Definition:** Records all **API calls and user activity** performed in your AWS account — used for auditing, troubleshooting, and security analysis.

**Example:** Check who deleted an EC2 instance, when it happened, and from which IP address.

**Key use cases:** security investigations, compliance audits, tracking "who did what, when."

---

### AWS Systems Manager
**Definition:** A management service for securely managing, automating, and operating EC2 instances (and other AWS resources) from a **central console** — without needing to log into each server individually.

**Example:** Install software updates or run shell commands across multiple EC2 instances at once.

**Key use cases:** Session Manager (browser-based shell access without SSH/open ports), Run Command, Patch Management, Automation.

---

### AWS Config
**Definition:** Continuously records AWS resource **configurations** and evaluates them against organizational rules and best practices (compliance).

**Example:** Detect when an S3 bucket becomes publicly accessible, or when an EC2 instance launches without required tags.

**Key use cases:** compliance monitoring, configuration drift detection, security-group rule enforcement.

> ⚠️ Note: AWS Config may incur charges even under the Free Tier — enable deliberately.

---

### AWS Trusted Advisor
**Definition:** An automated recommendation service that analyzes your AWS environment and provides best-practice guidance across five categories: **cost optimization, security, performance, fault tolerance, and service limits.**

**Example:** Identify idle EC2 instances, unused EBS volumes, overly open security groups, or resources that can be optimized to cut costs.

---

### Quick Comparison Table

| Service | Primary Purpose | Answers the Question... |
|---|---|---|
| **CloudWatch** | Performance monitoring & alarms | "Is my system healthy right now?" |
| **CloudTrail** | API/activity audit log | "Who did what, and when?" |
| **Systems Manager** | Centralized fleet management | "How do I manage many instances at once, securely?" |
| **AWS Config** | Configuration compliance | "Is my resource configuration correct/compliant?" |
| **Trusted Advisor** | Best-practice recommendations | "What should I fix or optimize?" |

---

## Part 2: Hands-On Lab Manual — Smart Cloud Infrastructure Monitoring System

A 12-step lab project that builds a monitored EC2 web server and exercises each of the services above in a realistic incident-response workflow.

### Lab 1 — Launch EC2 Instance
- Launch an **Ubuntu EC2 instance** (`t2.micro`)
- Create a **key pair** for SSH access
- Configure a **Security Group** allowing:
  - **SSH (port 22)**
  - **HTTP (port 80)**

### Lab 2 — Connect via SSH
```bash
# Ubuntu
ssh -i key.pem ubuntu@PUBLIC_IP

# Amazon Linux
ssh -i key.pem ec2-user@PUBLIC_IP
```

### Lab 3 — Install a Web Server
```bash
# Ubuntu (Apache)
sudo apt update && sudo apt install apache2 -y

# Amazon Linux (httpd)
sudo yum update -y
sudo yum install httpd -y
sudo systemctl enable --now httpd
```

### Lab 4 — Deploy a Website
- Place a simple HTML file in `/var/www/html`
- Verify it loads by visiting the instance's public IP in a browser

### Lab 5 — Create a CloudWatch Dashboard
Build a dashboard tracking:
- `CPUUtilization`
- `NetworkIn`
- `NetworkOut`
- `DiskReadBytes`
- `DiskWriteBytes`

### Lab 6 — Create a CloudWatch Alarm
- Create an alarm: **CPU > 60%**
- Configure an **SNS topic** for email notifications
- Confirm the SNS **email subscription** (check inbox for confirmation link)

### Lab 7 — Generate Load & Observe
- Use `stress` or `stress-ng` to artificially spike CPU load
- Watch CloudWatch metrics update in near real-time
- Confirm the alarm transitions to **ALARM** state and the notification is received

```bash
# Example (install + run stress)
sudo apt install stress -y
stress --cpu 2 --timeout 120
```

### Lab 8 — Review CloudTrail Event History
- After performing EC2/S3 actions, open **CloudTrail → Event History**
- Review the logged API events (who, what, when, source IP)

### Lab 9 — Enable AWS Config
- Enable AWS Config
- Create a **compliance rule** for Security Groups (e.g., flag overly permissive inbound rules)
- ⚠️ Note: may incur charges even on Free Tier

### Lab 10 — Systems Manager Session Manager
- Attach the **`AmazonSSMManagedInstanceCore`** IAM role to the EC2 instance
- Connect via **Systems Manager → Session Manager** (browser-based shell — no SSH key or open port 22 needed)

### Lab 11 — Install CloudWatch Agent
- Install and configure the **CloudWatch Agent** on the instance
- Configure it to ship logs (e.g., Apache/httpd logs) for centralized analysis in CloudWatch Logs

### Lab 12 — Incident Response Simulation
1. Investigate a **high CPU** scenario using:
   - **CloudWatch** (metrics/alarms)
   - **CloudTrail** (recent activity/API calls)
   - **Systems Manager** (remote command execution/session access)
2. Restart the web service:
   ```bash
   # Ubuntu
   sudo systemctl restart apache2

   # Amazon Linux
   sudo systemctl restart httpd
   ```
3. Verify recovery — confirm CPU normalizes and the website is reachable again

---

## How the Labs Map to the Core Services

| Lab(s) | Service Exercised |
|---|---|
| 5, 6, 7, 11 | **CloudWatch** — dashboards, alarms, load testing, log shipping |
| 8 | **CloudTrail** — auditing API activity |
| 9 | **AWS Config** — compliance rules |
| 10 | **Systems Manager** — secure, keyless remote access |
| 12 | **All four together** — realistic incident response workflow |

---

### Quick Reference Summary
- **CloudWatch** = metrics + alarms + logs → performance monitoring
- **CloudTrail** = API/activity history → security & audit
- **Systems Manager** = centralized, agent-based management → no SSH key needed with Session Manager
- **AWS Config** = configuration compliance tracking → drift & misconfiguration detection
- **Trusted Advisor** = automated best-practice recommendations → cost, security, performance, fault tolerance, service limits
- End-to-end lab flow: **provision → serve → monitor → alert → stress-test → audit → enforce compliance → manage remotely → log → respond to incident**