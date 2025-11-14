---
title: "Ngày 11 - Kiến thức cơ bản về Amazon EC2"
weight: 1
chapter: false
pre: "<b> 1.3.1. </b>"
---

**Ngày:** 2025-09-22 (Thứ Hai)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Compute trên AWS

### Amazon Elastic Compute Cloud (EC2)

- **Amazon EC2** cung cấp năng lực tính toán có thể co giãn trên cloud, tương tự máy chủ ảo hoặc vật lý.  
- Phù hợp cho các workload như web hosting, ứng dụng, cơ sở dữ liệu, dịch vụ xác thực và nhiều tác vụ máy chủ tổng quát khác.

**Instance Type**

- Cấu hình EC2 được xác định bởi **instance type**, không phải phần cứng tự chọn.  
- Mỗi loại quy định:  
  - CPU (Intel, AMD, ARM – Graviton 1/2/3) / GPU  
  - Bộ nhớ  
  - Kết nối mạng  
  - Lưu trữ

![image](/images/1-Worklog/Week3/image.png)

**Nhóm instance tiêu biểu:**

- **General Purpose:** T3, T4g, M5, M6i (cân bằng giữa compute, memory, network).  
- **Compute Optimized:** C5, C6i, C7g (hiệu năng xử lý cao).  
- **Memory Optimized:** R5, R6i, X2 (tối ưu cho workload dùng nhiều RAM).  
- **Storage Optimized:** I3, D2, H1 (tốc độ đọc/ghi tuần tự cao trên storage cục bộ).  
- **Accelerated Computing:** P4, G5, Inf1 (GPU/FPGA cho ML, đồ họa).

---

### Amazon Machine Image (AMI)

- **AMI (Amazon Machine Image)** là mẫu chứa cấu hình phần mềm của instance, gồm hệ điều hành, ứng dụng và thiết lập.  
- Các loại AMI:
  - AMI do AWS cung cấp (Amazon Linux, Windows, Ubuntu, v.v.).  
  - AMI trên AWS Marketplace.  
  - AMI tùy chỉnh do người dùng tạo.

**Lợi ích của AMI tùy chỉnh**

- Launch và cấu hình instance nhanh hơn.  
- Đơn giản hóa sao lưu và khôi phục.  
- Đảm bảo môi trường nhất quán trên nhiều instance.

**Thành phần AMI:**

- Mẫu ổ đĩa gốc (OS và ứng dụng).  
- Quyền launch.  
- Ánh xạ thiết bị khối (block device mapping).

---

# **Hands-On Labs**

## Lab 01 – AWS Account & IAM Setup

1. Tạo tài khoản AWS → *01-01*  
2. Cấu hình thiết bị MFA ảo → *01-02*  
3. Tạo nhóm Admin và người dùng Admin → *01-03*  
4. Cập nhật hỗ trợ xác thực tài khoản → *01-04*
