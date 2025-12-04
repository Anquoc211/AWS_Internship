---
title: "Day 30 - Database Migration & Best Practices"
weight: 5
chapter: false
pre: "<b> 1.6.5. </b>"
---

**Date:** 2025-10-17 (Thứ Sáu)  
**Status:** "Done"  

---

# **Lecture Notes**

## AWS Database Migration Service (DMS)

AWS DMS helps migrate databases to AWS quickly and securely with minimal downtime.

**Key Features:**

- **Homogeneous Migrations:** Same database engine (e.g., Oracle to Oracle)
- **Heterogeneous Migrations:** Different engines (e.g., Oracle to Aurora)
- **Continuous Replication:** Keep source and target in sync
- **Schema Conversion:** AWS Schema Conversion Tool (SCT)

**Migration Types:**

- **Full Load:** One-time migration of existing data
- **Full Load + CDC:** Initial load plus ongoing changes
- **CDC Only:** Replicate only ongoing changes

**Supported Sources:**

- Oracle, SQL Server, MySQL, PostgreSQL, MongoDB, SAP ASE, IBM Db2
- Amazon RDS, Amazon Aurora, Amazon S3

**Supported Targets:**

- Amazon RDS, Amazon Aurora, Amazon Redshift, Amazon DynamoDB
- Amazon S3, Amazon Elasticsearch, Amazon Kinesis Data Streams

---

## Database Best Practices

### Performance Optimization

**RDS/Aurora:**

- Use appropriate instance types
- Enable Enhanced Monitoring
- Optimize queries and indexes
- Use Read Replicas for read-heavy workloads
- Enable Performance Insights

**Redshift:**

- Choose appropriate distribution keys
- Use sort keys for frequently filtered columns
- Vacuum and analyze tables regularly
- Use columnar compression
- Implement workload management (WLM)

**ElastiCache:**

- Choose appropriate node types
- Use cluster mode for Redis scalability
- Implement proper cache eviction policies
- Monitor cache hit rates
- Use connection pooling

---

### Security Best Practices

- **Encryption at Rest:** Enable for all databases
- **Encryption in Transit:** Use SSL/TLS connections
- **Network Isolation:** Deploy in private subnets
- **IAM Authentication:** Use for RDS/Aurora when possible
- **Secrets Manager:** Store database credentials securely
- **Security Groups:** Restrict access to minimum required
- **Audit Logging:** Enable CloudWatch Logs and CloudTrail

---

### High Availability & Disaster Recovery

**RDS/Aurora:**

- Enable Multi-AZ for production workloads
- Configure automated backups
- Test restore procedures regularly
- Use Aurora Global Database for multi-region DR
- Implement read replicas in different regions

**Redshift:**

- Enable automated snapshots
- Copy snapshots to other regions
- Use Redshift Spectrum for data lake integration
- Implement cross-region snapshot copy

**ElastiCache:**

- Enable Multi-AZ with automatic failover (Redis)
- Configure backup and restore (Redis)
- Use cluster mode for Redis scalability
- Implement application-level retry logic

---

### Cost Optimization

- **Right-sizing:** Choose appropriate instance types
- **Reserved Instances:** Commit for 1-3 years for discounts
- **Aurora Serverless:** For variable workloads
- **Redshift Serverless:** For intermittent analytics
- **Storage Optimization:** Use appropriate storage types
- **Lifecycle Policies:** Archive old data to S3/Glacier
- **Monitor Usage:** Use Cost Explorer and Budgets

---

# **Exploration**

## [The Data Warehouse Toolkit](https://www.amazon.com/Data-Warehouse-Toolkit-Definitive-Dimensional/dp/1118530802)

- Canonical reference for dimensional modeling and DW design patterns.

---

## Week 6 Summary

Tuần này đã hoàn thành kiến thức về AWS Database Services:

Database Fundamentals (RDBMS, NoSQL, OLTP vs OLAP)  
Amazon RDS & Aurora  
Amazon Redshift  
Amazon ElastiCache  
AWS Database Migration Service  

**Labs completed:** 2 labs (RDS & EC2 Integration, Database Migration Service)

---

## Tổng kết 6 tuần đầu (8/9 - 17/10/2025)

**30 ngày làm việc đã hoàn thành:**

### Week 1: Cloud Computing Fundamentals
- AWS basics, infrastructure, management tools, cost optimization

### Week 2: AWS Networking Services  
- VPC, subnets, security groups, load balancing, hybrid connectivity

### Week 3: AWS Compute Services
- EC2, AMI, storage, auto scaling, pricing models

### Week 4: AWS Storage Services
- S3, Glacier, Snow Family, Storage Gateway, backup & DR

### Week 5: AWS Security & Identity
- IAM, Cognito, Organizations, KMS, Security Hub

### Week 6: AWS Database Services
- RDS, Aurora, Redshift, ElastiCache, DMS

**Tổng số labs hoàn thành:** 25+ labs

**Tiếp theo:** Tuần 7-8 sẽ bắt đầu từ ngày 20/10/2025 (Thứ Hai)
