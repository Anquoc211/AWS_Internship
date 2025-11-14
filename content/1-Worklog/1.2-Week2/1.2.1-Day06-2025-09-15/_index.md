---
title: "Day 06 - Amazon VPC Fundamentals"
weight: 1
chapter: false
pre: "<b> 1.2.1. </b>"
---

**Date:** 2025-09-15 (Thứ Hai)  
**Status:** "Done"  

---

# **Lecture Notes**

## Networking Services on AWS

### Amazon Virtual Private Cloud (VPC)

- Amazon Virtual Private Cloud (Amazon VPC) allows you to launch AWS resources into a virtual network you define.

![image](/images/1-Worklog/Week2/c7387b7b-422f-41ee-a0b7-886017264e85.png)

- A VPC exists within a single Region. When creating a VPC, you must define an IPv4 CIDR block (required) and optionally an IPv6 one.  
- The default limit is 5 VPCs per Region per Account.  
- Commonly used to separate environments such as Production, Development, and Staging.  
- To achieve full resource isolation, use separate AWS Accounts rather than multiple VPCs.

![image](/images/1-Worklog/Week2/image.png)

### Subnets

- A subnet resides within one Availability Zone.  
- The subnet CIDR must be a subset of the parent VPC's CIDR block.  
- AWS reserves 5 IP addresses in each subnet: network, broadcast, router, DNS, and future use.  

![image](/images/1-Worklog/Week2/image%201.png)

**Reserved IP Addresses Example (10.0.0.0/24):**

- 10.0.0.0 - Network address
- 10.0.0.1 - VPC router
- 10.0.0.2 - DNS server
- 10.0.0.3 - Reserved for future use
- 10.0.0.255 - Broadcast address

---

# **Hands-On Labs**

## Lab 03 – Amazon VPC & Networking Basics

1. Create VPC → *03-03.1*  
2. Create Subnet → *03-03.2*
