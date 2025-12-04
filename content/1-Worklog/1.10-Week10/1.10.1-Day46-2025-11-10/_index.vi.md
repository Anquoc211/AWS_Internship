---
title: "Ngày 46 - Xác thực & Thiết lập dự án"
weight: 1
chapter: false
pre: "<b> 1.10.1. </b>"
---

**Ngày:** 2025-11-10 (Thứ Hai)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú bài giảng**

## Xem lại ngữ cảnh dự án

### Kiến trúc Thư viện Online

**Frontend Stack:**
- Next.js 14 (App Router)
- AWS Amplify (Hosting + CI/CD)
- Amplify UI Components
- TailwindCSS cho styling
- React Query cho quản lý state

**Tích hợp dịch vụ AWS:**
- Amazon Cognito (Xác thực)
- API Gateway (HTTP API endpoints)
- S3 (Lưu trữ file qua presigned URLs)
- CloudFront (Phân phối nội dung)
- DynamoDB (Lưu trữ metadata)

### Thiết lập môi trường phát triển

**Yêu cầu:**
- Node.js 18+ và npm/yarn
- AWS CLI đã cấu hình
- Git và tài khoản GitHub
- Tài khoản AWS với quyền phù hợp

**Cấu trúc dự án:**
```
online-library/
├── src/
│   ├── app/              # Trang Next.js App Router
│   ├── components/       # UI components tái sử dụng
│   ├── lib/              # Hàm tiện ích & AWS SDK
│   ├── hooks/            # Custom React hooks
│   └── types/            # Định nghĩa TypeScript
├── public/               # Tài nguyên tĩnh
├── amplify/              # Cấu hình Amplify
└── .env.local            # Biến môi trường
```

## Luồng xác thực

### Chiến lược tích hợp Cognito

**Hành trình người dùng:**
1. Đăng ký → Xác thực email → Đăng nhập
2. Lưu trữ JWT token trong localStorage/cookies
3. Tự động refresh token trước khi hết hạn
4. Truy cập dựa trên vai trò (User vs Admin)

**Các thành phần chính:**
- Trang đăng nhập với email/password
- Form đăng ký với validation
- Luồng xác thực email
- Chức năng đặt lại mật khẩu
- HOC bảo vệ route

### Xác thực Amplify UI

**Lợi ích:**
- Components đã xây dựng sẵn, có thể tùy chỉnh
- Validation form tích hợp sẵn
- Quản lý token tự động
- Thiết kế responsive ngay từ đầu

**Cấu hình:**
```javascript
// amplify-config.js
const amplifyConfig = {
  Auth: {
    region: 'ap-southeast-1',
    userPoolId: process.env.NEXT_PUBLIC_USER_POOL_ID,
    userPoolWebClientId: process.env.NEXT_PUBLIC_USER_POOL_CLIENT_ID,
  },
  API: {
    endpoints: [
      {
        name: 'OnlineLibraryAPI',
        endpoint: process.env.NEXT_PUBLIC_API_ENDPOINT,
      }
    ]
  }
}
```

## Những hiểu biết quan trọng

- Amplify UI components giảm đáng kể thời gian triển khai auth
- Token refresh nên tự động diễn ra ở background
- Lưu role/groups của user trong JWT claims cho authorization frontend
- Biến môi trường quan trọng cho bảo mật - không bao giờ commit dữ liệu nhạy cảm
- Thiết lập CI/CD trong Amplify yêu cầu cài đặt build phù hợp

---

# **Công việc đã hoàn thành**

1. **Khởi tạo dự án**
   - Tạo dự án Next.js 14 với TypeScript và TailwindCSS
   - Cài đặt AWS Amplify và các dependencies cần thiết
   - Thiết lập cấu trúc thư mục dự án

2. **Cấu hình Amplify**
   - Cấu hình Amplify với thông tin đăng nhập Cognito User Pool
   - Thiết lập cấu hình endpoint API Gateway
   - Tạo cấu trúc biến môi trường

3. **Trang xác thực**
   - Xây dựng trang đăng nhập với Amplify Authenticator
   - Tạo trang đăng ký với xác thực email
   - Triển khai HOC bảo vệ route cho các trang yêu cầu xác thực

4. **Quản lý người dùng**
   - Tạo custom hook `useUser` cho ngữ cảnh người dùng
   - Triển khai xử lý JWT token
   - Thiết lập phát hiện vai trò admin từ Cognito groups

5. **Thiết lập CI/CD**
   - Kết nối repository GitHub với Amplify Hosting
   - Cấu hình cài đặt build cho triển khai Next.js
   - Thiết lập biến môi trường trong Amplify Console

---

# **Thách thức & Giải pháp**

**Thách thức:** Thay đổi API Amplify v6 so với phiên bản trước  
**Giải pháp:** Cập nhật sử dụng cú pháp Amplify.configure() mới và Authenticator.Provider

**Thách thức:** Thời gian refresh token gây lỗi 401  
**Giải pháp:** Triển khai tự động refresh token 5 phút trước khi hết hạn

**Thách thức:** Vai trò admin không phản ánh ngay sau khi đăng nhập  
**Giải pháp:** Thêm cơ chế re-fetch token sau khi xác thực thành công
