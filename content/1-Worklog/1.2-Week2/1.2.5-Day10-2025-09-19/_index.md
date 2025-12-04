---
title: "Day 10 - Elastic Load Balancing"
weight: 5
chapter: false
pre: "<b> 1.2.5. </b>"
---

**Date:** 2025-09-19 (Thứ Sáu)  
**Status:** "Done"  

---

# **Lecture Notes**

## Elastic Load Balancing (ELB)

### Overview

- A fully managed service distributing traffic across multiple targets (EC2, containers, etc.).  
- Supports HTTP, HTTPS, TCP, TLS.  
- Can be deployed in public or private subnets.  
- Provides DNS names; only NLB supports static IPs.  
- Includes health checks and access logs (to S3).  
- Supports sticky sessions (session affinity).  
- Types: **Application**, **Network**, **Classic**, and **Gateway** Load Balancer.

### Application Load Balancer (ALB)

- Operates at Layer 7 (HTTP/HTTPS).  
- Supports path-based routing (e.g., /mobile vs /desktop).  
- Targets: EC2, Lambda, IP addresses, containers (ECS/EKS).

![image](/images/1-Worklog/Week2/image%2017.png)

**ALB Features:**

- Host-based routing
- Path-based routing
- HTTP header-based routing
- Query string parameter-based routing
- WebSocket support
- HTTP/2 support

### Network Load Balancer (NLB)

- Operates at Layer 4 (TCP/TLS).  
- Supports static IPs and handles millions of requests per second.  
- Targets: EC2, IP addresses, containers (ECS/EKS).

**NLB Features:**

- Ultra-low latency
- Static IP addresses
- Preserve source IP
- Long-lived TCP connections
- TLS termination

### Gateway Load Balancer (GWLB)

- Operates at Layer 3 (IP packets).  
- Uses the GENEVE protocol on port 6081.  
- Routes traffic to virtual appliances such as firewalls or monitoring tools.  
- Partner list: [aws.amazon.com/elasticloadbalancing/partners](https://aws.amazon.com/elasticloadbalancing/partners)

![image](/images/1-Worklog/Week2/image%2018.png)

---

# **Exploration**

## [AWS Advanced Networking – Specialty Study Guide](https://www.amazon.co.uk/Certified-Advanced-Networking-Official-Study/dp/1119439833)

- Official study guide covering exam topics, AWS network design principles, and real-world architecture scenarios.

---

# **Hands-On Labs**

## Lab 20 – AWS Transit Gateway

1. Preparation Steps → *20-02*  
2. Create Transit Gateway → *20-03*  
3. Create TGW Attachments → *20-04*  
4. Create TGW Route Tables → *20-05*  
5. Add TGW Routes to VPC Route Tables → *20-06*

---

## Week 2 Summary

Tuần này đã hoàn thành kiến thức về AWS Networking:

Amazon VPC và Subnets  
Security Groups và NACLs  
VPC Peering và Transit Gateway  
VPN và Direct Connect  
Elastic Load Balancing (ALB, NLB, GWLB)  

**Labs completed:** 4 labs (VPC Basics, Hybrid DNS, VPC Peering, Transit Gateway)
