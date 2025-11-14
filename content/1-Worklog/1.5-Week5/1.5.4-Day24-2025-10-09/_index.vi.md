---
title: "Ngày 24 - SCPs, Identity Center & KMS"
weight: 4
chapter: false
pre: "<b> 1.5.4. </b>"
---

**Ngày:** 2025-10-09 (Thứ Năm)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Service Control Policy (SCP)

- Xác định **quyền tối đa** cho tài khoản; chỉ giới hạn chứ không cấp quyền.  
- Áp dụng cho tài khoản hoặc OU; ảnh hưởng tất cả user/role, **kể cả root**; **Deny ghi đè Allow**.

**Ví dụ SCP (cấm xóa bucket):**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    { 
      "Effect": "Deny", 
      "Action": ["s3:DeleteBucket"], 
      "Resource": "*" 
    }
  ]
}
```

![image](/images/1-Worklog/Week5/image%2010.png)

**Tình huống dùng SCP:**

- Ngăn tài khoản rời khỏi organization.  
- Giới hạn region được phép tạo tài nguyên.  
- Ép buộc yêu cầu mã hóa.  
- Ngăn tắt các dịch vụ bảo mật.  
- Bắt buộc gắn tag nhất định cho tài nguyên.

**Best practice:**

- Bắt đầu với least privilege.  
- Thử nghiệm ở môi trường non-prod trước.  
- Dùng explicit deny cho kiểm soát quan trọng.  
- Ghi chú mục đích từng SCP.  
- Rà soát và cập nhật định kỳ.

---

## AWS Identity Center (trước là AWS SSO)

- Tập trung hóa truy cập vào tài khoản AWS và ứng dụng bên ngoài.  
- Nguồn danh tính: built-in, AWS Managed Microsoft AD, AD on-prem (trust/AD Connector), hoặc IdP ngoài.  
- **Permission Set** định nghĩa quyền cho user/group trên tài khoản đích (Identity Center tạo IAM role tương ứng). Có thể cấp nhiều permission set cho một người dùng.

![image](/images/1-Worklog/Week5/image%2011.png)

**Tính năng Identity Center:**

- Single sign-on cho nhiều tài khoản AWS.  
- Tích hợp Microsoft Active Directory.  
- Hỗ trợ SAML 2.0.  
- MFA.  
- Quản lý permission tập trung.  
- Ghi log audit qua CloudTrail.

---

## AWS Key Management Service (KMS)

- Dịch vụ quản lý khóa bảo vệ dữ liệu, tích hợp sâu với các dịch vụ AWS và hỗ trợ audit đầy đủ.

**Điểm nổi bật**

- Tạo/quản lý khóa mà không cần tự vận hành HSM.  
- Kiểm soát truy cập chi tiết với IAM & key policy; mọi thao tác được log trong CloudTrail.

**Các nhóm khóa**

- Khóa do khách hàng quản lý (CMK), khóa do AWS quản lý và khóa thuộc AWS-owned.

![image](/images/1-Worklog/Week5/image%2012.png)

**Loại khóa KMS:**

- **Symmetric:** Một khóa duy nhất để mã hóa/giải mã (AES-256).  
- **Asymmetric:** Cặp khóa public/private (RSA, ECC).

**Tính năng KMS:**

- Tự động xoay vòng khóa.  
- Key policy và grant.  
- Envelope encryption.  
- Tích hợp với dịch vụ AWS.  
- Log CloudTrail.  
- Khóa multi-region.

---

# **Hands-On Labs**

## Lab 33 – AWS KMS & CloudTrail Integration (Phần 1)

1. Tạo Policy và Role → *33-2.1*  
2. Tạo Group và User → *33-2.2*  
3. Tạo KMS Key → *33-3*  
4. Tạo S3 Bucket → *33-4.1*  
5. Upload dữ liệu lên S3 → *33-4.2*

## Lab 30 – IAM Restriction Policy

1. Tạo Restriction Policy → *30-3*  
2. Tạo IAM Limited User → *30-4*  
3. Kiểm tra giới hạn của IAM User → *30-5*  
4. Dọn dẹp tài nguyên → *30-6*
