---
title: "Nhật ký học tập - Hành trình AWS"
weight: 1
chapter: true
pre: "<b> 1. </b>"
---

# Nhật ký học tập

## Tổng quan

Đây là nhật ký ghi chép hành trình học AWS và phát triển phần mềm từ ngày 8 tháng 9, 2025 đến ngày 14 tháng 11, 2025 (50 ngày làm việc trong 10 tuần).

### Cấu trúc

Nhật ký được tổ chức theo **tuần**, mỗi tuần gồm **5 ngày làm việc** (Thứ Hai đến Thứ Sáu).

### Tiến độ

- **Tuần 1** (8-12/9): Các khái niệm cơ bản về Cloud Computing
- **Tuần 2** (15-19/9): Dịch vụ Mạng AWS
- **Tuần 3** (22-26/9): Dịch vụ Tính toán AWS
- **Tuần 4** (29/9-3/10): Dịch vụ Lưu trữ AWS
- **Tuần 5** (6-10/10): Bảo mật & Quản lý Danh tính AWS
- **Tuần 6** (13-17/10): Dịch vụ Cơ sở dữ liệu AWS
- **Tuần 7** (20-24/10): Chủ đề AWS Nâng cao
- **Tuần 8** (27-31/10): Học Xử lý Ngôn ngữ Tự nhiên
- **Tuần 9** (3-7/11): Ôn tập NLP & Các khái niệm Nâng cao
- **Tuần 10** (10-14/11): Hoàn thiện Dự án Thư viện Trực tuyến (Frontend)

### Thống kê

- **Tổng thời gian:** 10 tuần (50 ngày làm việc)
- **Các tuần tập trung AWS:** 7 tuần (Tuần 1-7)
- **Các tuần phát triển Front-End:** 3 tuần (Tuần 8-10)
- **Tổng số labs:** Hơn 25 bài lab thực hành AWS
- **Lĩnh vực chính:** Dịch vụ AWS Cốt lõi, NLP, Phát triển Full-Stack

### Nội dung chính

#### Dịch vụ AWS Cốt lõi (Tuần 1-7)

1. **Các khái niệm cơ bản về Cloud Computing**
   - Kiến thức cơ bản về AWS, hạ tầng toàn cầu, công cụ quản lý
   - Tối ưu hóa chi phí, gói hỗ trợ
   - Well-Architected Framework

2. **Mạng**
   - VPC, subnets, security groups, NACLs
   - Cân bằng tải (ALB, NLB, GWLB)
   - VPC Peering, Transit Gateway
   - Hybrid DNS với Route 53 Resolver

3. **Tính toán**
   - EC2, AMI, EBS, Instance Store
   - Auto Scaling, các mô hình định giá
   - Lightsail, EFS, FSx

4. **Lưu trữ**
   - S3, các lớp lưu trữ, Glacier
   - Snow Family, Storage Gateway
   - Khôi phục thảm họa, AWS Backup

5. **Bảo mật & Quản lý Danh tính**
   - IAM, Cognito, Organizations
   - KMS, Security Hub
   - Identity Center (SSO)
   - Mô hình Trách nhiệm Chia sẻ

6. **Cơ sở dữ liệu**
   - RDS, Aurora, Redshift
   - ElastiCache, DMS
   - Các thực hành tốt nhất về cơ sở dữ liệu

7. **Chủ đề Nâng cao**
   - Serverless (Lambda)
   - Containers (ECS, EKS, ECR)
   - Giám sát (CloudWatch, X-Ray, CloudTrail)
   - Tự động hóa Lambda với tích hợp Slack

#### Xử lý Ngôn ngữ Tự nhiên (Tuần 8-9)

8. **Kiến thức Cơ bản & Ứng dụng NLP**
   - Tiền xử lý văn bản (tokenization, stemming, lemmatization)
   - Phân tích cảm xúc và phân loại văn bản
   - Nhận dạng thực thể có tên (NER)
   - Thư viện NLP (NLTK, spaCy)
   - Xây dựng pipeline NLP và các dự án thực tế

#### Phát triển Front-End (Tuần 10)

9. **Dự án Thư viện Trực tuyến**
   - Xác thực với AWS Cognito và Amplify UI
   - Tải file lên với S3 presigned URLs
   - Bảng quản trị và hệ thống quản lý sách
   - Giao diện đọc sách với CloudFront signed URLs
   - Chức năng tìm kiếm sử dụng DynamoDB GSI
   - Triển khai CI/CD với AWS Amplify
