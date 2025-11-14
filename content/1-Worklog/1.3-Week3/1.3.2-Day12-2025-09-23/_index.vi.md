---
title: "Ngày 12 - Lưu trữ & Sao lưu cho EC2"
weight: 2
chapter: false
pre: "<b> 1.3.2. </b>"
---

**Ngày:** 2025-09-23 (Thứ Ba)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Lưu trữ & Bảo mật cho EC2

### Sao lưu trong EC2

- **AWS Backup** cung cấp giải pháp sao lưu tập trung cho nhiều dịch vụ, bao gồm EC2.  
- **EBS Snapshot** sao lưu các volume EBS:
  - Sao lưu theo thời điểm (point-in-time).  
  - Dạng incremental (chỉ lưu block thay đổi).  
  - Lưu trữ trong S3 (không truy cập trực tiếp).  
- **AMI Backup** chụp toàn bộ cấu hình EC2 dưới dạng image.

![image](/images/1-Worklog/Week3/image%201.png)

**Best practices cho Snapshot:**

- Lên lịch snapshot định kỳ.  
- Sao chép snapshot sang Region khác cho DR.  
- Gắn thẻ (tag) để quản lý vòng đời.  
- Sử dụng Amazon Data Lifecycle Manager (DLM).

---

### Key Pair

- **Key Pair** dùng để xác thực an toàn khi kết nối EC2:
  - **Public Key** – lưu trên instance.  
  - **Private Key** – người dùng giữ để SSH (Linux) hoặc RDP (Windows).  
- Thay thế mật khẩu, tăng cường bảo mật.  
- **Lưu ý:** Nếu mất private key, AWS không thể khôi phục.

![image](/images/1-Worklog/Week3/image%202.png)

**Quản lý Key Pair:**

- Tạo key pair trên AWS hoặc import key sẵn có.  
- Lưu trữ private key an toàn.  
- Dùng key pair khác nhau cho từng môi trường.  
- Luân phiên (rotate) định kỳ.

---

### Elastic Block Store (EBS)

- **Amazon EBS** cung cấp lưu trữ dạng block bền vững cho EC2.  
- Các loại volume:
  - **General Purpose SSD (gp2/gp3)** – cân bằng hiệu năng và chi phí.  
  - **Provisioned IOPS SSD (io1/io2)** – cho workload cần IOPS cao.  
  - **Throughput Optimized HDD (st1)** – dữ liệu lớn, truy cập tuần tự.  
  - **Cold HDD (sc1)** – dữ liệu ít truy cập, chi phí thấp.

**Tính năng chính:**

- Gắn/Tháo volume với instance.  
- Dữ liệu vẫn giữ khi instance tắt.  
- Tạo snapshot để sao lưu hoặc copy sang Region khác.  
- Tự động nhân bản trong phạm vi AZ.

![image](/images/1-Worklog/Week3/image%203.png)

**So sánh volume EBS:**

| Loại | Tình huống | Max IOPS | Max Throughput |
|------|------------|----------|----------------|
| gp3 | Mục đích tổng quát | 16.000 | 1.000 MB/s |
| io2 | Hiệu năng cao | 64.000 | 1.000 MB/s |
| st1 | Big data | 500 | 500 MB/s |
| sc1 | Lưu trữ lạnh | 250 | 250 MB/s |
