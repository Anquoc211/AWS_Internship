---
title: "Ngày 06 - Kiến thức Cơ bản về Amazon VPC"
weight: 1
chapter: false
pre: "<b> 1.2.1. </b>"
---

**Ngày:** 2025-09-15 (Thứ Hai)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Dịch vụ mạng trên AWS

### Amazon Virtual Private Cloud (VPC)

- Amazon VPC cho phép triển khai tài nguyên AWS vào một mạng ảo do chính bạn định nghĩa.

![image](/images/1-Worklog/Week2/c7387b7b-422f-41ee-a0b7-886017264e85.png)

- Mỗi VPC tồn tại trong phạm vi một Region. Khi tạo VPC, cần định nghĩa dải IPv4 CIDR (bắt buộc) và có thể thêm IPv6.  
- Giới hạn mặc định: 5 VPC mỗi Region cho mỗi tài khoản.  
- Thường dùng để tách biệt môi trường Production, Development, Staging.  
- Muốn cô lập hoàn toàn tài nguyên, nên dùng các AWS Account khác nhau thay vì nhiều VPC cùng tài khoản.

![image](/images/1-Worklog/Week2/image.png)

### Subnet

- Mỗi subnet nằm trong một Availability Zone.  
- CIDR của subnet phải là tập con của CIDR VPC cha.  
- AWS dành sẵn 5 địa chỉ IP trong mỗi subnet: network, broadcast, router, DNS và một địa chỉ dự phòng.

![image](/images/1-Worklog/Week2/image%201.png)

**Ví dụ các IP được dành trước (10.0.0.0/24):**

- 10.0.0.0 – Địa chỉ mạng  
- 10.0.0.1 – Router của VPC  
- 10.0.0.2 – Máy chủ DNS  
- 10.0.0.3 – Dành cho sử dụng trong tương lai  
- 10.0.0.255 – Địa chỉ broadcast

---

# **Hands-On Labs**

## Lab 03 – Amazon VPC & Networking Basics

1. Tạo VPC → *03-03.1*  
2. Tạo Subnet → *03-03.2*
