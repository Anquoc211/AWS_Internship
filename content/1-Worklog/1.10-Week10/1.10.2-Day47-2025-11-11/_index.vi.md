---
title: "Ngày 47 - Luồng tải sách lên & Presigned URLs"
weight: 2
chapter: false
pre: "<b> 1.10.2. </b>"
---

**Ngày:** 2025-11-11 (Thứ Ba)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú bài giảng**

## Kiến trúc luồng Upload

### Chiến lược S3 Presigned URL

**Tại sao dùng Presigned URLs?**
- Upload trực tiếp lên S3 tránh giới hạn payload API Gateway (10MB)
- Giảm thời gian thực thi và chi phí Lambda
- Trải nghiệm người dùng tốt hơn với theo dõi tiến trình
- Kiến trúc Serverless tự động mở rộng

**Luồng Upload:**
```
User → Yêu cầu Upload URL (API) → Lambda tạo Presigned PUT URL → 
User upload trực tiếp lên S3 → S3 Event Notification → Lambda cập nhật DynamoDB
```

### Cân nhắc bảo mật

**Cấu hình Presigned URL:**
- Thời gian hết hạn ngắn (15 phút cho upload)
- Giới hạn kích thước file qua điều kiện `content-length-range`
- Giới hạn MIME types cụ thể (application/pdf, application/epub+zip)
- Upload vào prefix tạm `uploads/`, không phải thư mục public

**Chiến lược xác thực:**
- Client-side: kiểm tra kích thước và loại file trước khi yêu cầu URL
- Server-side: Lambda xác thực metadata trước khi tạo URL
- S3 Event: Lambda sau upload xác thực MIME type thực tế bằng magic bytes
- Theo dõi trạng thái: PENDING → VALIDATING → APPROVED/REJECTED

## Triển khai Upload Frontend

### Quy trình Upload nhiều bước

**Bước 1: Chọn & Xác thực File**
- Giao diện kéo thả hoặc chọn file
- Xác thực client-side (kích thước ≤50MB, loại PDF/ePub)
- Xem trước metadata file (tên, kích thước, loại)

**Bước 2: Thu thập Metadata**
- Các trường form: tiêu đề, tác giả, mô tả, ảnh bìa
- Chọn thể loại/danh mục
- Tùy chọn: ISBN, năm xuất bản, ngôn ngữ

**Bước 3: Yêu cầu Upload URL**
- POST đến endpoint API `/upload` với metadata
- Nhận presigned URL và book ID
- Lưu book ID để theo dõi trạng thái

**Bước 4: Upload trực tiếp S3**
- PUT request đến presigned URL với nội dung file
- Theo dõi tiến trình upload với XMLHttpRequest hoặc fetch
- Hiển thị thanh tiến trình và thời gian ước tính còn lại

**Bước 5: Giám sát trạng thái**
- Poll endpoint `/books/{id}/status`
- Hiển thị trạng thái hiện tại (PENDING, VALIDATING, APPROVED, REJECTED)
- Hiển thị lý do từ chối nếu có

## Những hiểu biết quan trọng

- Presigned URLs là credentials có thời hạn - xử lý hết hạn một cách graceful
- Luôn xác thực cả client và server để ngăn uploads độc hại
- Theo dõi tiến trình cải thiện đáng kể trải nghiệm người dùng với files lớn
- S3 Event Notifications cho phép xác thực async mà không cần polling
- Thông báo trạng thái rõ ràng giúp người dùng hiểu quy trình upload

---

# **Công việc đã hoàn thành**

1. **UI Components Upload**
   - Tạo component upload file với kéo thả
   - Xây dựng form metadata với validation (React Hook Form + Zod)
   - Triển khai chỉ báo tiến trình upload với phần trăm
   - Thêm xác thực loại file và kích thước

2. **Tích hợp API**
   - Tạo phương thức API client cho luồng upload
   - Triển khai API call `createUploadUrl` với metadata
   - Xây dựng upload trực tiếp S3 với theo dõi tiến trình
   - Thêm xử lý lỗi cho network failures

3. **Theo dõi trạng thái**
   - Tạo cơ chế polling trạng thái với intervals
   - Xây dựng component status badge (PENDING, APPROVED, REJECTED)
   - Triển khai cập nhật trạng thái thời gian thực
   - Thêm hệ thống thông báo cho thay đổi trạng thái

4. **Trải nghiệm người dùng**
   - Thiết kế wizard upload với form nhiều bước
   - Thêm xem trước file trước khi upload
   - Triển khai thông báo thành công/lỗi
   - Tạo trang "My Uploads" hiển thị tất cả uploads của người dùng

5. **Xử lý lỗi**
   - Xử lý hết hạn presigned URL
   - Triển khai logic retry cho uploads thất bại
   - Thêm thông báo lỗi validation
   - Tạo UI fallback cho lỗi upload

---

# **Thách thức & Giải pháp**

**Thách thức:** Upload files lớn bị timeout  
**Giải pháp:** Triển khai chunked upload với retry logic cho chunks thất bại

**Thách thức:** Presigned URL hết hạn trong quá trình upload chậm  
**Giải pháp:** Yêu cầu URL mới nếu upload mất hơn 10 phút

**Thách thức:** Người dùng đóng browser trong quá trình upload  
**Giải pháp:** Thêm theo dõi localStorage để resume uploads và hiển thị cảnh báo trước khi unload trang

**Thách thức:** Theo dõi tiến trình không chính xác  
**Giải pháp:** Sử dụng XMLHttpRequest upload.onprogress event để theo dõi chính xác ở cấp độ byte
