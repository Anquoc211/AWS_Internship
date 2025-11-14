---
title: "Day 14 - EC2 Auto Scaling"
weight: 4
chapter: false
pre: "<b> 1.3.4. </b>"
---

**Date:** 2025-09-25 (Thứ Năm)  
**Status:** "Done"  

---

# **Lecture Notes**

## Amazon EC2 Auto Scaling

- **EC2 Auto Scaling** automatically adjusts the number of EC2 instances based on demand.

**Benefits**

- Elastic capacity adjustment  
- Increased application availability  
- Cost optimization  

**Components**

- **Auto Scaling Group (ASG)** – logical group of EC2 instances  
- **Launch Template / Configuration** – defines instance parameters  
- **Scaling Policies** – rules for adding/removing instances  

### Scaling Policies

- **Simple / Step Scaling** – add/remove instances when thresholds are met  
- **Target Tracking** – maintain a metric (e.g., CPU = 50%)  
- **Scheduled Scaling** – scale on a predefined schedule  
- **Predictive Scaling** – uses ML to forecast and scale proactively  

**Scaling Policy Examples:**

```json
{
  "TargetTrackingScalingPolicyConfiguration": {
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0
  }
}
```

### Integration with Load Balancer

- ASGs often pair with **Elastic Load Balancers (ELB)**.  
- New instances automatically register; terminated instances deregister automatically.

**Auto Scaling Best Practices:**

- Use multiple AZs for high availability
- Set appropriate cooldown periods
- Monitor CloudWatch metrics
- Use lifecycle hooks for custom actions
- Test scaling policies before production

---

## EC2 Pricing Options

- **On-Demand:** Pay per hour/second. Most expensive but flexible.  
- **Reserved Instances:** 1- or 3-year commitment for discount; tied to specific instance type/family.  
- **Savings Plans:** 1- or 3-year commitment; flexible across instance families.  
- **Spot Instances:** Use spare capacity at up to 90% discount; can be terminated with 2-minute notice.  

> Combine multiple pricing models within an Auto Scaling Group for cost optimization.

**Pricing Comparison:**

| Model | Discount | Flexibility | Commitment |
|-------|----------|-------------|------------|
| On-Demand | 0% | High | None |
| Reserved | 40-60% | Low | 1-3 years |
| Savings Plans | 40-60% | Medium | 1-3 years |
| Spot | 50-90% | Low | None |

---

# **Hands-On Labs**

## Lab 09 – AWS Support Plans

1. AWS Support Packages → *09-01*  
2. Types of Support Requests → *09-02*  
3. Change Support Package → *09-03*  
4. Manage Support Requests → *09-04*
