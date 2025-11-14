---
title: "Ngày 21 - Shared Responsibility & Kiến thức IAM cơ bản"
weight: 1
chapter: false
pre: "<b> 1.5.1. </b>"
---

**Ngày:** 2025-10-06 (Thứ Hai)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Bảo mật

### Mô hình Trách nhiệm chia sẻ (Shared Responsibility Model)

- Trên môi trường cloud, bảo mật là **trách nhiệm được chia sẻ** giữa nhà cung cấp và khách hàng.  
- Khách hàng cần cấu hình dịch vụ an toàn, áp dụng best practice và triển khai các kiểm soát bảo mật từ lớp hypervisor trở lên (ứng dụng/dữ liệu).

![image](/images/1-Worklog/Week5/image.png)

- Phạm vi trách nhiệm thay đổi tùy mô hình dịch vụ:  
  - Dịch vụ cấp hạ tầng  
  - Dịch vụ được quản lý một phần  
  - Dịch vụ fully-managed

![image](/images/1-Worklog/Week5/image%201.png)

**Trách nhiệm của AWS (Security OF the Cloud):**

- Bảo mật vật lý trung tâm dữ liệu.  
- Hạ tầng phần cứng và mạng.  
- Lớp ảo hóa.  
- Vận hành các dịch vụ managed.

**Trách nhiệm của khách hàng (Security IN the Cloud):**

- Mã hóa dữ liệu.  
- Cấu hình mạng.  
- Quản lý truy cập.  
- Bảo mật ứng dụng.  
- Vá OS (đối với EC2).

---

## AWS Identity and Access Management (IAM)

### Root Account

- Có quyền không giới hạn với mọi dịch vụ/tài nguyên AWS và có thể gỡ mọi quyền đã cấp.  
- Best practice:  
  - Tạo và dùng **IAM Administrator** cho tác vụ hằng ngày.  
  - Lưu trữ thông tin root an toàn (dual control).  
  - Duy trì email và domain của root hợp lệ, được gia hạn.  
  - Bật MFA cho tài khoản root.

### Tổng quan IAM

- IAM kiểm soát quyền truy cập dịch vụ/tài nguyên trong tài khoản.  
- **Principal** bao gồm: root user, IAM user, federated user, IAM role, phiên assumed-role, dịch vụ AWS và người dùng ẩn danh.  
- Ghi nhớ:  
  - IAM user không phải là tài khoản AWS riêng.  
  - Người dùng mới tạo **không có quyền** mặc định.  
  - Cấp quyền bằng cách gắn **policy** vào user, group hoặc role.  
  - Dùng **IAM group** để quản lý nhiều user (group không lồng nhau được).

![image](/images/1-Worklog/Week5/image%202.png)

---

# **Hands-On Labs**

## Lab 48 – IAM Access Keys & Roles (Phần 1)

1. Tạo EC2 Instance → *48-1.1*  
2. Tạo S3 Bucket → *48-1.2*  
3. Tạo IAM User và Access Key → *48-2.1*
