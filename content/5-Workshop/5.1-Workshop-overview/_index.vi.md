---
title: "Tổng quan Workshop"
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
---

## Mục tiêu

Giới thiệu workshop xác thực: cài đặt AWS Amplify CLI, tạo ứng dụng React với luồng xác thực hoàn chỉnh sử dụng Amazon Cognito, triển khai đăng ký người dùng, đăng nhập, quản lý mật khẩu, MFA và đăng nhập qua mạng xã hội. Nhấn mạnh việc triển khai xác thực an toàn nhanh chóng mà không cần quản lý hạ tầng.

## Kiến trúc tổng quan

- Amazon Cognito User Pool quản lý thư mục người dùng và xác thực.  
- AWS Amplify cung cấp công cụ CLI và thư viện frontend để tích hợp liền mạch.  
- Ứng dụng React sử dụng Amplify UI components cho giao diện xác thực.  
- JWT tokens xử lý quản lý phiên và phân quyền API.

## Kết quả mong đợi

- Ứng dụng React với đăng ký người dùng và xác minh email.  
- Chức năng đăng nhập/đăng xuất với session persistence.  
- Tính năng đặt lại và thay đổi mật khẩu.  
- Xác thực đa yếu tố (MFA) với TOTP.  
- Tích hợp đăng nhập mạng xã hội (Google/Facebook).  
- Protected routes với role-based access control.

## Cấu trúc & thời gian

- Module 1: Cài đặt Amplify và cấu hình ban đầu.  
- Module 2: Luồng đăng ký người dùng và xác minh email.  
- Module 3: Đăng nhập, quản lý phiên và đăng xuất.  
- Module 4: Quản lý mật khẩu (quên/đặt lại/thay đổi).  
- Module 5: Tính năng nâng cao (MFA và social login).  
- Module 6: Triển khai production và dọn dẹp.