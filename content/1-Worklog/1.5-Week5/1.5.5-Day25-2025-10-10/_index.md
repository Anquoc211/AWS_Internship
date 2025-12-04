---
title: "Day 25 - AWS Security Hub & Automation"
weight: 5
chapter: false
pre: "<b> 1.5.5. </b>"
---

**Date:** 2025-10-10 (Thứ Sáu)  
**Status:** "Done"  

---

# **Lecture Notes**

## AWS Security Hub

- Aggregates and prioritizes security findings and posture across accounts/services.

**Capabilities**

- Automated checks, normalized findings, prioritized remediation workflows.
- Compliance standards: CIS AWS Foundations, PCI DSS, AWS Foundational Security Best Practices.

**Integrations**

- GuardDuty, Inspector, Macie, Firewall Manager, IAM Access Analyzer, plus partner tools.

**Outcomes**

- Less time aggregating, more time fixing; unified visibility and improved security hygiene.

![image](/images/1-Worklog/Week5/image%2013.png)

**Security Hub Features:**

- Continuous security posture monitoring
- Automated compliance checks
- Centralized findings across accounts
- Integration with 50+ AWS and partner services
- Custom insights and dashboards
- Automated remediation with EventBridge

**Security Standards:**

- **AWS Foundational Security Best Practices:** 50+ controls
- **CIS AWS Foundations Benchmark:** Industry best practices
- **PCI DSS:** Payment card industry standards
- **NIST:** National Institute of Standards framework

---

## Security Automation

**AWS Services for Automation:**

- **AWS Config:** Track resource configuration changes
- **Amazon EventBridge:** Event-driven automation
- **AWS Lambda:** Serverless remediation functions
- **AWS Systems Manager:** Automated patching and compliance

**Common Automation Patterns:**

- Auto-remediate non-compliant resources
- Automated incident response
- Security group rule validation
- Encryption enforcement
- Tag compliance

---

# **Exploration**

## [AWS Certified Security – Specialty: All-in-One Exam Guide (SCS-C01)](https://dokumen.pub/aws-certified-security-specialty-all-in-one-exam-guide-exam-scs-c01-1260461734-9781260461725-1260461726.html)

- Comprehensive preparation material for the Security Specialty certification.

---

# **Hands-On Labs**

## Lab 18 – AWS Security Hub

1. Enable Security Hub → *18-02*  
2. Score for Each Set of Criteria → *18-03*  
3. Clean Up Resources → *18-04*

## Lab 22 – AWS Lambda Automation with Slack

1. Create VPC → *22-2.1*  
2. Create Security Group → *22-2.2*  
3. Create EC2 Instance → *22-2.3*  
4. Incoming Webhooks (Slack) → *22-2.4*  
5. Create Tag for Instance → *22-3*  
6. Create Role for Lambda → *22-4*  
7. Function: Stop Instance → *22-5.1*  
8. Function: Start Instance → *22-5.2*  
9. Check Result → *22-6*  
10. Clean Up Resources → *22-7*

## Lab 27 – AWS Resource Groups & Tagging (Part 2)

4. Use Tags with CLI → *27-2.2*  
5. Create a Resource Group → *27-3*  
6. Clean Up Resources → *27-4*

## Lab 33 – AWS KMS & CloudTrail Integration (Part 2)

6. Create CloudTrail → *33-5.1*  
7. Log to CloudTrail → *33-5.2*  
8. Create Amazon Athena → *33-5.3*  
9. Query with Athena → *33-5.4*  
10. Test & Share Encrypted S3 Data → *33-6*  
11. Resource Cleanup → *33-7*

## Lab 44 – IAM Advanced Role Control

1. Create IAM Group → *44-2*  
2. Create IAM Users → *44-3.1*  
3. Check Permissions → *44-3.2*  
4. Create Admin IAM Role → *44-4.1*  
5. Configure Switch Role → *44-4.2*  
6. Restrict Switch Role by IP → *44-4.3.1*  
7. Restrict Switch Role by Time → *44-4.3.2*  
8. Clean Up Resources → *44-5*

---

## Week 5 Summary

Tuần này đã hoàn thành kiến thức về AWS Security:

Shared Responsibility Model  
AWS IAM (Users, Groups, Roles, Policies)  
Amazon Cognito  
AWS Organizations & SCPs  
AWS Identity Center  
AWS KMS  
AWS Security Hub  

**Labs completed:** 8 labs (Security Hub, Lambda Automation, Resource Groups, IAM Policies, KMS & CloudTrail, Advanced Role Control)
