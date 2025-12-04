---
title: "Ngày 48 - Bảng điều khiển Admin & Quy trình duyệt"
weight: 3
chapter: false
pre: "<b> 1.10.3. </b>"
---

**Ngày:** 2025-11-12 (Thứ Tư)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú bài giảng**

## Kiến trúc bảng điều khiển Admin

### Kiểm soát truy cập dựa trên vai trò (RBAC)

**Triển khai Cognito Groups:**
- Admin users được gán vào nhóm `Admins` trong Cognito User Pool
- JWT token chứa claim `cognito:groups: ["Admins"]`
- Frontend kiểm tra user groups trước khi render admin routes
- API Gateway xác thực JWT, Lambda xác thực group membership

**Luồng phân quyền:**
```
Admin Login → Cognito trả JWT với groups → 
Frontend kiểm tra groups → Hiện/Ẩn admin navigation →
Admin action → API xác thực JWT + groups → Lambda thực thi
```

### Tính năng bảng điều khiển Admin

**Chức năng cốt lõi:**
1. **Xem xét sách Pending**: Liệt kê tất cả sách với trạng thái `PENDING`
2. **Hành động Duyệt/Từ chối**: Duyệt hoặc từ chối với lý do
3. **Quản lý sách**: Xem tất cả sách, tìm kiếm, lọc theo trạng thái
4. **Quản lý người dùng**: Xem uploaders, activity logs
5. **Phân tích**: Thống kê upload, tỷ lệ duyệt, sử dụng lưu trữ

**Quy trình trạng thái:**
- `PENDING` → Trạng thái upload ban đầu
- `VALIDATING` → Đang xác thực MIME server-side
- `APPROVED` → Admin duyệt, sách có thể truy cập công khai
- `REJECTED` → Admin từ chối với lý do
- `TAKEDOWN` → Nội dung bị gỡ do vi phạm bản quyền

## Triển khai quy trình duyệt

### Luồng Backend (Lambda)

**approveBook Lambda:**
```
1. Xác thực JWT và kiểm tra nhóm Admins
2. Xác minh sách tồn tại và trạng thái là PENDING
3. Copy file từ uploads/ sang public/books/
4. Cập nhật DynamoDB: status=APPROVED, approverID, approvalTimestamp
5. Xóa file gốc từ uploads/
6. Trả response thành công
```

**rejectBook Lambda:**
```
1. Xác thực JWT và kiểm tra nhóm Admins
2. Cập nhật DynamoDB: status=REJECTED, reason, rejectionTimestamp
3. Tùy chọn: Di chuyển file sang quarantine/ thay vì xóa
4. Gửi thông báo cho uploader (cải tiến tương lai)
```

### Giao diện Admin Frontend

**Danh sách sách Pending:**
- Hiển thị: thumbnail, tiêu đề, tác giả, uploader, ngày upload
- Hành động: Xem trước (download tạm), Duyệt, Từ chối
- Bộ lọc: theo ngày, theo uploader, theo loại file
- Phân trang cho tập dữ liệu lớn

**Modal duyệt:**
- Xác nhận hành động với chi tiết sách
- Tùy chọn: thêm ghi chú duyệt
- Trạng thái loading trong quá trình xử lý
- Thông báo thành công/lỗi

**Modal từ chối:**
- Bắt buộc chọn/nhập lý do từ chối
- Lý do định sẵn: bản quyền, không phù hợp, chất lượng, khác
- Trường message tùy chỉnh
- Xác nhận trước khi gửi

## Những hiểu biết quan trọng

- Bảng điều khiển admin là khu vực có đặc quyền cao - kiểm soát truy cập nghiêm ngặt thiết yếu
- Audit logging quan trọng cho trách nhiệm giải trình (ai duyệt/từ chối cái gì và khi nào)
- Soft delete (quarantine) tốt hơn hard delete cho tuân thủ pháp lý
- Cập nhật trạng thái thời gian thực cải thiện hiệu quả quy trình admin
- Lý do từ chối rõ ràng giúp uploaders cải thiện submissions tương lai

---

# **Công việc đã hoàn thành**

1. **Bảo vệ Route Admin**
   - Tạo wrapper route chỉ dành cho admin kiểm tra Cognito groups
   - Triển khai xử lý truy cập không được phép (chuyển hướng đến dashboard)
   - Thêm menu điều hướng admin (chỉ hiển thị cho admins)
   - Xây dựng trang "Access Denied" cho non-admin users

2. **Giao diện sách Pending**
   - Tạo component danh sách sách pending với DataTable
   - Triển khai modal xem trước sách (hiển thị metadata và bìa)
   - Thêm bulk selection cho batch operations
   - Xây dựng bộ lọc trạng thái (PENDING, VALIDATING, ALL)

3. **Hệ thống duyệt**
   - Tạo modal xác nhận duyệt
   - Triển khai tích hợp API `approveBook`
   - Thêm cập nhật UI optimistic (thay đổi trạng thái ngay lập tức)
   - Xây dựng toast notifications thành công/lỗi

4. **Hệ thống từ chối**
   - Tạo modal từ chối với chọn lý do
   - Xây dựng dropdown lý do từ chối (định sẵn + tùy chỉnh)
   - Triển khai tích hợp API `rejectBook`
   - Thêm xem lịch sử từ chối cho uploaders

5. **Bảng điều khiển Admin**
   - Tạo tổng quan dashboard với thẻ thống kê
   - Xây dựng biểu đồ: uploads theo thời gian, tỷ lệ duyệt, phân phối trạng thái
   - Triển khai feed hoạt động gần đây
   - Thêm nút hành động nhanh (badge số lượng pending)

6. **Hiển thị Audit Logging**
   - Tạo audit log viewer hiển thị tất cả hành động
   - Triển khai lọc theo loại hành động, khoảng ngày, admin
   - Thêm chức năng xuất audit logs sang CSV
   - Xây dựng xem chi tiết hành động với metadata

---

# **Thách thức & Giải pháp**

**Thách thức:** Cập nhật trạng thái thời gian thực mà không polling  
**Giải pháp:** Triển khai kết nối WebSocket cho admin panel để nhận thay đổi trạng thái tức thì từ backend

**Thách thức:** Danh sách lớn sách pending gây vấn đề hiệu suất  
**Giải pháp:** Triển khai virtual scrolling và phân trang server-side với cursor-based navigation

**Thách thức:** Duyệt/từ chối nhầm lẫn  
**Giải pháp:** Thêm modal xác nhận với đếm ngược 2 giây trước khi bật hành động

**Thách thức:** Hành động admin không phản ánh ngay lập tức  
**Giải pháp:** Triển khai cập nhật UI optimistic với rollback khi lỗi, plus cache invalidation
