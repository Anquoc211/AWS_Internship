---
title: "Ngày 16 - Kiến thức cơ bản về Amazon S3"
weight: 1
chapter: false
pre: "<b> 1.4.1. </b>"
---

**Ngày:** 2025-09-29 (Thứ Hai)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Dịch vụ lưu trữ trên AWS

### Amazon Simple Storage Service (S3)

Amazon S3 là dịch vụ lưu trữ đối tượng, cho phép lưu và truy xuất dữ liệu với quy mô gần như không giới hạn, độ sẵn sàng cao, bảo mật mạnh và hiệu năng tốt.

### Tính năng cốt lõi của S3

- **Bucket và Object:** Dữ liệu được lưu dưới dạng object trong bucket. Mỗi object tối đa 5 TB.  
- **Availability & Durability:** Thiết kế đạt 99,99% availability và 99,999999999% (11 số 9) durability.  
- **Bảo mật:** Nhiều lớp bảo vệ như IAM, bucket policy, ACL, mã hóa.  
- **Khả năng mở rộng:** Tự động scale dung lượng và throughput mà không giảm hiệu năng.

**Cấu trúc object S3:**

- **Key:** Tên/đường dẫn object.  
- **Value:** Dữ liệu object.  
- **Version ID:** Dùng khi bật versioning.  
- **Metadata:** Metadata hệ thống và người dùng.  
- **Access Control:** Quyền truy cập.

### S3 Access Points

Access Point giúp đơn giản hóa việc quản lý truy cập cho dataset dùng chung.

- **Kiểm soát theo ứng dụng:** Mỗi access point có policy riêng.  
- **Đơn giản vận hành:** Dễ quản lý quyền cho dataset dùng chung nhiều ứng dụng.  
- **Kiểm soát mạng:** Có thể cấu hình chỉ cho phép truy cập từ các VPC cụ thể.

---

### Các lớp lưu trữ S3

Chọn storage class phù hợp với mô hình truy cập và chi phí:

- **S3 Standard:** Dữ liệu truy cập thường xuyên; availability và hiệu năng cao nhất.  
- **S3 Intelligent-Tiering:** Tự động di chuyển object giữa các tier để tối ưu chi phí.  
- **S3 Standard-IA:** Dữ liệu ít truy cập, vẫn truy xuất mili-giây.  
- **S3 One Zone-IA:** Tương tự Standard-IA nhưng lưu ở một AZ.  
- **S3 Glacier Flexible Retrieval:** Lưu trữ chi phí thấp, truy xuất phút–giờ.  
- **S3 Glacier Deep Archive:** Chi phí thấp nhất, truy xuất ~12 giờ.

**So sánh storage class:**

| Class | Độ bền | Availability | Lưu tối thiểu | Thời gian truy xuất |
|-------|--------|--------------|---------------|---------------------|
| Standard | 11 số 9 | 99,99% | Không | Tức thời |
| Intelligent-Tiering | 11 số 9 | 99,9% | Không | Tức thời |
| Standard-IA | 11 số 9 | 99,9% | 30 ngày | Tức thời |
| One Zone-IA | 11 số 9 | 99,5% | 30 ngày | Tức thời |
| Glacier Flexible | 11 số 9 | 99,99% | 90 ngày | Phút-giờ |
| Glacier Deep Archive | 11 số 9 | 99,99% | 180 ngày | 12 giờ |

---

# **Hands-On Labs**

## Lab 57 – Amazon S3 & CloudFront (Phần 1)

1. Tạo S3 Bucket → *57-2.1*  
2. Tải dữ liệu lên → *57-2.2*  
3. Bật Static Website → *57-3*  
4. Cấu hình Public Access Block → *57-4*
