---
title: "Ngày 08 - Bảo mật VPC & Flow Logs"
weight: 3
chapter: false
pre: "<b> 1.2.3. </b>"
---

**Ngày:** 2025-09-17 (Thứ Tư)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Bảo mật VPC

### Security Group (SG)

- Tường lửa ảo **stateful** kiểm soát lưu lượng vào/ra tài nguyên AWS.  
- Quy tắc dựa trên giao thức, cổng, nguồn hoặc security group khác.  
- Chỉ hỗ trợ rule cho phép (allow).  
- Áp dụng ở cấp độ Elastic Network Interface (ENI).

![image](/images/1-Worklog/Week2/image%207.png)
![image](/images/1-Worklog/Week2/image%208.png)

**Đặc điểm của Security Group:**

- Stateful: lưu lượng trả về được cho phép tự động.  
- Chỉ có rule cho phép.  
- Đánh giá toàn bộ rule trước khi quyết định.  
- Áp dụng ở mức instance/ENI.

### Network Access Control List (NACL)

- Tường lửa ảo **stateless** hoạt động ở cấp subnet.  
- Quy tắc điều khiển lưu lượng vào/ra theo giao thức, cổng và nguồn.  
- NACL mặc định cho phép tất cả lưu lượng.

![image](/images/1-Worklog/Week2/image%209.png)
![image](/images/1-Worklog/Week2/image%2010.png)

**Đặc điểm của NACL:**

- Stateless: phải cho phép rõ ràng lưu lượng chiều về.  
- Hỗ trợ cả rule allow và deny.  
- Các rule được xử lý theo thứ tự số.  
- Áp dụng ở mức subnet.

### VPC Flow Logs

- Ghi nhận metadata về lưu lượng IP đi/đến các network interface trong VPC.  
- Log có thể gửi tới Amazon CloudWatch Logs hoặc S3.  
- Flow Logs không ghi phần payload của gói tin.

![image](/images/1-Worklog/Week2/image%2011.png)

**Trường hợp sử dụng:**

- Khắc phục sự cố kết nối.  
- Theo dõi mẫu lưu lượng.  
- Phân tích bảo mật.  
- Đáp ứng yêu cầu tuân thủ.

---

# **Hands-On Labs**

## Lab 03 – Amazon VPC & Networking (tiếp)

6. Khởi chạy EC2 trong các subnet → *04-1*  
7. Kiểm tra kết nối giữa các instance → *04-2*  
8. Tạo NAT Gateway (Private ↔ Internet) → *04-3*  
9. EC2 Instance Connect Endpoint (không cần bastion) → *04-5*
