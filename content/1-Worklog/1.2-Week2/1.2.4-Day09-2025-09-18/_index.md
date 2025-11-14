---
title: "Day 09 - VPC Connectivity & Load Balancing"
weight: 4
chapter: false
pre: "<b> 1.2.4. </b>"
---

**Date:** 2025-09-18 (Thứ Năm)  
**Status:** "Done"  

---

# **Lecture Notes**

## VPC Peering & Transit Gateway

### VPC Peering

- Enables direct, private connectivity between two VPCs without traversing the Internet.  
- Does not support transitive routing or overlapping CIDRs.

![image](/images/1-Worklog/Week2/image%2012.png)
![image](/images/1-Worklog/Week2/image%2013.png)
![image](/images/1-Worklog/Week2/image%2014.png)

**VPC Peering Limitations:**

- No transitive peering
- No overlapping CIDR blocks
- Limited to 125 peering connections per VPC
- Cross-region peering supported

### AWS Transit Gateway (TGW)

- Acts as a hub to connect multiple VPCs and on-prem networks, simplifying complex mesh topologies.  
- TGW Attachments associate subnets in specific AZs with a TGW.  
- All subnets within the same AZ can reach the TGW once attached.

![image](/images/1-Worklog/Week2/image%2015.png)
![image](/images/1-Worklog/Week2/image%2016.png)

**Transit Gateway Benefits:**

- Centralized connectivity hub
- Simplified network architecture
- Scalable to thousands of VPCs
- Supports inter-region peering

---

## VPN & Direct Connect

### Site-to-Site VPN

- Establishes a secure IPSec connection between an on-premises data center and AWS VPC.  
- Consists of:  
  - **Virtual Private Gateway (VGW):** AWS-managed, multi-AZ endpoints.  
  - **Customer Gateway (CGW):** Customer-managed device or software appliance.

### AWS Direct Connect

- Provides a dedicated private network connection between an on-prem data center and AWS.  
- Typical latency: 20–30 ms.  
- In Vietnam, available through Hosted Connections (via partners).  
- Bandwidth is adjustable.

---

# **Hands-On Labs**

## Lab 10 – Hybrid DNS (Route 53 Resolver)

1. Generate Key Pair → *10-02.1*  
2. Initialize CloudFormation Template → *10-02.2*  
3. Configure Security Group → *10-02.3*  
4. Set up DNS System → *10-05*  
5. Create Route 53 Outbound Endpoint → *10-05.1*  
6. Create Resolver Rules → *10-05.2*  
7. Create Inbound Endpoints → *10-05.3*

## Lab 19 – VPC Peering

1. Initialize CloudFormation Templates → *19-02.1*  
2. Create Security Group → *19-02.2*  
3. Create EC2 Instance (Test Peering) → *19-02.3*  
4. Create Peering Connection → *19-04*  
5. Configure Route Tables (Cross-VPC) → *19-05*  
6. Enable Cross-Peer DNS → *19-06*
