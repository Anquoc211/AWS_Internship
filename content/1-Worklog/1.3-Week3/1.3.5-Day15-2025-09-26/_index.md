---
title: "Day 15 - Lightsail, EFS & FSx"
weight: 5
chapter: false
pre: "<b> 1.3.5. </b>"
---

**Date:** 2025-09-26 (Thứ Sáu)  
**Status:** "Done"  

---

# **Lecture Notes**

## Amazon Lightsail

- Simplified compute service with predictable monthly pricing (starting ~$3.5/month).  
- Includes bundled data transfer at lower rates than EC2.  
- Ideal for small workloads, development, or testing environments.  
- Supports snapshots for backups.  
- Runs inside a managed VPC and can connect to standard VPCs via one-click peering.

**Lightsail Use Cases:**

- Simple web applications
- WordPress sites
- Development/test environments
- Small business applications
- Learning and experimentation

**Lightsail vs EC2:**

| Feature | Lightsail | EC2 |
|---------|-----------|-----|
| Pricing | Fixed monthly | Pay-as-you-go |
| Complexity | Simple | Advanced |
| Scalability | Limited | Unlimited |
| Target | Small projects | Enterprise |

---

## Amazon EFS (Elastic File System)

- Fully managed NFSv4 file system mountable by multiple EC2 instances simultaneously.  
- Scales automatically to petabytes.  
- Pay only for the storage used (unlike EBS provisioned size).  
- Can be mounted from on-prem via VPN or Direct Connect.

**EFS Features:**

- Concurrent access from multiple instances
- Automatic scaling
- Regional service (multi-AZ)
- Lifecycle management
- Encryption at rest and in transit

**EFS Storage Classes:**

- **Standard:** Frequently accessed files
- **Infrequent Access (IA):** Lower cost for rarely accessed files
- **One Zone:** Single AZ for cost savings

---

## Amazon FSx

- Managed, scalable file systems for Windows, Lustre, and NetApp ONTAP.  
- AWS handles setup, scaling, and backups.  
- Accessible from EC2, on-prem servers, or users via SMB or NFS protocols.

**FSx Variants:**

### FSx for Windows File Server

- Native Windows file system
- SMB protocol support
- Active Directory integration
- DFS namespaces

### FSx for Lustre

- High-performance computing
- Machine learning workloads
- Sub-millisecond latencies
- S3 integration

### FSx for NetApp ONTAP

- Multi-protocol (NFS, SMB, iSCSI)
- Data deduplication and compression
- Snapshots and replication

---

## AWS Application Migration Service (MGN)

- Used for migrating or replicating physical/virtual servers to AWS for DR or modernization.  
- Continuously replicates source machines to lightweight staging instances on EC2.  
- During cut-over, MGN launches fully functional EC2 instances from the replicated data.

**Migration Phases:**

1. **Install agent** on source servers
2. **Continuous replication** to AWS
3. **Testing** with non-disruptive test instances
4. **Cutover** to production

---

# **Exploration**

## [Microsoft Workloads on AWS](https://youtube.com/playlist?list=PLhr1KZpdzukdJllxulUM7pMB7aJ2_FfTP&si=HMwzvhqsFFljKfnC)

- A curated series covering deployment, optimization, and best practices for running Microsoft workloads on AWS.

---

## Week 3 Summary

Tuần này đã hoàn thành kiến thức về AWS Compute:

Amazon EC2 và Instance Types  
AMI, EBS, Instance Store  
EC2 Auto Scaling  
EC2 Pricing Options  
Lightsail, EFS, FSx  

**Labs completed:** 3 labs (IAM Setup, Budgets, Support Plans)
