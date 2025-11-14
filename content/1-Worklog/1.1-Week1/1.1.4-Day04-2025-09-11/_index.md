---
title: "Day 04 - Cost Optimization on AWS"
weight: 4
chapter: false
pre: "<b> 1.1.4. </b>"
---

**Date:** 2025-09-11 (Thứ Năm)  
**Status:** "Done"  

---

# **Lecture Notes**

## Cost Optimization on AWS

### Cost Optimization Strategies

- Choose the right resource types and Regions.  
- Use pricing models such as Reserved Instances, Savings Plans, and Spot Instances.  
- Remove or schedule idle resources.  
- Leverage serverless architectures.  
- Continuously review and improve cost efficiency with AWS Budgets and Cost Explorer.  
- Tag resources with Cost Allocation Tags for department-level tracking.

### AWS Pricing Calculator

[calculator.aws](https://calculator.aws/#/)

- Create and share cost estimates for common services.  
- Pricing varies by Region.

![image](/images/1-Worklog/Week1/image%202.png)

**Key Features:**

- Estimate costs before deployment
- Compare pricing across regions
- Export and share estimates
- Template-based estimation

---

## AWS Support Plans

- Four tiers: **Basic**, **Developer**, **Business**, and **Enterprise**.  
- Plans can be upgraded temporarily during critical incidents.

### Support Plan Comparison

| Feature | Basic | Developer | Business | Enterprise |
|---------|-------|-----------|----------|------------|
| Cost | Free | $29/month | $100/month | $15,000/month |
| Response Time | N/A | 12-24 hours | 1 hour (urgent) | 15 min (critical) |
| Technical Support | Forums only | Business hours | 24/7 | 24/7 + TAM |

---

# **Hands-On Labs**

### Lab 07 – AWS Budgets & Cost Management

1. Create Budget by Template → *07-01*  
2. Create Cost Budget Tutorial → *07-02*  
3. Create Usage Budget → *07-03*  
4. Create Reserved Instance (RI) Budget → *07-04*  
5. Create Savings Plans Budget → *07-05*  
6. Clean Up Budgets → *07-06*

### Lab 09 – AWS Support Plans

1. AWS Support Packages → *09-01*  
2. Types of Support Requests → *09-02*  
3. Change Support Package → *09-03*  
4. Manage Support Requests → *09-04*
