---
title: "Ngày 07 - Định tuyến VPC & Network Interface"
weight: 2
chapter: false
pre: "<b> 1.2.2. </b>"
---

**Ngày:** 2025-09-16 (Thứ Ba)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Định tuyến VPC & ENI

### Route Table

- Route table xác định cách điều hướng lưu lượng mạng.  
- Mỗi VPC có một route table mặc định chỉ chứa tuyến local để các subnet giao tiếp nội bộ.  
- Có thể tạo thêm route table tùy chỉnh, nhưng tuyến local không thể xóa.

![image](/images/1-Worklog/Week2/f02d3571-7a7c-4388-a88f-87929790a60b.png)

### Elastic Network Interface (ENI)

- ENI là card mạng ảo có thể gắn sang các EC2 instance khác nhau.  
- Khi chuyển ENI, địa chỉ IP riêng, EIP và MAC được giữ nguyên.  
- Elastic IP (EIP) là địa chỉ IPv4 công cộng tĩnh có thể gắn vào ENI.  
- Bị tính phí nếu EIP không gắn cho tài nguyên nào.

![image](/images/1-Worklog/Week2/image%202.png)

**Tình huống sử dụng ENI:**

- Tách mạng quản trị khỏi mạng dữ liệu.  
- Xây dựng thiết bị mạng/bảo mật (appliance).  
- Instance hai cổng mạng chạy workload ở các subnet khác nhau.  
- Giải pháp high availability chi phí thấp.

### VPC Endpoint

- **VPC Endpoint** cho phép kết nối riêng tư tới dịch vụ AWS qua **AWS PrivateLink** mà không đi Internet công cộng.  
- Hai loại endpoint:  
  - **Interface Endpoint:** Tạo một ENI với IP riêng.  
  - **Gateway Endpoint:** Sử dụng route table (chỉ dành cho S3 và DynamoDB).

![image](/images/1-Worklog/Week2/image%203.png)

---

# **Hands-On Labs**

## Lab 03 – Amazon VPC & Networking (tiếp tục)

3. Tạo Internet Gateway (IGW) → *03-03.3*  
4. Tạo Route Table (Outbound qua IGW) → *03-03.4*  
5. Tạo Security Group → *03-03.5*
