---
title: "Ngày 15 - Lightsail, EFS & FSx"
weight: 5
chapter: false
pre: "<b> 1.3.5. </b>"
---

**Ngày:** 2025-09-26 (Thứ Sáu)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Amazon Lightsail

- Dịch vụ compute đơn giản với giá cố định hàng tháng (bắt đầu ~3,5 USD/tháng).  
- Bao gồm băng thông đi kèm với giá thấp hơn EC2.  
- Lý tưởng cho workload nhỏ, môi trường dev/test.  
- Hỗ trợ snapshot để sao lưu.  
- Chạy trong VPC được quản lý và có thể kết nối VPC tiêu chuẩn qua peering (một lần nhấp).

**Trường hợp dùng Lightsail:**

- Ứng dụng web đơn giản.  
- Trang WordPress.  
- Môi trường phát triển/thử nghiệm.  
- Ứng dụng doanh nghiệp nhỏ.  
- Học tập và thử nghiệm.

**So sánh Lightsail và EC2:**

| Tiêu chí | Lightsail | EC2 |
|----------|-----------|-----|
| Giá | Cố định hàng tháng | Trả theo dùng |
| Độ phức tạp | Đơn giản | Nhiều tùy chọn |
| Khả năng mở rộng | Giới hạn | Không giới hạn |
| Đối tượng | Dự án nhỏ | Doanh nghiệp |

---

## Amazon EFS (Elastic File System)

- Dịch vụ hệ thống file NFSv4 do AWS quản lý, nhiều EC2 có thể mount đồng thời.  
- Tự động scale tới hàng petabyte.  
- Trả tiền theo dung lượng thực tế sử dụng (khác với EBS phải provision).  
- Có thể mount từ on-prem thông qua VPN hoặc Direct Connect.

**Tính năng EFS:**

- Truy cập đồng thời từ nhiều instance.  
- Tự động mở rộng.  
- Dịch vụ cấp độ Region (đa AZ).  
- Quản lý vòng đời.  
- Mã hóa dữ liệu khi lưu trữ và truyền tải.

**Các lớp lưu trữ EFS:**

- **Standard:** File truy cập thường xuyên.  
- **Infrequent Access (IA):** Chi phí thấp cho file ít truy cập.  
- **One Zone:** Một AZ để tiết kiệm chi phí.

---

## Amazon FSx

- Các hệ thống file được quản lý, mở rộng cho Windows, Lustre, NetApp ONTAP.  
- AWS lo phần thiết lập, mở rộng, sao lưu.  
- Truy cập từ EC2, máy chủ on-prem hoặc người dùng qua giao thức SMB/NFS.

**Các biến thể FSx:**

### FSx for Windows File Server

- Hệ thống file Windows gốc.  
- Hỗ trợ giao thức SMB.  
- Tích hợp Active Directory.  
- Hỗ trợ DFS namespace.

### FSx for Lustre

- Phù hợp workload HPC.  
- Machine Learning, mô phỏng.  
- Độ trễ dưới mili giây.  
- Tích hợp S3.

### FSx for NetApp ONTAP

- Hỗ trợ đa giao thức (NFS, SMB, iSCSI).  
- Giảm trùng lặp dữ liệu, nén.  
- Snapshots và replication.

---

## AWS Application Migration Service (MGN)

- Dịch vụ migrate/replicate máy chủ vật lý hoặc ảo lên AWS để DR hoặc hiện đại hóa.  
- Liên tục sao chép máy nguồn sang instance staging EC2 nhẹ.  
- Khi cut-over, MGN tạo EC2 đầy đủ chức năng từ dữ liệu đã replicate.

**Các giai đoạn migration:**

1. **Cài agent** lên máy nguồn.  
2. **Sao chép liên tục** vào AWS.  
3. **Kiểm thử** bằng instance test không ảnh hưởng.  
4. **Cutover** sang production.

---

# **Khám phá**

## [Microsoft Workloads on AWS](https://youtube.com/playlist?list=PLhr1KZpdzukdJllxulUM7pMB7aJ2_FfTP&si=HMwzvhqsFFljKfnC)

- Playlist tuyển chọn về triển khai, tối ưu và best practices khi chạy workload Microsoft trên AWS.

---

## Tổng kết Tuần 3

Tuần này đã hoàn thành kiến thức về AWS Compute:

✅ Amazon EC2 và các loại instance  
✅ AMI, EBS, Instance Store  
✅ EC2 Auto Scaling  
✅ Các mô hình giá của EC2  
✅ Lightsail, EFS, FSx  

**Labs đã hoàn tất:** 3 labs (IAM Setup, Budgets, Support Plans)
