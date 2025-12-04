---
title: "Tuần 10 - Hoàn thành dự án Thư viện Online (Frontend)"
weight: 10
chapter: false
pre: "<b> 1.10. </b>"
---

**Tuần:** 10/11/2025 đến 14/11/2025  
**Trạng thái:** "Hoàn thành"  

## Tổng quan tuần 10

Tuần này tập trung vào hoàn thành triển khai frontend Thư viện Online, tích hợp với các dịch vụ AWS và hoàn thiện ứng dụng serverless.

## Chủ đề chính
- **Tích hợp xác thực**: Triển khai đăng nhập, đăng ký và quản lý người dùng dựa trên Cognito với Amplify UI
- **Phát triển luồng Upload**: Xây dựng giao diện tải sách lên với S3 presigned URLs và theo dõi trạng thái thời gian thực
- **Tạo bảng điều khiển Admin**: Phát triển quy trình duyệt admin, quản lý sách và các tính năng kiểm duyệt
- **Giao diện đọc sách**: Triển khai trải nghiệm đọc sách với CloudFront signed URLs và render PDF
- **Tìm kiếm & Khám phá**: Xây dựng chức năng tìm kiếm sử dụng DynamoDB GSI và tối ưu trải nghiệm người dùng
- **Kiểm thử & Triển khai**: Kiểm thử tích hợp cuối cùng, sửa lỗi và triển khai production qua Amplify CI/CD
- **Tối ưu hiệu suất**: Triển khai chiến lược caching và tối ưu thời gian tải
