---
title: "Day 20 - AWS Backup & FSx"
weight: 5
chapter: false
pre: "<b> 1.4.5. </b>"
---

**Date:** 2025-10-03 (Thứ Sáu)  
**Status:** "Done"  

---

# **Lecture Notes**

## AWS Backup

Centralized backup service for automating and governing data protection at scale.

### Key Capabilities

- **Central management:** Define and apply backup policies across services.
- **Multi-service support:** EC2, EBS, RDS, DynamoDB, EFS, Storage Gateway, S3, and more.
- **Scheduling & lifecycle:** Automate backups and retention.
- **Compliance:** Support for governance and audit requirements.

### Benefits

- **Operational simplicity:** No custom scripts or disparate tools.
- **Time savings:** Automated, policy-driven protection.
- **Reporting & audit:** Visibility into backup status and compliance.

### Backup Vault Lock

- Immutability controls to prevent modifications or deletions of protected backups for strict compliance.

**AWS Backup Features:**

- Cross-region backup copy
- Cross-account backup
- Backup policies (plans)
- Lifecycle management
- Encryption at rest
- Tag-based backup policies

**Backup Plan Example:**

```json
{
  "BackupPlanName": "DailyBackups",
  "Rules": [{
    "RuleName": "DailyRule",
    "ScheduleExpression": "cron(0 5 ? * * *)",
    "StartWindowMinutes": 60,
    "CompletionWindowMinutes": 120,
    "Lifecycle": {
      "DeleteAfterDays": 30,
      "MoveToColdStorageAfterDays": 7
    }
  }]
}
```

---

# **Exploration**

## [AWS Skill Builder](https://skillbuilder.aws/?showRedirectNotFoundBanner=true)

- Curated learning plans and deep-dive content for storage specialists:
  - Storage Learning Plan: Block Storage  
  - Storage Learning Plan: Object Storage

---

# **Hands-On Labs**

## Lab 13 – AWS Backup

1. Create S3 Bucket → *13-02.1*  
2. Deploy Infrastructure → *13-02.2*  
3. Create Backup Plan → *13-03*  
4. Set Up Notifications → *13-04*  
5. Test Restore → *13-05*  
6. Clean Up Resources → *13-06*

## Lab 25 – Amazon FSx (File Systems)

1. Create SSD Multi-AZ File System → *25-2.2*  
2. Create HDD Multi-AZ File System → *25-2.3*  
3. Create New File Shares → *25-3*  
4. Test Performance → *25-4*  
5. Monitor Performance → *25-5*  
6. Enable Data Deduplication → *25-6*  
7. Enable Shadow Copies → *25-7*  
8. Manage User Sessions and Open Files → *25-8*  
9. Enable User Storage Quotas → *25-9*  
10. Scale Throughput Capacity → *25-11*  
11. Scale Storage Capacity → *25-12*  
12. Delete Environment → *25-13*

---

## Week 4 Summary

Tuần này đã hoàn thành kiến thức về AWS Storage:

Amazon S3 và Storage Classes  
S3 Static Website và CORS  
AWS Snow Family  
AWS Storage Gateway  
Disaster Recovery Strategies  
AWS Backup  

**Labs completed:** 5 labs (Backup, VM Import/Export, Storage Gateway, FSx, S3 & CloudFront)
