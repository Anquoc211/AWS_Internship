---
title: "Day 16 - Amazon S3 Fundamentals"
weight: 1
chapter: false
pre: "<b> 1.4.1. </b>"
---

**Date:** 2025-09-29 (Thứ Hai)  
**Status:** "Done"  

---

# **Lecture Notes**

## Storage Services on AWS

### Amazon Simple Storage Service (S3)

Amazon S3 is an object storage service designed to store and retrieve any amount of data from anywhere on the web. It offers virtually unlimited scalability, high availability, strong security, and excellent performance.

### Core S3 Features

- **Buckets and Objects:** Data is stored as objects inside buckets. Each object can be up to 5 TB.
- **Availability and Durability:** S3 is designed for 99.99% availability and 99.999999999% (11 nines) durability.
- **Security:** Multiple layers of security including IAM, bucket policies, ACLs, and encryption.
- **Scalability:** Automatically scales storage and request throughput without performance degradation.

**S3 Object Structure:**

- **Key:** Object name/path
- **Value:** Object data
- **Version ID:** For versioning
- **Metadata:** System and user metadata
- **Access Control:** Permissions

### S3 Access Points

Access Points simplify managing data access for shared datasets in S3.

- **Per-application access control:** Each access point has its own policy.
- **Operational simplicity:** Eases permission management for shared datasets used by many applications.
- **Network controls:** Can be configured to accept requests only from specific VPCs.

---

### S3 Storage Classes

Choose among storage classes optimized for different access patterns and cost profiles:

- **S3 Standard:** For frequently accessed data; highest availability and performance.
- **S3 Intelligent-Tiering:** Automatically moves objects between tiers to optimize cost.
- **S3 Standard-IA (Infrequent Access):** Lower cost for infrequently accessed data with millisecond retrieval.
- **S3 One Zone-IA:** Like Standard-IA but stored in a single AZ.
- **S3 Glacier Flexible Retrieval:** Low-cost archival with minutes-to-hours retrieval.
- **S3 Glacier Deep Archive:** Lowest-cost archival with ~12-hour retrieval.

**Storage Class Comparison:**

| Class | Durability | Availability | Min Storage | Retrieval |
|-------|------------|--------------|-------------|-----------|
| Standard | 11 9's | 99.99% | None | Instant |
| Intelligent-Tiering | 11 9's | 99.9% | None | Instant |
| Standard-IA | 11 9's | 99.9% | 30 days | Instant |
| One Zone-IA | 11 9's | 99.5% | 30 days | Instant |
| Glacier Flexible | 11 9's | 99.99% | 90 days | Minutes-hours |
| Glacier Deep Archive | 11 9's | 99.99% | 180 days | 12 hours |

---

# **Hands-On Labs**

## Lab 57 – Amazon S3 & CloudFront (Part 1)

1. Create S3 Bucket → *57-2.1*  
2. Load Data → *57-2.2*  
3. Enable Static Website → *57-3*  
4. Configure Public Access Block → *57-4*
