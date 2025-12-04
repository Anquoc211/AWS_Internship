---
title: "Module 3: Hệ thống duyệt Admin"
date: "2025-01-15T11:00:00+07:00"
weight: 3
chapter: false
pre: " <b> 5.3. </b> "
---

## Tổng quan

Trong module này, bạn sẽ xây dựng quy trình duyệt admin hoàn chỉnh cho nền tảng Thư viện Online. Bạn sẽ triển khai kiểm soát truy cập dựa trên vai trò sử dụng Cognito User Groups, tạo Lambda functions cho logic duyệt/từ chối, và xây dựng giao diện dashboard admin.

**Thời lượng:** ~90 phút

**Dịch vụ sử dụng:**
- Amazon Cognito (User Groups)
- AWS Lambda (logic duyệt/từ chối)
- Amazon S3 (thao tác file)
- Amazon DynamoDB (cập nhật status)
- Amazon API Gateway (admin endpoints)

---

## Những gì bạn sẽ học

- Cấu hình Cognito User Groups cho truy cập dựa trên vai trò
- Triển khai Lambda functions chỉ dành cho admin
- Xây dựng thao tác copy file giữa các thư mục S3
- Cập nhật DynamoDB status với quy trình duyệt
- Tạo UI dashboard admin với các điều khiển duyệt
- Triển khai audit logging cho trách nhiệm giải trình

---

## Kiến trúc cho Module này

```
Admin User → Dashboard → API Gateway → Lambda (approveBook/rejectBook)
                                        ↓
                                  Kiểm tra Admin Role (JWT)
                                        ↓
                                  Copy S3: uploads/ → public/books/
                                        ↓
                                  Cập nhật DynamoDB (APPROVED/REJECTED)
                                        ↓
                                  Log vào CloudWatch
```

---

## Xác minh & Kiểm thử

### Checklist

- Cognito Admin group được tạo và users được gán
- Approval Lambda copies files chính xác
- Rejection Lambda moves files vào quarantine
- DynamoDB cập nhật với status và metadata đúng
- Admin dashboard hiển thị sách pending
- Approval modal hoạt động và cập nhật UI
- Rejection modal yêu cầu lý do
- Non-admin users không thể truy cập admin endpoints

### Các vấn đề thường gặp & Giải pháp

**Vấn đề:** JWT không chứa cognito:groups claim  
**Giải pháp:** Đảm bảo user được gán vào group và đăng nhập lại để nhận token mới

**Vấn đề:** S3 copy operation thất bại với lỗi permission  
**Giải pháp:** Xác minh Lambda có đúng IAM permissions cho cả hai buckets

**Vấn đề:** Admin dashboard hiển thị danh sách trống  
**Giải pháp:** Kiểm tra getPendingBooks Lambda queries StatusIndex GSI đúng cách

---

## Dọn dẹp (Tùy chọn)

```bash
cdk destroy
```

---

## Bước tiếp theo

Chúc mừng! Bạn đã hoàn thành Module 3. Bây giờ bạn có:
- Kiểm soát truy cập dựa trên vai trò với Cognito User Groups
- Lambda functions cho quy trình duyệt và từ chối
- Thao tác file S3 với permissions phù hợp
- Admin dashboard với điều khiển duyệt

**Tiếp tục đến:** [Module 4: Phân phối nội dung](../5.4-module4/)

---

## Tài nguyên bổ sung

- [Tài liệu Cognito User Groups](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-pools-user-groups.html)
- [S3 Copy Object Operations](https://docs.aws.amazon.com/AmazonS3/latest/userguide/copy-object.html)
- [DynamoDB Update Expressions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.UpdateExpressions.html)
