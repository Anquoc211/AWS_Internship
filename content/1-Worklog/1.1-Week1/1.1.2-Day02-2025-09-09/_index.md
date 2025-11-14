---
title: "Day 02 - AWS Global Infrastructure"
weight: 2
chapter: false
pre: "<b> 1.1.2. </b>"
---

**Date:** 2025-09-09 (Thứ Ba)  
**Status:** "Done"  

---

# **Lecture Notes**

## AWS Infrastructure

### Data Centers

- Each data center can host tens of thousands of servers.  
- AWS builds and manages its own custom hardware for efficiency and reliability.

### Availability Zone (AZ)

- One or more physically separate data centers within a Region.  
- Each AZ is designed for fault isolation.  
- Connected via low-latency, high-throughput private links.  
- AWS recommends deploying workloads across at least two AZs.

### Region

- A Region contains at least three Availability Zones.  
- There are currently 25+ Regions worldwide.  
- Regions are interconnected by the AWS backbone network.  
- Most services are Region-scoped by default.

### Edge Locations

- Global network of edge sites designed to serve content with minimal latency.  
- Used by services such as:  
  - Amazon CloudFront (CDN)  
  - AWS WAF (Web Application Firewall)  
  - Amazon Route 53 (DNS Service)

---

# **Hands-On Labs**

### Lab 01 – AWS Account & IAM Setup

1. Create an AWS Account → *01-01*  
2. Configure Virtual MFA Device → *01-02*  
3. Create Admin Group and Admin User → *01-03*  
4. Account Authentication Support → *01-04*
