---
title: "Workshop: Xây dựng Thư viện Online với AWS Serverless"
date: "2025-01-15T09:00:00+07:00"
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

# Xây dựng nền tảng Thư viện Online Serverless

## Tổng quan Workshop

Trong workshop thực hành này, bạn sẽ xây dựng một nền tảng thư viện online serverless hoàn chỉnh sử dụng các dịch vụ AWS. Ứng dụng có tính năng xác thực người dùng, tải file lên với quy trình duyệt của admin, phân phối nội dung an toàn qua CloudFront và chức năng tìm kiếm.

**Thời lượng:** 6 modules (~6 giờ)

**Chi phí:** ~$10/tháng (không tính AWS Free Tier)

**Độ khó:** Trung cấp

---

## Những gì bạn sẽ xây dựng

Một nền tảng thư viện online sẵn sàng production với:

- **Xác thực người dùng** với Amazon Cognito
- **Hệ thống tải file lên** sử dụng S3 Presigned URLs
- **Quy trình duyệt Admin** với kiểm soát truy cập dựa trên vai trò
- **Phân phối nội dung an toàn** qua CloudFront Signed URLs
- **Chức năng tìm kiếm** sử dụng DynamoDB GSI
- **CI/CD Pipeline** với AWS Amplify

---

## Kiến trúc

![Sơ đồ kiến trúc](/images/2-Proposal/Architect.jpeg)

Giải pháp sử dụng kiến trúc serverless hoàn toàn:

- **Frontend:** Next.js host trên AWS Amplify
- **Xác thực:** Amazon Cognito User Pools
- **API:** Amazon API Gateway (HTTP API)
- **Tính toán:** AWS Lambda (Python)
- **Lưu trữ:** Amazon S3 với CloudFront CDN
- **Cơ sở dữ liệu:** Amazon DynamoDB
- **Hạ tầng:** AWS CDK (TypeScript)

---

## Yêu cầu trước khi bắt đầu

Trước khi bắt đầu workshop này, bạn cần có:

- **Tài khoản AWS** với quyền quản trị
- **AWS CLI** đã cài đặt và cấu hình
- **Node.js 18+** và npm/yarn
- **Python 3.9+** đã cài đặt
- **Git** cho quản lý version
- **Kiến thức cơ bản** về:
  - JavaScript/TypeScript
  - Python
  - REST APIs
  - Các dịch vụ AWS cơ bản

---

## Các Module Workshop

### [Module 1: Nền tảng & Xác thực](5.1-module1/)
- Thiết lập dự án AWS CDK
- Triển khai Cognito User Pool
- Cấu hình luồng xác thực
- Xây dựng trang đăng nhập/đăng ký

### [Module 2: Hạ tầng Upload](5.2-module2/)
- Tạo S3 buckets với policies phù hợp
- Triển khai tạo presigned URL
- Xây dựng form upload với theo dõi tiến trình
- Lưu metadata trong DynamoDB

### [Module 3: Hệ thống duyệt Admin](5.3-module3/)
- Thiết lập nhóm người dùng admin
- Tạo Lambda functions duyệt
- Xây dựng UI dashboard admin
- Triển khai theo dõi trạng thái

### [Module 4: Phân phối nội dung](5.4-module4/)
- Cấu hình CloudFront distribution
- Triển khai Origin Access Control (OAC)
- Tạo CloudFront Signed URLs
- Xây dựng giao diện đọc PDF/ePub

### [Module 5: Tìm kiếm & Khám phá](5.5-module5/)
- Thiết kế DynamoDB GSI cho tìm kiếm
- Tạo Lambda function tìm kiếm
- Xây dựng UI tìm kiếm với bộ lọc
- Triển khai phân trang

### [Module 6: Triển khai & Vận hành](5.6-module6/)
- Thiết lập CI/CD với Amplify
- Cấu hình giám sát và cảnh báo
- Triển khai tối ưu chi phí
- Tăng cường bảo mật

---

## Mục tiêu học tập

Sau khi hoàn thành workshop này, bạn sẽ:

✅ Hiểu các pattern kiến trúc serverless  
✅ Triển khai xác thực an toàn với Cognito  
✅ Xử lý tải file lên quy mô lớn với S3  
✅ Xây dựng quy trình duyệt với Lambda  
✅ Phân phối nội dung an toàn qua CloudFront  
✅ Thiết kế tìm kiếm hiệu quả với DynamoDB  
✅ Triển khai với tự động hóa CI/CD  
✅ Giám sát và tối ưu chi phí  

---

## Ước tính chi phí

**Chi phí hàng tháng (không tính Free Tier):** ~$9.80

| Dịch vụ | Chi phí ước tính |
|---------|------------------|
| Amazon Cognito | $5.00 (100 MAU) |
| AWS Amplify | $1.31 (hosting) |
| Amazon CloudFront | $0.86 (CDN) |
| Amazon API Gateway | $0.01 (10k requests) |
| AWS Lambda | $0.00 (sử dụng thấp) |
| Amazon S3 | $0.05 (2GB storage) |
| Amazon DynamoDB | $0.03 (on-demand) |
| Amazon CloudWatch | $1.64 (logs) |
| Amazon Route 53 | $0.90 (hosted zone) |

**Lưu ý:** AWS Free Tier giảm đáng kể hoặc loại bỏ các chi phí này trong 12 tháng đầu tiên.

---

## Hỗ trợ & Tài nguyên

- **GitHub Repository:** [Online Library Workshop](https://github.com/your-repo/online-library-workshop)
- **Tài liệu AWS:** [docs.aws.amazon.com](https://docs.aws.amazon.com)
- **Hỗ trợ cộng đồng:** Mục Q&A của workshop
- **Hướng dẫn khắc phục sự cố:** Có sẵn trong mỗi module

---

## Dọn dẹp

⚠️ **Quan trọng:** Nhớ dọn dẹp tài nguyên sau khi hoàn thành workshop để tránh phát sinh chi phí. Hướng dẫn dọn dẹp chi tiết được cung cấp trong Module 6.

---

**Sẵn sàng bắt đầu?** Bắt đầu với [Module 1: Nền tảng & Xác thực](5.1-module1/)