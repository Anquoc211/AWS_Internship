---
title: "Ngày 20 - AWS Backup & FSx"
weight: 5
chapter: false
pre: "<b> 1.4.5. </b>"
---

**Ngày:** 2025-10-03 (Thứ Sáu)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## AWS Backup

- Dịch vụ backup tập trung giúp tự động hóa và quản trị bảo vệ dữ liệu ở quy mô lớn.

### Khả năng chính

- **Quản lý tập trung:** Định nghĩa và áp chính sách backup cho nhiều dịch vụ.  
- **Hỗ trợ đa dịch vụ:** EC2, EBS, RDS, DynamoDB, EFS, Storage Gateway, S3,...  
- **Lịch và vòng đời:** Tự động hóa lịch backup và retention.  
- **Tuân thủ:** Đáp ứng yêu cầu governance và audit.

### Lợi ích

- **Đơn giản vận hành:** Không cần script tùy biến hay công cụ rời rạc.  
- **Tiết kiệm thời gian:** Tự động bảo vệ dựa trên policy.  
- **Báo cáo & audit:** Theo dõi trạng thái backup và tuân thủ.

### Backup Vault Lock

- Cơ chế đảm bảo tính bất biến, ngăn chỉnh sửa/xóa backup đã bảo vệ nhằm đáp ứng yêu cầu tuân thủ nghiêm ngặt.

**Tính năng nổi bật của AWS Backup:**

- Sao chép backup liên vùng.  
- Backup chéo tài khoản.  
- Backup plan (chính sách) linh hoạt.
- Quản lý vòng đời (lưu trữ lạnh, xóa theo hạn).  
- Mã hóa dữ liệu khi lưu trữ.  
- Gắn thẻ để điều khiển chính sách backup theo tag.

**Ví dụ Backup Plan:**

```json
{
  "BackupPlanName": "DailyBackups",
  "Rules": [{
    "RuleName": "DailyRule",
    "ScheduleExpression": "cron(0 5 ? * * *)",
    "StartWindowMinutes": 60,
    "CompletionWindowMinutes": 120,
    "Lifecycle": {
      "DeleteAfterDays": 30,
      "MoveToColdStorageAfterDays": 7
    }
  }]
}
```

---

# **Khám phá**

## [AWS Skill Builder](https://skillbuilder.aws/?showRedirectNotFoundBanner=true)

- Các learning plan chọn lọc và nội dung chuyên sâu dành cho chuyên gia lưu trữ:  
  - Storage Learning Plan: Block Storage  
  - Storage Learning Plan: Object Storage

---

# **Hands-On Labs**

## Lab 13 – AWS Backup

1. Tạo S3 Bucket → *13-02.1*  
2. Triển khai hạ tầng mẫu → *13-02.2*  
3. Tạo Backup Plan → *13-03*  
4. Thiết lập thông báo → *13-04*  
5. Kiểm tra khôi phục → *13-05*  
6. Dọn dẹp tài nguyên → *13-06*

## Lab 25 – Amazon FSx (File Systems)

1. Tạo File System SSD Multi-AZ → *25-2.2*  
2. Tạo File System HDD Multi-AZ → *25-2.3*  
3. Tạo File Share mới → *25-3*  
4. Kiểm thử hiệu năng → *25-4*  
5. Giám sát hiệu năng → *25-5*  
6. Bật tính năng Data Deduplication → *25-6*  
7. Bật Shadow Copies → *25-7*  
8. Quản lý phiên người dùng & file mở → *25-8*  
9. Bật quota người dùng → *25-9*  
10. Scale thông lượng → *25-11*  
11. Mở rộng dung lượng lưu trữ → *25-12*  
12. Xóa môi trường → *25-13*

---

## Tổng kết Tuần 4

Tuần này đã hoàn tất các chủ đề về AWS Storage:

✅ Amazon S3 và các lớp lưu trữ  
✅ S3 Static Website và CORS  
✅ AWS Snow Family  
✅ AWS Storage Gateway  
✅ Chiến lược Disaster Recovery  
✅ AWS Backup  

**Labs đã hoàn tất:** 5 labs (Backup, VM Import/Export, Storage Gateway, FSx, S3 & CloudFront)
