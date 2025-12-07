---
title: "Yêu cầu"
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

## Tài khoản & Quyền

- Tài khoản AWS với quyền tạo Cognito User Pools, IAM roles và tài nguyên Amplify.  
- Không cần quyền VPC hoặc networking nâng cao cho workshop này.

## Công cụ

- **Node.js 18+** và npm đã cài đặt trên máy local.  
- **AWS Amplify CLI**: Cài đặt bằng `npm install -g @aws-amplify/cli`  
- **Code editor** (VS Code, Sublime hoặc editor bạn thích).  
- **Trình duyệt web** để test luồng xác thực.

## Cài đặt nhanh

1. **Cấu hình Amplify CLI:**
   ```bash
   amplify configure
   ```
   Làm theo hướng dẫn để tạo IAM user với quyền phù hợp.

2. **Chọn region** gần với bạn (ví dụ: `us-east-1` hoặc `ap-southeast-1`).

3. **Kiểm tra cài đặt:**
   ```bash
   node --version  # Phải là 18+
   npm --version
   amplify --version
   ```