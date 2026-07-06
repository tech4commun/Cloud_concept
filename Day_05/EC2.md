
# Introduction to Amazon EC2 — Lab Guide

**Duration:** ~1 hour
**Goal:** Launch, monitor, resize, and terminate an Amazon EC2 instance.

## Overview

Amazon EC2 (Elastic Compute Cloud) provides resizable compute capacity in the cloud. You pay only for what you use, and instances can be launched or scaled in minutes.

### Objectives
- Launch a web server with termination protection enabled
- Monitor an EC2 instance
- Update a security group to allow HTTP access
- Resize an instance (type + EBS volume)
- Test termination protection
- Terminate the instance

---

## Task 1: Launch Your EC2 Instance

1. Go to **EC2** in the AWS Console → **Launch instance**.
2. **Name:** `Web Server`
3. **AMI:** Amazon Linux 2023
4. **Instance type:** `t3.micro` (2 vCPUs, 1 GiB RAM)
5. **Key pair:** Proceed without a key pair (not needed for this lab)
6. **Network settings → Edit:**
   - VPC: the one named *Lab VPC*
   - Subnet: *Public Subnet 1*
7. **Firewall (security groups):** Select existing → `Web Server security group`
8. **Storage:** Leave default (8 GiB root volume)
9. **Advanced details → Termination protection:** `Enable`
10. **User data** (paste as-is):

```bash
#!/bin/bash
dnf -y install httpd
systemctl enable httpd
systemctl start httpd
echo '<html><h1>Hello From Your Web Server!</h1></html>' > /var/www/html/index.html
```

   This installs Apache, enables it on boot, starts it, and creates a simple webpage.

11. Click **Launch instance**.
12. Go to **Instances** and wait until:
    - Instance state: `Running`
    - Status check: `3/3 checks passed`

✅ **Done:** Instance launched successfully.

---

## Task 2: Monitor Your Instance

1. Select the instance → **Status and alarms** tab
   - Confirm System / Instance / Attached EBS reachability checks pass.
2. **Monitoring** tab → view CloudWatch metrics (limited data since instance is new).
3. **Actions → Monitor and troubleshoot → Get system log**
   - Shows console output — useful for diagnosing boot/service issues.
4. **Actions → Monitor and troubleshoot → Get instance screenshot**
   - Shows what the instance display would look like if a monitor were attached.

✅ **Done:** Explored ways to monitor an instance.

---

## Task 3: Update Security Group & Access the Web Server

1. Copy the instance's **Public IPv4 address**.
2. Open it in a browser (`http://<ip>`) → it won't load yet.
   - **Why?** The security group blocks inbound traffic on port 80 (HTTP).
3. Go to **Security Groups** → select `Web Server security group`.
4. **Inbound rules** tab → **Edit inbound rules → Add rule:**
   - Type: `HTTP`
   - Source: `Anywhere-IPv4`
5. **Save rules** (creates rules for both `0.0.0.0/0` and `::/0`).
   > Note: "Anywhere" access is not recommended for production.
6. Refresh the browser tab — you should see:
   `Hello From Your Web Server!`

✅ **Done:** HTTP traffic now allowed to the instance.

---

## Task 4: Resize Your Instance (Type + EBS Volume)

### Stop the instance
1. Select instance → **Instance state → Stop instance** → confirm.
2. Wait for state: `Stopped`.

### Change instance type
1. **Actions → Instance settings → Change instance type**
2. New type: `t3.small` → **Change**

### Resize the EBS volume
1. Go to **Volumes** (left nav, under Elastic Block Store).
2. Select the volume → **Actions → Modify volume**
3. Change size: `8 GiB` → `10 GiB` → **Modify** → confirm.

### Restart the instance
1. Go to **Instances** → select `Web Server`
2. **Instance state → Start instance**

> Volume changes go through: `Modifying → Optimizing → Complete`.

✅ **Done:** Instance upgraded to `t3.small` with a 10 GiB volume.

---

## Task 5: Test Termination Protection

1. Select instance → **Instance state → Terminate (delete) instance** → confirm.
2. **Expected error:**
   `Failed to terminate: instance may not be terminated. Modify its 'disableApiTermination' attribute and try again.`
   - This confirms termination protection is working.
3. To actually terminate:
   - **Actions → Instance settings → Change termination protection**
   - Uncheck **Enable** → **Save**
4. Refresh, then **Instance state → Terminate (delete) instance** → confirm.
5. Wait ~30 seconds for state: `Terminated`.

✅ **Done:** Termination protection tested and instance terminated.

---

## Summary

| Step | Action |
|------|--------|
| 1 | Launched EC2 instance with termination protection |
| 2 | Monitored instance health, logs, and screenshot |
| 3 | Opened HTTP (port 80) via security group |
| 4 | Resized instance type & EBS volume |
| 5 | Disabled protection and terminated instance |

## Additional Resources
- [Launch Your Instance](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
- [Amazon EC2 Instance Types](https://aws.amazon.com/ec2/instance-types/)
- [Amazon Machine Images (AMI)](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/AMIs.html)
- [Security Groups](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-security-groups.html)
- [Termination Protection](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/terminating-instances.html#Using_ChangingDisableAPITermination)