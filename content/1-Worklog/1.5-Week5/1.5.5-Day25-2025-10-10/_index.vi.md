---
title: "Ngày 25 - AWS Security Hub & Automation"
weight: 5
chapter: false
pre: "<b> 1.5.5. </b>"
---

**Ngày:** 2025-10-10 (Thứ Sáu)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## AWS Security Hub

- Tổng hợp và ưu tiên hóa các phát hiện bảo mật, posture across account/dịch vụ.

**Khả năng**

- Kiểm tra tự động, chuẩn hóa findings, workflow xử lý ưu tiên.  
- Hỗ trợ chuẩn tuân thủ: CIS AWS Foundations, PCI DSS, AWS Foundational Security Best Practices.

**Tích hợp**

- GuardDuty, Inspector, Macie, Firewall Manager, IAM Access Analyzer và nhiều công cụ đối tác.

**Kết quả**

- Giảm thời gian thu thập, tập trung khắc phục; cái nhìn hợp nhất và nâng cao hygiene bảo mật.

![image](/images/1-Worklog/Week5/image%2013.png)

**Tính năng Security Hub:**

- Theo dõi posture bảo mật liên tục.  
- Kiểm tra tuân thủ tự động.  
- Tổng hợp findings đa tài khoản.  
- Tích hợp 50+ dịch vụ AWS & đối tác.  
- Custom insight và dashboard.  
- Tự động khắc phục qua EventBridge.

**Chuẩn bảo mật được hỗ trợ:**

- **AWS Foundational Security Best Practices:** hơn 50 control.  
- **CIS AWS Foundations Benchmark:** chuẩn ngành.  
- **PCI DSS:** tiêu chuẩn thẻ thanh toán.  
- **NIST:** khung bảo mật NIST.

---

## Security Automation

**Dịch vụ AWS phục vụ tự động hóa:**

- **AWS Config:** Theo dõi thay đổi cấu hình tài nguyên.  
- **Amazon EventBridge:** Tự động hóa dựa trên sự kiện.  
- **AWS Lambda:** Hàm serverless xử lý remediation.  
- **AWS Systems Manager:** Tự động vá lỗi và quản lý tuân thủ.

**Mẫu tự động hóa phổ biến:**

- Tự động khắc phục tài nguyên không tuân thủ.  
- Ứng phó sự cố tự động.  
- Kiểm tra quy tắc security group.  
- Cưỡng chế mã hóa.  
- Đảm bảo tuân thủ tag.

---

# **Khám phá**

## [AWS Certified Security – Specialty: All-in-One Exam Guide (SCS-C01)](https://dokumen.pub/aws-certified-security-specialty-all-in-one-exam-guide-exam-scs-c01-1260461734-9781260461725-1260461726.html)

- Tài liệu ôn luyện toàn diện cho chứng chỉ Security Specialty.

---

# **Hands-On Labs**

## Lab 18 – AWS Security Hub

1. Bật Security Hub → *18-02*  
2. Đánh giá từng bộ tiêu chí → *18-03*  
3. Dọn dẹp tài nguyên → *18-04*

## Lab 22 – AWS Lambda Automation with Slack

1. Tạo VPC → *22-2.1*  
2. Tạo Security Group → *22-2.2*  
3. Tạo EC2 Instance → *22-2.3*  
4. Cấu hình Slack Incoming Webhook → *22-2.4*  
5. Tạo tag cho instance → *22-3*  
6. Tạo role cho Lambda → *22-4*  
7. Hàm Stop Instance → *22-5.1*  
8. Hàm Start Instance → *22-5.2*  
9. Kiểm tra kết quả → *22-6*  
10. Dọn dẹp → *22-7*

## Lab 27 – AWS Resource Groups & Tagging (Phần 2)

4. Sử dụng tag với CLI → *27-2.2*  
5. Tạo Resource Group → *27-3*  
6. Dọn dẹp tài nguyên → *27-4*

## Lab 33 – AWS KMS & CloudTrail Integration (Phần 2)

6. Tạo CloudTrail → *33-5.1*  
7. Ghi log vào CloudTrail → *33-5.2*  
8. Tạo Amazon Athena → *33-5.3*  
9. Query bằng Athena → *33-5.4*  
10. Kiểm tra & chia sẻ dữ liệu S3 đã mã hóa → *33-6*  
11. Dọn dẹp → *33-7*

## Lab 44 – IAM Advanced Role Control

1. Tạo IAM Group → *44-2*  
2. Tạo IAM User → *44-3.1*  
3. Kiểm tra quyền → *44-3.2*  
4. Tạo IAM Role Admin → *44-4.1*  
5. Cấu hình Switch Role → *44-4.2*  
6. Giới hạn Switch Role theo IP → *44-4.3.1*  
7. Giới hạn Switch Role theo thời gian → *44-4.3.2*  
8. Dọn dẹp → *44-5*

---

## Tổng kết Tuần 5

Tuần này đã hoàn tất các chủ đề về AWS Security:

✅ Shared Responsibility Model  
✅ AWS IAM (Users, Groups, Roles, Policies)  
✅ Amazon Cognito  
✅ AWS Organizations & SCPs  
✅ AWS Identity Center  
✅ AWS KMS  
✅ AWS Security Hub  

**Labs đã hoàn tất:** 8 labs (Security Hub, Lambda Automation, Resource Groups, IAM Policies, KMS & CloudTrail, Advanced Role Control)
