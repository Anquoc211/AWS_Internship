---
title: "Day 19 - Disaster Recovery on AWS"
weight: 4
chapter: false
pre: "<b> 1.4.4. </b>"
---

**Date:** 2025-10-02 (Thứ Năm)  
**Status:** "Done"  

---

# **Lecture Notes**

## Disaster Recovery (DR) on AWS

**Disaster Recovery** is about restoring IT services after major incidents (outages, disasters, hardware failures, cyberattacks).

- **RTO (Recovery Time Objective):** How quickly to restore service.
- **RPO (Recovery Point Objective):** How much data loss (time window) is acceptable.

### DR Strategies (ordered by complexity & cost)

**Backup & Restore**

- Maintain backups only (EBS/RDS snapshots, S3/Glacier).
- Restore to new infrastructure during incidents.
- **RTO:** hours–days. **RPO:** depends on backup frequency. **Cost:** lowest.

**Pilot Light**

- Minimal core services always running on AWS.
- Scale out to full production during DR.
- **RTO:** hours. **RPO:** minutes. **Cost:** moderate.

**Warm Standby**

- Full system running at reduced scale on AWS.
- Scale up on failover.
- **RTO:** minutes–hours. **RPO:** seconds–minutes. **Cost:** higher.

**Multi-Site (Active/Active or Active/Passive)**

- Production running across on-prem and AWS, or multi-Region AWS.
- Traffic can be shifted instantly (Route 53, Global Accelerator).
- **RTO/RPO:** near zero. **Cost:** highest.

**DR Strategy Comparison:**

| Strategy | RTO | RPO | Cost | Complexity |
|----------|-----|-----|------|------------|
| Backup & Restore | Hours-Days | Hours | $ | Low |
| Pilot Light | Hours | Minutes | $$ | Medium |
| Warm Standby | Minutes | Seconds | $$$ | Medium-High |
| Multi-Site | Seconds | Near-zero | $$$$ | High |

---

## DR Best Practices

### Planning

- Define RTO and RPO requirements
- Document recovery procedures
- Identify critical systems and dependencies
- Establish communication plans

### Implementation

- Automate recovery processes
- Use multiple AZs and Regions
- Implement data replication
- Regular backup testing

### Testing

- Conduct DR drills regularly
- Test recovery procedures
- Measure actual RTO/RPO
- Update documentation

---

# **Hands-On Labs**

## Lab 14 – AWS VM Import/Export (Part 2)

4. Import Virtual Machine to AWS → *14-02.3*  
5. Deploy Instance from AMI → *14-02.4*  
6. Set Up S3 Bucket ACL → *14-03.1*  
7. Export Virtual Machine from Instance → *14-03.2*  
8. Resource Cleanup on AWS → *14-05*
