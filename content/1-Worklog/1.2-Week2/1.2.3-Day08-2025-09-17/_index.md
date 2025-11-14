---
title: "Day 08 - VPC Security & Flow Logs"
weight: 3
chapter: false
pre: "<b> 1.2.3. </b>"
---

**Date:** 2025-09-17 (Thứ Tư)  
**Status:** "Done"  

---

# **Lecture Notes**

## VPC Security

### Security Group (SG)

- A **stateful** virtual firewall that controls inbound and outbound traffic to AWS resources.  
- Rules are based on protocol, port, source, or another security group.  
- Only allow rules are supported.  
- Applied to Elastic Network Interfaces (ENIs).

![image](/images/1-Worklog/Week2/image%207.png)
![image](/images/1-Worklog/Week2/image%208.png)

**Security Group Characteristics:**

- Stateful: return traffic automatically allowed
- Supports allow rules only
- Evaluates all rules before deciding
- Applies to instance level (ENI)

### Network Access Control List (NACL)

- A **stateless** virtual firewall that operates at the subnet level.  
- Rules control inbound and outbound traffic by protocol, port, and source.  
- Default NACL allows all traffic.

![image](/images/1-Worklog/Week2/image%209.png)
![image](/images/1-Worklog/Week2/image%2010.png)

**NACL Characteristics:**

- Stateless: must explicitly allow return traffic
- Supports both allow and deny rules
- Rules processed in number order
- Applies to subnet level

### VPC Flow Logs

- Capture metadata about IP traffic to and from network interfaces in your VPC.  
- Logs can be delivered to Amazon CloudWatch Logs or S3.  
- Flow Logs do not record packet payloads.

![image](/images/1-Worklog/Week2/image%2011.png)

**Flow Log Use Cases:**

- Troubleshoot connectivity issues
- Monitor traffic patterns
- Security analysis
- Compliance requirements

---

# **Hands-On Labs**

## Lab 03 – Amazon VPC & Networking (continued)

6. Launch EC2 Instances in Subnets → *04-1*  
7. Test Connection Between Instances → *04-2*  
8. Create NAT Gateway (Private ↔ Internet) → *04-3*  
9. EC2 Instance Connect Endpoint (no bastion) → *04-5*
