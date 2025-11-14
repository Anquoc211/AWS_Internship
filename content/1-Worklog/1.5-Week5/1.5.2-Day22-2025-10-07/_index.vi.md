---
title: "Ngày 22 - IAM Policies & Roles"
weight: 2
chapter: false
pre: "<b> 1.5.2. </b>"
---

**Ngày:** 2025-10-07 (Thứ Ba)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## IAM Policies

- Policy IAM là tài liệu JSON mô tả quyền.  
- Các loại:
  - **Identity-based policy** (gắn vào principal).  
  - **Resource-based policy** (gắn vào resource).
- Quy tắc đánh giá: mọi **explicit Deny** sẽ ghi đè Allow trên tất cả policy.

Ví dụ ràng buộc admin S3:

- Cho phép toàn bộ `s3:*` trên một bucket cụ thể.  
- Explicit **Deny** mọi hành động không phải S3.

![image](/images/1-Worklog/Week5/image%203.png)

**Cấu trúc policy:**

```json
{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Action": "s3:GetObject",
    "Resource": "arn:aws:s3:::my-bucket/*",
    "Condition": {
      "IpAddress": {
        "aws:SourceIp": "203.0.113.0/24"
      }
    }
  }]
}
```

**Logic đánh giá policy:**

1. Mặc định mọi request bị từ chối.  
2. Allow rõ ràng sẽ ghi đè deny mặc định.  
3. Deny rõ ràng ghi đè mọi Allow.  
4. Permissions boundary giới hạn quyền tối đa.

---

## IAM Roles

- Role cung cấp **quyền tạm thời** cho user, dịch vụ hoặc danh tính bên ngoài.  
- Các tình huống phổ biến:
  - Cho phép dịch vụ AWS hành động thay bạn (ví dụ EC2 ghi vào S3).  
  - Truy cập chéo tài khoản.  
  - Liên kết danh tính từ IdP ngoài (federation).  
  - Cấp credential cho ứng dụng trên EC2 mà không cần lưu access key.

![image](/images/1-Worklog/Week5/image%204.png)

**Lợi ích**

- Không có credential dài hạn, phiên ngắn, hỗ trợ nguyên tắc least privilege và quản lý truy cập quy mô lớn.

![image](/images/1-Worklog/Week5/image%205.png)

**Các loại role:**

- **Service Role:** Cho dịch vụ AWS (EC2, Lambda, ...).  
- **Cross-Account Role:** Truy cập tài nguyên ở tài khoản khác.  
- **Identity Provider Role:** Cho người dùng liên kết (federated).  
- **Instance Profile:** Vỏ chứa role dành cho EC2 instance.

---

# **Hands-On Labs**

## Lab 48 – IAM Access Keys & Roles (Phần 2)

4. Sử dụng Access Key → *48-2.2*  
5. Tạo IAM Role → *48-3.1*  
6. Gán IAM Role → *48-3.2*  
7. Dọn dẹp tài nguyên → *48-4*

## Lab 28 – IAM Cross-Region Role & Policy (Phần 1)

1. Tạo IAM User → *28-2.1*  
2. Tạo IAM Policy → *28-3*  
3. Tạo IAM Role → *28-4*
