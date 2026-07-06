## 01/07
Question: Under which condition would your data be preserved if you use Amazon EC2 instance store as a block volume?

Options:
- The attached EC2 instance stops
- The attached EC2 instance reboots
- The attached EC2 instance terminates
- The attached EC2 instance hibernates

Answer: The attached EC2 instance reboots
Reason: EC2 instance store is ephemeral. Data persists only during reboot. Stopping, terminating, or hibernating wipes the data.


## 02/07
Question: Which use case is appropriate for EC2 instance stores?

Options:
- High performance persistent data
- Temporary or ephemeral data
- Archival data
- Compliance data

Answer: Temporary or ephemeral data
Reason: Instance store is fast but ephemeral. Best for caches, temp files, scratch data.


## 03/07
Question: What category of storage device is Amazon EBS?

Options:
- Storage Area Network (SAN) devices
- Network attached storage (NAS) devices
- Attached disk drive on the same host
- Direct attached storage device or disk drives

Answer: Storage Area Network (SAN) devices
Reason: EBS is network-attached block storage, similar to a SAN. Instance store = DAS.


## 04/07
Question: Which of the following Amazon EBS volume types is designed to meet the needs of approximately 80% of all block storage workloads?

Options:
- gp3
- io2
- st1
- sc1

Answer: gp3
Reason: gp3 General Purpose SSD is the default for most workloads.


## 05/07
Question: What AWS service can you use to manage encryption keys to encrypt your Amazon EBS volumes?

Options:
- AWS Key Management Service
- Amazon GuardDuty
- AWS CloudFormation
- AWS Certificate Manager

Answer: AWS Key Management Service
Reason: EBS encryption uses KMS keys to manage encryption at rest.


## 06/07
Question: What functionality or benefit is provided by the Elastic Volumes feature available for use with Amazon EBS volumes?

Options:
- You can elect to have volume types automatically modified based on performance data once a day or once a week.
- You can dynamically increase capacity, change volume types, or tune performance.
- You can elect to have volume performance automatically increase based on Amazon CloudWatch data every 30 minutes.
- You can dynamically move volumes from one EC2 instance to another EC2 instance without disruption.

Answer: You can dynamically increase capacity, change volume types, or tune performance.
Reason: Elastic Volumes allows live modification of size, type, IOPS, throughput without detaching.


## 07/07
Question: Which service-native feature allows you to create point-in-time incremental backups of your Amazon EBS volumes?

Options:
- AWS Backup
- Elastic Volumes
- Replication
- EBS snapshots

Answer: EBS snapshots
Reason: EBS snapshots provide point-in-time, incremental backups stored in S3.