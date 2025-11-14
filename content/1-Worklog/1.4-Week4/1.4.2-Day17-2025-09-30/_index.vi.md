---
title: "Ngày 17 - Tính năng nâng cao của S3"
weight: 2
chapter: false
pre: "<b> 1.4.2. </b>"
---

**Ngày:** 2025-09-30 (Thứ Ba)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Hosting website tĩnh trên Amazon S3

Hosting trực tiếp website tĩnh (HTML, CSS, JS, hình ảnh) từ S3.

### Khả năng chính

- **Thiết lập đơn giản:** Chỉ vài bước để bật chế độ static website cho bucket.  
- **Chi phí thấp:** Trả phí lưu trữ và băng thông tiêu chuẩn, không cần máy chủ web riêng.  
- **Scale linh hoạt:** Tự động xử lý spike traffic.  
- **Tích hợp CDN:** Dễ dàng kết hợp Amazon CloudFront để tăng hiệu năng toàn cầu.

**Cấu hình website tĩnh:**

```json
{
  "IndexDocument": {
    "Suffix": "index.html"
  },
  "ErrorDocument": {
    "Key": "error.html"
  }
}
```

---

## Cross-Origin Resource Sharing (CORS)

CORS cho phép tài nguyên web (font, JavaScript, ...) trên một domain truy cập tài nguyên ở domain khác.

### Cấu hình CORS trên S3

- **Định nghĩa policy:** Chỉ rõ những origin nào được phép truy cập nội dung bucket.  
- **Kiểm soát method:** Cho phép các HTTP method cụ thể (GET, PUT, POST, ...).  
- **Tăng cường bảo mật:** Ngăn truy cập cross-origin trái phép.

**Ví dụ cấu hình CORS:**

```json
[
  {
    "AllowedHeaders": ["*"],
    "AllowedMethods": ["GET", "HEAD"],
    "AllowedOrigins": ["https://example.com"],
    "ExposeHeaders": ["ETag"],
    "MaxAgeSeconds": 3000
  }
]
```

---

## Hiệu năng & thiết kế khóa object

Cách đặt tên object ảnh hưởng đáng kể tới hiệu năng S3:

- **Prefix ngẫu nhiên:** Phân tán key qua nhiều partition để tăng song song.  
- **Tránh prefix tuần tự:** Không dùng tiền tố tăng dần (ví dụ timestamp) cho workload throughput cao.  
- **Truy cập song song:** Thiết kế key hỗ trợ đọc/ghi đồng thời.

**Best practice đặt key:**

```
❌ Tệ:  2025-09-30-file1.jpg, 2025-09-30-file2.jpg
✅ Tốt: a1b2/2025-09-30-file1.jpg, c3d4/2025-09-30-file2.jpg
```

---

## S3 Glacier – Lưu trữ dài hạn

Các lớp Glacier được tối ưu cho lưu trữ dài hạn chi phí thấp.

### Tùy chọn truy xuất

- **Expedited / Fast:** Vài phút; chi phí cao.  
- **Standard:** 3–5 giờ; cân bằng chi phí.  
- **Bulk:** 5–12 giờ; rẻ nhất cho khôi phục khối lượng lớn.

### Glacier Deep Archive

Lớp chi phí thấp nhất cho lưu trữ nhiều năm, thời gian truy xuất khoảng 12 giờ.

---

# **Hands-On Labs**

## Lab 57 – Amazon S3 & CloudFront (Phần 2)

5. Cấu hình object public → *57-5*  
6. Kiểm tra website → *57-6*  
7. Chặn toàn bộ public access → *57-7.1*  
8. Cấu hình CloudFront → *57-7.2*  
9. Kiểm tra CloudFront → *57-7.3*  
10. Bật Versioning cho bucket → *57-8*
