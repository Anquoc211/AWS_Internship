---
title: "Workshop: Building an Online Library with AWS Serverless"
date: "2025-01-15T09:00:00+07:00"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Building a Serverless Online Library Platform

## Workshop Overview

In this hands-on workshop, you'll build a complete serverless online library platform using AWS services. The application features user authentication, file uploads with admin approval workflow, secure content delivery via CloudFront, and search functionality.

**Duration:** 6 modules (~6 hours)

**Cost:** ~$10/month (excluding AWS Free Tier)

**Difficulty:** Intermediate

---

## What You'll Build

A production-ready online library platform with:

- **User Authentication** with Amazon Cognito
- **File Upload System** using S3 Presigned URLs
- **Admin Approval Workflow** with role-based access control
- **Secure Content Delivery** via CloudFront Signed URLs
- **Search Functionality** using DynamoDB GSI
- **CI/CD Pipeline** with AWS Amplify

---

## Architecture

![Architecture Diagram](/images/2-Proposal/Architect.jpeg)

The solution uses a fully serverless architecture:

- **Frontend:** Next.js hosted on AWS Amplify
- **Authentication:** Amazon Cognito User Pools
- **API:** Amazon API Gateway (HTTP API)
- **Compute:** AWS Lambda (Python)
- **Storage:** Amazon S3 with CloudFront CDN
- **Database:** Amazon DynamoDB
- **Infrastructure:** AWS CDK (TypeScript)

---

## Prerequisites

Before starting this workshop, you should have:

- **AWS Account** with administrative access
- **AWS CLI** installed and configured
- **Node.js 18+** and npm/yarn
- **Python 3.9+** installed
- **Git** for version control
- **Basic knowledge** of:
  - JavaScript/TypeScript
  - Python
  - REST APIs
  - Basic AWS services

---

## Workshop Modules

### [Module 1: Foundation & Authentication](5.1-module1/)
- Set up AWS CDK project
- Deploy Cognito User Pool
- Configure authentication flow
- Build login/registration pages

### [Module 2: Upload Infrastructure](5.2-module2/)
- Create S3 buckets with proper policies
- Implement presigned URL generation
- Build upload form with progress tracking
- Store metadata in DynamoDB

### [Module 3: Admin Approval System](5.3-module3/)
- Set up admin user groups
- Create approval Lambda functions
- Build admin dashboard UI
- Implement status tracking

### [Module 4: Content Delivery](5.4-module4/)
- Configure CloudFront distribution
- Implement Origin Access Control (OAC)
- Generate CloudFront Signed URLs
- Build PDF/ePub reader interface

### [Module 5: Search & Discovery](5.5-module5/)
- Design DynamoDB GSI for search
- Create search Lambda function
- Build search UI with filters
- Implement pagination

### [Module 6: Deployment & Operations](5.6-module6/)
- Set up CI/CD with Amplify
- Configure monitoring and alerts
- Implement cost optimization
- Security hardening

---

## Learning Objectives

By the end of this workshop, you will:

✅ Understand serverless architecture patterns  
✅ Implement secure authentication with Cognito  
✅ Handle file uploads at scale with S3  
✅ Build approval workflows with Lambda  
✅ Deliver content securely via CloudFront  
✅ Design efficient search with DynamoDB  
✅ Deploy with CI/CD automation  
✅ Monitor and optimize costs  

---

## Cost Estimation

**Monthly costs (without Free Tier):** ~$9.80

| Service | Estimated Cost |
|---------|---------------|
| Amazon Cognito | $5.00 (100 MAU) |
| AWS Amplify | $1.31 (hosting) |
| Amazon CloudFront | $0.86 (CDN) |
| Amazon API Gateway | $0.01 (10k requests) |
| AWS Lambda | $0.00 (low usage) |
| Amazon S3 | $0.05 (2GB storage) |
| Amazon DynamoDB | $0.03 (on-demand) |
| Amazon CloudWatch | $1.64 (logs) |
| Amazon Route 53 | $0.90 (hosted zone) |

**Note:** AWS Free Tier significantly reduces or eliminates these costs for the first 12 months.

---

## Support & Resources

- **GitHub Repository:** [Online Library Workshop](https://github.com/your-repo/online-library-workshop)
- **AWS Documentation:** [docs.aws.amazon.com](https://docs.aws.amazon.com)
- **Community Support:** Workshop Q&A section
- **Troubleshooting Guide:** Available in each module

---

## Clean Up

⚠️ **Important:** Remember to clean up resources after completing the workshop to avoid ongoing charges. Detailed cleanup instructions are provided in Module 6.

---

**Ready to begin?** Start with [Module 1: Foundation & Authentication](5.1-module1/)