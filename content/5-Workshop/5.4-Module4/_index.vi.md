---
title: "Module 4: Phân phối nội dung với CloudFront"
date: "2025-01-15T12:00:00+07:00"
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

## Tổng quan

Trong module này, bạn sẽ triển khai phân phối nội dung an toàn sử dụng Amazon CloudFront với signed URLs. Bạn sẽ cấu hình Origin Access Control (OAC) để bảo vệ nội dung S3, tạo signed URLs có thời hạn cho truy cập được ủy quyền, và xây dựng giao diện đọc PDF/ePub.

**Thời lượng:** ~90 phút

**Dịch vụ sử dụng:**
- Amazon CloudFront (CDN với signed URLs)
- Amazon S3 (origin được bảo vệ)
- AWS Lambda (tạo signed URL)
- CloudFront Key Pairs (cho signing)

---

## Những gì bạn sẽ học

- Cấu hình CloudFront distribution với OAC
- Tạo CloudFront signed URLs từ Lambda
- Triển khai URL expiration và security policies
- Xây dựng PDF và ePub reader components
- Xử lý content streaming và caching

---

## Kiến trúc cho Module này

```
User → Yêu cầu sách → Lambda (getReadUrl)
                        ↓
                  Kiểm tra DynamoDB (APPROVED?)
                        ↓
                  Tạo CloudFront Signed URL (TTL 15 phút)
                        ↓
User ← Signed URL ← Response
     ↓
Truy cập nội dung qua CloudFront (với OAC tới S3)
```

---

## Xác minh & Kiểm thử

### Checklist

- CloudFront distribution được tạo với OAC
- S3 bucket policy chỉ cho phép CloudFront access
- Lambda tạo signed URLs hợp lệ
- Signed URLs hết hạn sau TTL
- PDF render chính xác trong browser
- Navigation controls hoạt động (prev/next page)
- Zoom functionality hoạt động
- Unauthorized users không thể truy cập nội dung

### Các vấn đề thường gặp & Giải pháp

**Vấn đề:** PDF không load với lỗi CORS  
**Giải pháp:** Xác minh CloudFront CORS settings và S3 bucket CORS configuration

**Vấn đề:** Signed URL hết hạn quá nhanh  
**Giải pháp:** Điều chỉnh TTL trong Lambda hoặc triển khai cơ chế refresh URL

**Vấn đề:** PDF.js worker không load  
**Giải pháp:** Đảm bảo PDF.js worker URL có thể truy cập và version khớp

---

## Dọn dẹp

```bash
cdk destroy
```

---

## Bước tiếp theo

Chúc mừng! Bạn đã hoàn thành Module 4. Bây giờ bạn có:
- Phân phối nội dung an toàn qua CloudFront
- Tạo Signed URL với expiration
- PDF reader với navigation và zoom
- Nội dung S3 được bảo vệ với OAC

**Tiếp tục đến:** [Module 5: Tìm kiếm & Khám phá](../5.5-module5/)

---

## Tài nguyên bổ sung

- [Tài liệu CloudFront Signed URLs](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-signed-urls.html)
- [Hướng dẫn CloudFront OAC](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- [Tài liệu React PDF](https://github.com/wojtekmaj/react-pdf)
