---
title: "Ngày 23 - Amazon Cognito & Organizations"
weight: 3
chapter: false
pre: "<b> 1.5.3. </b>"
---

**Ngày:** 2025-10-08 (Thứ Tư)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Amazon Cognito

- Dịch vụ managed cho xác thực/ủy quyền và quản lý người dùng cho web & mobile.  
- Thành phần:
  - **User Pools:** Thư mục người dùng phục vụ đăng ký/đăng nhập.  
  - **Identity Pools:** Danh tính liên kết (federated) cung cấp credential tạm thời để truy cập dịch vụ AWS.

![image](/images/1-Worklog/Week5/image%206.png)
![image](/images/1-Worklog/Week5/image%207.png)
![image](/images/1-Worklog/Week5/image%208.png)
![image](/images/1-Worklog/Week5/image%209.png)

**Tính năng của Cognito User Pools:**

- Đăng ký và đăng nhập.  
- Hỗ trợ IdP xã hội (Google, Facebook, Amazon).  
- Hỗ trợ IdP SAML.  
- Multi-factor authentication (MFA).  
- Xác thực email và số điện thoại.  
- Tùy biến luồng đăng nhập.  
- Lambda trigger để mở rộng logic.

**Tính năng của Cognito Identity Pools:**

- Credential AWS tạm thời.  
- Phân biệt truy cập authenticated và unauthenticated.  
- Kiểm soát truy cập dựa trên role.  
- Tích hợp với User Pool.  
- Hỗ trợ IdP bên ngoài.

---

## AWS Organizations

- Quản lý tập trung nhiều tài khoản AWS trong một tổ chức.

**Lợi ích**

- Quản lý tài khoản tập trung.  
- **Consolidated Billing**.  
- Cấu trúc phân cấp bằng **Organizational Unit (OU)**.  
- Thiết lập guardrail bằng **Service Control Policy (SCP)**.

### Organizational Unit (OU)

- Gom tài khoản theo phòng ban, dự án hoặc môi trường; có thể lồng OUs để áp policy theo tầng.

**Ví dụ cấu trúc OU:**

```
Root
├── Production OU
│   ├── Web App Account
│   └── Database Account
├── Development OU
│   ├── Dev Account
│   └── Test Account
└── Security OU
    └── Audit Account
```

### Consolidated Billing

- Một hóa đơn cho mọi tài khoản; hưởng lợi từ volume pricing; không phát sinh phí thêm.

**Lợi ích:**

- Giảm giá theo khối lượng dùng chung giữa các tài khoản.  
- Dễ theo dõi và lập báo cáo.  
- Đơn giản hóa phương thức thanh toán.  
- Chia sẻ Reserved Instance.

---

# **Hands-On Labs**

## Lab 28 – IAM Cross-Region Role & Policy (Phần 2)

5. Switch Role → *28-5.1*  
6. Truy cập EC2 Console – Tokyo → *28-5.2.1*  
7. Truy cập EC2 Console – N. Virginia → *28-5.2.2*  
8. Tạo EC2 (không đáp ứng tag) → *28-5.2.3*  
9. Chỉnh sửa tag tài nguyên EC2 → *28-5.2.4*  
10. Kiểm tra chính sách → *28-5.2.5*

## Lab 27 – AWS Resource Groups & Tagging (Phần 1)

1. Tạo EC2 Instance kèm Tag → *27-2.1.1*  
2. Quản lý tag cho tài nguyên AWS → *27-2.1.2*  
3. Lọc tài nguyên theo tag → *27-2.1.3*
