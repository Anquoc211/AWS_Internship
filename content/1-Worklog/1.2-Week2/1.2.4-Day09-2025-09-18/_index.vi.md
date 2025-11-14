---
title: "Ngày 09 - Kết nối VPC & Cân bằng tải"
weight: 4
chapter: false
pre: "<b> 1.2.4. </b>"
---

**Ngày:** 2025-09-18 (Thứ Năm)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## VPC Peering & Transit Gateway

### VPC Peering

- Cho phép kết nối riêng tư trực tiếp giữa hai VPC mà không qua Internet.  
- Không hỗ trợ định tuyến chuyển tiếp (transitive) và không chấp nhận CIDR trùng nhau.

![image](/images/1-Worklog/Week2/image%2012.png)
![image](/images/1-Worklog/Week2/image%2013.png)
![image](/images/1-Worklog/Week2/image%2014.png)

**Hạn chế của VPC Peering:**

- Không có peering chuyển tiếp.  
- Không được phép trùng CIDR.  
- Mỗi VPC tối đa 125 kết nối peering.  
- Hỗ trợ peering liên vùng (cross-region).

### AWS Transit Gateway (TGW)

- Hoạt động như một hub kết nối nhiều VPC và mạng on-premises, đơn giản hóa kiến trúc mesh phức tạp.  
- TGW Attachment gắn các subnet thuộc một AZ cụ thể vào TGW.  
- Các subnet trong cùng AZ có thể truy cập TGW sau khi được attach.

![image](/images/1-Worklog/Week2/image%2015.png)
![image](/images/1-Worklog/Week2/image%2016.png)

**Lợi ích của Transit Gateway:**

- Hub kết nối tập trung.  
- Đơn giản hóa kiến trúc mạng.  
- Mở rộng tới hàng nghìn VPC.  
- Hỗ trợ peering giữa các region.

---

## VPN & Direct Connect

### Site-to-Site VPN

- Thiết lập kết nối IPSec bảo mật giữa data center on-premises và AWS VPC.  
- Bao gồm:  
  - **Virtual Private Gateway (VGW):** endpoint đa AZ do AWS quản lý.  
  - **Customer Gateway (CGW):** thiết bị hoặc appliance do khách hàng quản lý.

### AWS Direct Connect

- Cung cấp đường truyền riêng giữa data center on-prem và AWS.  
- Độ trễ điển hình: 20–30 ms.  
- Tại Việt Nam hiện có thông qua Hosted Connection (đối tác).  
- Có thể điều chỉnh băng thông linh hoạt.

---

# **Hands-On Labs**

## Lab 10 – Hybrid DNS (Route 53 Resolver)

1. Tạo Key Pair → *10-02.1*  
2. Khởi tạo CloudFormation Template → *10-02.2*  
3. Cấu hình Security Group → *10-02.3*  
4. Thiết lập hệ thống DNS → *10-05*  
5. Tạo Route 53 Outbound Endpoint → *10-05.1*  
6. Tạo Resolver Rules → *10-05.2*  
7. Tạo Inbound Endpoints → *10-05.3*

## Lab 19 – VPC Peering

1. Khởi tạo CloudFormation Templates → *19-02.1*  
2. Tạo Security Group → *19-02.2*  
3. Tạo EC2 instance (test peering) → *19-02.3*  
4. Tạo kết nối Peering → *19-04*  
5. Cấu hình Route Table (Cross-VPC) → *19-05*  
6. Bật Cross-Peer DNS → *19-06*
