---
title: "Ngày 10 - Elastic Load Balancing"
weight: 5
chapter: false
pre: "<b> 1.2.5. </b>"
---

**Ngày:** 2025-09-19 (Thứ Sáu)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Elastic Load Balancing (ELB)

### Tổng quan

- Dịch vụ fully-managed phân phối lưu lượng tới nhiều target (EC2, container, v.v.).  
- Hỗ trợ giao thức HTTP, HTTPS, TCP, TLS.  
- Có thể triển khai ở subnet public hoặc private.  
- Cung cấp DNS name; chỉ Network Load Balancer hỗ trợ IP tĩnh.  
- Tích hợp health check và ghi log truy cập (lưu S3).  
- Hỗ trợ sticky session (session affinity).  
- Các loại chính: **Application**, **Network**, **Classic** và **Gateway** Load Balancer.

### Application Load Balancer (ALB)

- Hoạt động ở tầng 7 (HTTP/HTTPS).  
- Hỗ trợ định tuyến theo path (ví dụ /mobile vs /desktop).  
- Target: EC2, Lambda, địa chỉ IP, container (ECS/EKS).

![image](/images/1-Worklog/Week2/image%2017.png)

**Tính năng nổi bật của ALB:**

- Định tuyến theo host name.  
- Định tuyến theo path.  
- Định tuyến dựa trên HTTP header.  
- Định tuyến theo query string parameter.  
- Hỗ trợ WebSocket.  
- Hỗ trợ HTTP/2.

### Network Load Balancer (NLB)

- Hoạt động ở tầng 4 (TCP/TLS).  
- Hỗ trợ IP tĩnh, xử lý hàng triệu request/giây.  
- Target: EC2, địa chỉ IP, container (ECS/EKS).

**Điểm mạnh của NLB:**

- Độ trễ cực thấp.  
- Cung cấp địa chỉ IP tĩnh.  
- Giữ nguyên nguồn IP truy cập.  
- Hỗ trợ kết nối TCP dài hạn.  
- Có thể chấm dứt TLS (TLS termination).

### Gateway Load Balancer (GWLB)

- Hoạt động ở tầng 3 (gói IP).  
- Sử dụng giao thức GENEVE trên cổng 6081.  
- Định tuyến lưu lượng đến các virtual appliance như firewall, công cụ monitor.  
- Danh sách đối tác: [aws.amazon.com/elasticloadbalancing/partners](https://aws.amazon.com/elasticloadbalancing/partners)

![image](/images/1-Worklog/Week2/image%2018.png)

---

# **Khám phá**

## [AWS Advanced Networking – Specialty Study Guide](https://www.amazon.co.uk/Certified-Advanced-Networking-Official-Study/dp/1119439833)

- Sách hướng dẫn chính thức bao quát chủ đề kỳ thi, nguyên tắc thiết kế mạng trên AWS và các tình huống kiến trúc thực tế.

---

# **Hands-On Labs**

## Lab 20 – AWS Transit Gateway

1. Chuẩn bị môi trường → *20-02*  
2. Tạo Transit Gateway → *20-03*  
3. Tạo TGW Attachment → *20-04*  
4. Tạo TGW Route Table → *20-05*  
5. Thêm route TGW vào Route Table của VPC → *20-06*

---

## Tổng kết Tuần 2

Tuần này đã hoàn thành kiến thức về AWS Networking:

✅ Amazon VPC và Subnet  
✅ Security Group và NACL  
✅ VPC Peering và Transit Gateway  
✅ VPN và Direct Connect  
✅ Elastic Load Balancing (ALB, NLB, GWLB)  

**Labs đã hoàn tất:** 4 labs (VPC Basics, Hybrid DNS, VPC Peering, Transit Gateway)
