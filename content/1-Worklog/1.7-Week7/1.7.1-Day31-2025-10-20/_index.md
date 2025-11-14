---
title: "Day 31 - Vertical Slice Kickoff"
weight: 1
chapter: false
pre: "<b> 1.7.1. </b>"
---

**Date:** 2025-10-20 (Thứ Hai)  
**Status:** "Done"  

---

# **Lecture Notes**

## Project Context

### Ebook Demo – Vertical Slice 0

- Mục tiêu: demo tính năng **xem chi tiết sách** end-to-end trước khi phát triển toàn bộ hệ thống.
- Cách tiếp cận: **Vertical Slice Architecture** để xây từng lát cắt hoàn chỉnh thay vì làm theo tầng.
- Lợi ích: demo được ngay, phát hiện lỗi sớm, tạo nhịp phối hợp giữa frontend/backend.

### Kiến trúc Slice

```
User → Frontend → API → Database → Response → UI
```

- Mỗi lát cắt bao gồm UI, API contract, backend logic và dữ liệu giả để trình diễn.
- Cho phép thay thế từng thành phần mà không ảnh hưởng đến toàn bộ hệ thống.

## Vertical Slice Architecture

### Nguyên tắc chính

- Phát triển feature theo luồng người dùng, không tách biệt tầng.
- Giữ scope nhỏ để **demo nhanh** và nhận feedback sớm.
- Rõ ràng trách nhiệm của từng slice, giúp mở rộng dễ dàng.

### Lợi ích

- Tăng tốc đóng gói giá trị: có thể show cho stakeholder ngay.
- Giảm rủi ro integration vì mỗi slice tự kiểm chứng.
- Cho phép phát triển song song giữa các slice khác nhau.

## Key Insights

- Vertical slice đóng vai trò nền tảng trước khi mở rộng sang các feature khác.
- Mỗi slice cần có checklist rõ ràng (UI hoàn thiện, contract chuẩn, backend trả dữ liệu đúng).
- Coi slice như “mini product” với vòng đời riêng giúp giữ chất lượng.

---

# **Hands-On Labs**

- Xác định phạm vi slice 0 (luồng xem chi tiết sách, data tối thiểu).
- Vẽ sơ đồ luồng dữ liệu và xác định boundary giữa frontend/backend.
- Chuẩn hóa checklist demo (contract, mock, UI, backend).
