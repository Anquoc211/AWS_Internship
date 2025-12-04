---
title: "Module 6: Triển khai & Vận hành"
date: "2025-01-15T14:00:00+07:00"
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

## Tổng quan

Trong module cuối cùng này, bạn sẽ thiết lập triển khai production với CI/CD, triển khai giám sát và cảnh báo, tối ưu chi phí, và áp dụng tăng cường bảo mật cho nền tảng Thư viện Online. Bạn cũng sẽ học các thực hành vận hành tốt nhất để chạy ứng dụng serverless ở quy mô lớn.

**Thời lượng:** ~90 phút

**Dịch vụ sử dụng:**
- AWS Amplify (CI/CD)
- Amazon CloudWatch (giám sát, alarms)
- AWS CloudFormation (cập nhật stack)
- AWS Budgets (cảnh báo chi phí)
- AWS X-Ray (tracing)

---

## Những gì bạn sẽ học

- Thiết lập CI/CD tự động với Amplify và CDK
- Cấu hình CloudWatch dashboards và alarms
- Triển khai giám sát và tối ưu chi phí
- Áp dụng thực hành bảo mật tốt nhất và hardening
- Thiết lập distributed tracing với X-Ray
- Tạo runbooks vận hành và tài liệu

---

## Xác minh & Kiểm thử

### Checklist

- CI/CD pipeline triển khai tự động khi push
- CloudWatch dashboard hiển thị tất cả metrics
- Alarms kích hoạt và gửi thông báo
- Budget alerts được cấu hình
- X-Ray traces xuất hiện trong console
- IAM roles tuân theo least privilege
- Tất cả dữ liệu được mã hóa at rest và in transit
- Runbooks được tài liệu hóa và kiểm thử

---

## Dọn dẹp

### Dọn dẹp tài nguyên hoàn chỉnh

```bash
# Xóa Amplify app
aws amplify delete-app --app-id YOUR_APP_ID

# Xóa CDK stacks
cdk destroy --all

# Xóa S3 buckets (nếu không tự động xóa)
aws s3 rb s3://your-bucket-name --force

# Xóa CloudWatch log groups
aws logs delete-log-group --log-group-name /aws/lambda/function-name

# Xác minh tất cả tài nguyên đã xóa
aws resourcegroupstaggingapi get-resources \
  --tag-filters Key=Project,Values=online-library
```

---

## Tóm tắt Workshop

Chúc mừng! Bạn đã hoàn thành Workshop Thư viện Online. Bạn đã xây dựng:

### Thành tựu kỹ thuật

**Hệ thống xác thực** - Cognito với xác thực email và JWT  
**Hạ tầng Upload** - Presigned URLs với theo dõi tiến trình  
**Quy trình duyệt Admin** - Kiểm soát truy cập dựa trên vai trò  
**Phân phối nội dung** - CloudFront với signed URLs và OAC  
**Chức năng tìm kiếm** - DynamoDB GSI với phân trang  
**CI/CD Pipeline** - Triển khai tự động  
**Giám sát & Cảnh báo** - CloudWatch dashboards và alarms  
**Tăng cường bảo mật** - Encryption, IAM, và WAF  

### Kỹ năng đạt được

- Thiết kế và triển khai kiến trúc serverless
- Infrastructure as code với AWS CDK
- Phát triển frontend với Next.js và React
- Thiết kế API với API Gateway và Lambda
- Thiết kế database với DynamoDB
- Phân phối nội dung với S3 và CloudFront
- Thực hành DevOps với CI/CD
- Vận hành xuất sắc với giám sát và cảnh báo

### Tóm tắt chi phí

**Chi phí vận hành hàng tháng (không tính Free Tier):** ~$9.80

Nền tảng có thể mở rộng đến:
- **5,000 người dùng:** ~$50/tháng
- **50,000 người dùng:** ~$200/tháng

### Bước tiếp theo

**Nâng cao nền tảng:**
1. Thêm validation định dạng file (MIME type checking)
2. Triển khai tự động xóa file sau 72 giờ
3. Thêm bookmark và theo dõi tiến trình đọc
4. Xây dựng hệ thống recommendation
5. Thêm tính năng bình luận và đánh giá
6. Triển khai analytics với QuickSight

**Học thêm:**
- AWS Well-Architected Framework
- DynamoDB patterns nâng cao
- Kỹ thuật tối ưu CloudFront
- Thực hành bảo mật tốt nhất cho serverless

---

## Tài nguyên bổ sung

- [AWS Serverless Application Model](https://aws.amazon.com/serverless/sam/)
- [AWS Well-Architected Framework](https://aws.amazon.com/architecture/well-architected/)
- [Serverless Patterns Collection](https://serverlessland.com/patterns)
- [AWS CDK Workshop](https://cdkworkshop.com/)
- [AWS Security Best Practices](https://docs.aws.amazon.com/security/)

---

**Cảm ơn bạn đã hoàn thành workshop này!** 🎉

Bạn đã xây dựng thành công một ứng dụng serverless sẵn sàng production trên AWS. Áp dụng các kỹ năng này vào dự án của riêng bạn và tiếp tục khám phá hệ sinh thái AWS.
