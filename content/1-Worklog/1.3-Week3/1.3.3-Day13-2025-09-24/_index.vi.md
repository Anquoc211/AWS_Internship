---
title: "Ngày 13 - Instance Store & User Data"
weight: 3
chapter: false
pre: "<b> 1.3.3. </b>"
---

**Ngày:** 2025-09-24 (Thứ Tư)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Tính năng nâng cao của EC2

### Instance Store

- **Instance Store** cung cấp lưu trữ block tạm thời gắn trực tiếp vào host EC2.

**Đặc điểm**

- I/O và thông lượng rất cao.  
- Dữ liệu mất khi instance dừng hoặc terminate.  
- Không thể tách rời hoặc tạo snapshot.

**Tình huống sử dụng**

- Cache hoặc xử lý dữ liệu tạm.  
- Ứng dụng đã có cơ chế nhân bản/replication riêng.

![image](/images/1-Worklog/Week3/image%204.png)

**So sánh Instance Store và EBS:**

| Tiêu chí | Instance Store | EBS |
|----------|----------------|-----|
| Tính bền vững | Tạm thời | Bền vững |
| Hiệu năng | Rất cao | Cao |
| Snapshot | Không | Có |
| Tháo rời | Không | Có |
| Chi phí | Đã bao gồm | Tính riêng |

---

### User Data

- Script **User Data** chạy tự động khi instance khởi tạo (mỗi lần provision AMI).  
- Linux dùng bash script, Windows dùng PowerShell.

![image](/images/1-Worklog/Week3/image%205.png)

**Ví dụ User Data:**

```bash
#!/bin/bash
yum update -y
yum install -y httpd
systemctl start httpd
systemctl enable httpd
echo "<h1>Hello from $(hostname -f)</h1>" > /var/www/html/index.html
```

---

### Metadata

- **EC2 Instance Metadata** cung cấp thông tin về instance đang chạy như IP private/public, hostname, security group.  
- Thường dùng trong User Data để cấu hình động.

![image](/images/1-Worklog/Week3/image%206.png)

**Truy cập Metadata:**

```bash
# Lấy instance ID
curl http://169.254.169.254/latest/meta-data/instance-id

# Lấy public IP
curl http://169.254.169.254/latest/meta-data/public-ipv4

# Lấy credential IAM role
curl http://169.254.169.254/latest/meta-data/iam/security-credentials/role-name
```

---

# **Hands-On Labs**

## Lab 07 – AWS Budgets & Cost Management

1. Tạo Budget từ template → *07-01*  
2. Hướng dẫn tạo Cost Budget → *07-02*  
3. Tạo Usage Budget → *07-03*  
4. Tạo Budget cho Reserved Instance → *07-04*  
5. Tạo Budget cho Savings Plans → *07-05*  
6. Dọn dẹp Budget → *07-06*
