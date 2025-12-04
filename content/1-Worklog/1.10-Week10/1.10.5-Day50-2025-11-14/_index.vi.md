---
title: "Ngày 50 - Kiểm thử, Triển khai & Hoàn thành dự án"
weight: 5
chapter: false
pre: "<b> 1.10.5. </b>"
---

**Ngày:** 2025-11-14 (Thứ Sáu)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú bài giảng**

## Chiến lược kiểm thử

### Cách tiếp cận kiểm thử Frontend

**Unit Testing:**
- Kiểm thử các components riêng lẻ
- Mock API calls và dependencies bên ngoài
- Tập trung vào business logic và tương tác người dùng
- Công cụ: Jest, React Testing Library

**Integration Testing:**
- Kiểm thử tương tác components và luồng dữ liệu
- Xác thực tích hợp API với mock servers
- Kiểm thử luồng xác thực end-to-end
- Công cụ: Cypress, Playwright

**E2E Testing:**
- Mô phỏng quy trình người dùng thực tế
- Kiểm thử các đường dẫn quan trọng (upload, approve, read)
- Xác thực hành vi môi trường production
- Công cụ: Cypress E2E, manual QA

### Checklist kiểm thử

**Luồng xác thực:**
- ✓ Người dùng có thể đăng ký với email
- ✓ Xác thực email hoạt động chính xác
- ✓ Người dùng có thể đăng nhập và nhận JWT
- ✓ Protected routes chuyển hướng đến login
- ✓ Phát hiện vai trò admin hoạt động
- ✓ Token refresh tự động diễn ra

**Luồng Upload:**
- ✓ Xác thực file (size, type) hoạt động
- ✓ Tạo Presigned URL thành công
- ✓ Upload trực tiếp S3 với theo dõi tiến trình
- ✓ Cập nhật trạng thái chính xác (PENDING → VALIDATING)
- ✓ Xử lý lỗi cho uploads thất bại

**Duyệt Admin:**
- ✓ Chỉ admins có thể truy cập trang duyệt
- ✓ Danh sách sách pending hiển thị chính xác
- ✓ Hành động duyệt di chuyển file và cập nhật status
- ✓ Hành động từ chối cập nhật status với lý do
- ✓ Audit logs ghi lại tất cả hành động

**Trải nghiệm đọc:**
- ✓ Danh sách sách chỉ hiển thị sách đã duyệt
- ✓ Chức năng tìm kiếm trả kết quả chính xác
- ✓ Tạo Signed URL hoạt động
- ✓ PDF/ePub render chính xác trong reader
- ✓ Xử lý hết hạn URL một cách graceful

## Quy trình triển khai

### Checklist trước triển khai

**Chất lượng Code:**
- Tất cả lỗi TypeScript đã được giải quyết
- Cảnh báo ESLint đã được xử lý
- Build thành công không lỗi
- Không có console.log trong production

**Biến môi trường:**
- Tất cả env vars cần thiết đã set trong Amplify Console
- Endpoint API production đã cấu hình
- Cognito User Pool IDs chính xác
- CloudFront distribution ID đã set

**Kiểm tra bảo mật:**
- S3 bucket policies hạn chế truy cập công khai
- CloudFront OAC cấu hình đúng
- API Gateway JWT validation bật
- CORS cấu hình chỉ cho production domain

### Triển khai Amplify

**Triển khai tự động:**
```
GitHub push (main branch) → 
Amplify webhook triggers → 
Install dependencies → 
Build Next.js app → 
Deploy to CDN → 
Live tại custom domain
```

**Cài đặt Build:**
- Node.js version chỉ định (18.x)
- Build commands cấu hình
- Environment variables được inject
- Output directory set chính xác

**Xác minh sau triển khai:**
- Site tải thành công
- SSL certificate hoạt động
- Tất cả routes có thể truy cập
- API calls hoạt động
- Xác thực chức năng

## Tối ưu hiệu suất

### Tối ưu Frontend

**Code Splitting:**
- Lazy load các components nặng (PDF reader, ePub viewer)
- Dynamic imports cho admin pages
- Route-based code splitting

**Tối ưu Image:**
- Next.js Image component cho bìa sách
- Lazy loading images below fold
- Format WebP với fallbacks

**Chiến lược Caching:**
- React Query caching cho API responses
- Service worker cho offline capability (tương lai)
- LocalStorage cho user preferences

### Thiết lập giám sát

**Tích hợp CloudWatch:**
- Frontend error tracking qua CloudWatch RUM
- API Gateway access logs
- Lambda execution logs
- Custom metrics cho key actions

**Số liệu hiệu suất:**
- Theo dõi page load time
- Giám sát API response time
- Tỷ lệ thành công/thất bại upload
- Số liệu engagement người dùng

## Những hiểu biết quan trọng

- Kiểm thử toàn diện phát hiện vấn đề trước khi người dùng gặp phải
- CI/CD tự động giảm ma sát triển khai và lỗi con người
- Tối ưu hiệu suất là liên tục - giám sát và lặp lại
- Phản hồi người dùng sau launch vô giá cho ưu tiên cải thiện
- Documentation thiết yếu cho onboarding và maintenance

---

# **Công việc đã hoàn thành**

1. **Unit Testing**
   - Viết tests cho authentication components
   - Kiểm thử logic validation form upload
   - Tạo tests cho API client methods
   - Kiểm thử custom hooks (useUser, useUpload)

2. **Integration Testing**
   - Thiết lập Cypress test suite
   - Viết E2E tests cho auth flow
   - Tạo tests cho upload workflow
   - Kiểm thử quy trình duyệt admin

3. **Sửa lỗi**
   - Sửa vấn đề thời gian refresh token
   - Giải quyết độ chính xác tiến trình upload file
   - Sửa kiểm tra quyền admin panel
   - Chỉnh sửa render PDF trên mobile

4. **Tối ưu hiệu suất**
   - Triển khai code splitting cho reader components
   - Thêm React Query caching với stale-while-revalidate
   - Tối ưu image loading với Next.js Image
   - Giảm bundle size 30%

5. **Triển khai Production**
   - Cấu hình Amplify build settings
   - Set tất cả environment variables trong Amplify Console
   - Triển khai lên production domain
   - Xác minh SSL certificate và HTTPS

6. **Thiết lập giám sát**
   - Bật CloudWatch RUM cho frontend errors
   - Cấu hình CloudWatch alarms cho critical metrics
   - Thiết lập API Gateway logging
   - Tạo CloudWatch dashboard để giám sát

7. **Documentation**
   - Cập nhật README với hướng dẫn thiết lập
   - Ghi chép API endpoints và contracts
   - Tạo hướng dẫn người dùng cho uploaders
   - Viết manual admin cho quy trình duyệt

---

# **Thách thức & Giải pháp**

**Thách thức:** Cypress tests thất bại trong CI/CD pipeline  
**Giải pháp:** Cấu hình base URL phù hợp và thêm retry logic cho tests phụ thuộc network

**Thách thức:** Build thất bại do vấn đề environment variable  
**Giải pháp:** Sử dụng prefix `NEXT_PUBLIC_` cho client-side env vars và ghi chép tất cả variables cần thiết

**Thách thức:** Render PDF gây vấn đề memory trên mobile  
**Giải pháp:** Triển khai page-by-page rendering và vô hiệu hóa pre-caching trên mobile devices

**Thách thức:** Race condition giữa token refresh và API calls  
**Giải pháp:** Triển khai request queue đợi token refresh trước khi tiếp tục

---

# **Tóm tắt Tuần 10**

## Trạng thái hoàn thành dự án

**Xác thực & Quản lý người dùng**
- Tích hợp Cognito với Amplify UI
- Xác thực dựa trên JWT
- Kiểm soát truy cập dựa trên vai trò (User vs Admin)
- Protected routes và authorization

**Luồng Upload**
- Xác thực file và tạo presigned URL
- Upload trực tiếp S3 với theo dõi tiến trình
- Theo dõi trạng thái (PENDING, VALIDATING, APPROVED, REJECTED)
- Lịch sử upload của người dùng

**Quy trình duyệt Admin**
- Truy cập chỉ admin vào giao diện duyệt
- Xem xét sách pending với preview
- Duyệt/từ chối với audit logging
- Hệ thống status badge và notification

**Giao diện đọc sách**
- Trang chi tiết sách với metadata
- Tạo Signed URL cho truy cập an toàn
- Render PDF và ePub
- Chế độ đọc fullscreen

**Tìm kiếm & Khám phá**
- Tìm kiếm theo tiêu đề và tác giả sử dụng DynamoDB GSI
- Tùy chọn lọc và sắp xếp
- Duyệt tất cả sách đã duyệt
- Đề xuất sách liên quan

**Kiểm thử & Đảm bảo chất lượng**
- Unit tests cho các components quan trọng
- Integration tests cho quy trình chính
- E2E tests với Cypress
- Sửa lỗi và xử lý errors

**Triển khai**
- CI/CD với Amplify Hosting
- Môi trường production đã cấu hình
- Custom domain với SSL
- Thiết lập giám sát và logging

## Thành tựu kỹ thuật

- **Kiến trúc Serverless**: Xây dựng hoàn toàn trên AWS managed services
- **Tiết kiệm chi phí**: Ước tính $9.80/tháng cho MVP (100 users)
- **An toàn**: Presigned URLs, CloudFront OAC, JWT authentication
- **Có thể mở rộng**: Xử lý 5,000-50,000 users với thay đổi tối thiểu
- **Stack hiện đại**: Next.js 14, TypeScript, TailwindCSS, React Query

## Bài học kinh nghiệm

1. **Bắt đầu với vertical slices** - Xây dựng từng feature cho phép feedback nhanh hơn
2. **Bảo mật trước tiên** - Triển khai presigned URLs và OAC sớm ngăn lỗ hổng
3. **Kiểm thử liên tục** - Automated tests phát hiện vấn đề trước production
4. **Giám sát mọi thứ** - Tích hợp CloudWatch thiết yếu cho troubleshooting
5. **Ghi chép khi xây dựng** - Documentation tiết kiệm thời gian khi testing và deployment

## Các bước tiếp theo (Cải tiến tương lai)

- Triển khai bookmark và theo dõi tiến trình đọc
- Thêm categories sách và lọc nâng cao
- Xây dựng hệ thống recommendation dựa trên lịch sử đọc
- Thêm tính năng commenting và rating
- Triển khai batch upload cho admins
- Thêm analytics dashboard cho thống kê sử dụng
- Phiên bản mobile app (React Native)

---

**Dự án Thư viện Online hoàn thành thành công!**

Tổng thời gian phát triển: 2 tuần (Ngày 46-50)  
Dòng code: ~5,000+ frontend + backend configurations  
Tính năng đã phát hành: 15+ tính năng user-facing  
Dịch vụ AWS sử dụng: 10 dịch vụ (Amplify, Cognito, API Gateway, Lambda, S3, CloudFront, DynamoDB, Route 53, CloudWatch, CodePipeline)
