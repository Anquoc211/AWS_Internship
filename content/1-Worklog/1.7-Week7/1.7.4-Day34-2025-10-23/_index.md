---
title: "Day 34 - FastAPI Clean Architecture"
weight: 4
chapter: false
pre: "<b> 1.7.4. </b>"
---

**Date:** 2025-10-23 (Thứ Năm)  
**Status:** "Done"  

---

# **Lecture Notes**

## Clean Architecture Overview

- Tách rõ phần cấu hình, model, route và core logic để dễ mở rộng.
- Giữ `main.py` nhẹ: chỉ khởi tạo app, load config, mount router.
- Dùng Pydantic model để chuẩn hóa request/response, đảm bảo contract khớp OpenAPI.

```
backend/
├── main.py
├── core/
│   └── config.py
├── models/
│   └── book.py
├── routes/
│   └── books.py
└── services/
    └── books.py
```

## Cấu hình & Dependency

- `core/config.py` đọc biến môi trường, gom cấu hình CORS, API prefix, debug flag.
- Sử dụng dependency injection của FastAPI để truyền service layer vào router.
- Cho phép thay thế datasource (in-memory → PostgreSQL) mà không đổi giao diện hàm.

## CORS & API Stability

- CORS chỉ mở origin cần thiết (`http://localhost:3000` khi dev).
- Bật `allow_methods=["GET"]` cho slice đầu tiên để giảm bề mặt tấn công.
- Đảm bảo `/openapi.json` luôn trả về được để công cụ contract testing sử dụng.

## Start Simple, Refactor Later

- Bắt đầu bằng in-memory repository để demo nhanh, sau đó bổ sung DB thật.
- Document rõ TODO để tránh quên khi chuyển sang sprint tiếp theo.
- Logging tối giản, tập trung vào lỗi quan trọng (timeout, data mismatch).

---

# **Hands-On Labs**

- Refactor `main.py` chỉ giữ việc khởi tạo và router registration.
- Viết service layer `get_book_detail(id)` với dữ liệu giả lấy từ spec.
- Cấu hình `CORSMiddleware` khớp với URL của frontend mock và production.
