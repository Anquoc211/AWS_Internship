---
title: "Ngày 02 - Hạ tầng Toàn cầu của AWS"
weight: 2
chapter: false
pre: "<b> 1.1.2. </b>"
---

**Ngày:** 2025-09-09 (Thứ Ba)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Hạ tầng AWS

### Trung tâm dữ liệu (Data Center)

- Mỗi trung tâm dữ liệu có thể chứa hàng chục nghìn máy chủ.  
- AWS tự thiết kế và vận hành phần cứng riêng để tối ưu hiệu năng và độ tin cậy.

### Vùng khả dụng (Availability Zone - AZ)

- Một hoặc nhiều trung tâm dữ liệu tách biệt vật lý trong cùng một Region.  
- Mỗi AZ được thiết kế cách ly lỗi.  
- Kết nối với nhau bằng mạng riêng độ trễ thấp, băng thông cao.  
- AWS khuyến nghị triển khai workload tối thiểu trên hai AZ.

### Region

- Mỗi Region chứa ít nhất ba AZ.  
- Hiện có hơn 25 Region trên toàn thế giới.  
- Các Region kết nối với nhau qua mạng backbone của AWS.  
- Phần lớn dịch vụ mặc định ở phạm vi Region.

### Edge Location

- Mạng lưới edge toàn cầu giúp phân phối nội dung với độ trễ tối thiểu.  
- Được sử dụng bởi các dịch vụ như:  
  - Amazon CloudFront (CDN)  
  - AWS WAF (Tường lửa ứng dụng web)  
  - Amazon Route 53 (Dịch vụ DNS)

---

# **Hands-On Labs**

### Lab 01 – Thiết lập Tài khoản AWS & IAM

1. Tạo tài khoản AWS → *01-01*  
2. Cấu hình thiết bị MFA ảo → *01-02*  
3. Tạo nhóm Admin và người dùng Admin → *01-03*  
4. Cập nhật thông tin hỗ trợ xác thực tài khoản → *01-04*
