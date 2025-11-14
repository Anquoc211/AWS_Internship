---
title: "Day 07 - VPC Routing & Network Interfaces"
weight: 2
chapter: false
pre: "<b> 1.2.2. </b>"
---

**Date:** 2025-09-16 (Thứ Ba)  
**Status:** "Done"  

---

# **Lecture Notes**

## VPC Routing & ENI

### Route Tables

- A route table defines how traffic is directed.  
- Each VPC has a default route table containing only a local route allowing internal communication between subnets.  
- Custom route tables can be created, but the local route cannot be deleted.

![image](/images/1-Worklog/Week2/f02d3571-7a7c-4388-a88f-87929790a60b.png)

### Elastic Network Interface (ENI)

- An ENI is a virtual network card that can be moved between EC2 instances.  
- It retains its private IP, Elastic IP address, and MAC address when reassigned.  
- Elastic IP (EIP) is a static public IPv4 address that can be associated with an ENI.  
- Unused EIPs incur charges.

![image](/images/1-Worklog/Week2/image%202.png)

**ENI Use Cases:**

- Management network separate from data network
- Network and security appliances
- Dual-homed instances with workloads on distinct subnets
- Low-budget, high-availability solution

### VPC Endpoints

- A **VPC Endpoint** enables private connectivity to supported AWS services via **AWS PrivateLink** without using the public Internet.  
- Two types:  
  - **Interface Endpoint:** Uses an ENI with a private IP.  
  - **Gateway Endpoint:** Uses route tables (available for S3 and DynamoDB only).

![image](/images/1-Worklog/Week2/image%203.png)

---

# **Hands-On Labs**

## Lab 03 – Amazon VPC & Networking (continued)

3. Create Internet Gateway (IGW) → *03-03.3*  
4. Create Route Table (Outbound via IGW) → *03-03.4*  
5. Create Security Groups → *03-03.5*
