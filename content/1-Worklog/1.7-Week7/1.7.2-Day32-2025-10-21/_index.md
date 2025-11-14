---
title: "Day 32 - Contract-First & Mocking"
weight: 2
chapter: false
pre: "<b> 1.7.2. </b>"
---

**Date:** 2025-10-21 (Thứ Ba)  
**Status:** "Done"  

---

# **Lecture Notes**

## Contract-First Development

### Quy trình 5 bước

1. Viết OpenAPI spec để định nghĩa contract.  
2. Chia sẻ spec cho cả frontend và backend như **Single Source of Truth**.  
3. Frontend dựng UI với mock data dựa trên contract.  
4. Backend implement API bám sát schema (status code, payload).  
5. Chạy contract testing để đảm bảo backend tuân thủ spec.

```yaml
paths:
  /books/{id}:
    get:
      summary: Get book detail
      responses:
        "200":
          $ref: "#/components/responses/BookDetail"
```

### Lợi ích

- Hạn chế mismatch API vì mọi người xem chung một spec.
- Documentation, mock server, test script đều sinh tự động.
- Dễ review và cập nhật version của API trước khi triển khai thật.

### Insight

> Contract trước code giúp giảm ~80% lỗi integration khi hai team làm song song.

## Mock API với Prism

- Prism dùng OpenAPI để sinh response giả, cho phép frontend test UI sớm.
- Hỗ trợ nhiều scenario (200, 404, 500) chỉ bằng cách gán example trong spec.
- Giúp giữ nhịp làm việc khi backend chưa xong hoặc đang refactor.

### Khi nên dùng

- Sprint đầu tiên của vertical slice.  
- Cần trình diễn flow mà chưa có dữ liệu thật.  
- Muốn viết test tự động cho UI dựa trên contract.

## Ghi chú vận hành

- Chạy Prism ở `localhost:4010`, trỏ Next.js đến mock qua `NEXT_PUBLIC_API_URL`.
- Đảm bảo CORS headers trong mock giống với backend sản xuất.
- Luôn commit spec trước khi mock để mọi người dùng đúng phiên bản.

---

# **Hands-On Labs**

- Thiết lập OpenAPI spec cho endpoint `/books/{id}`.
- Khởi chạy Prism mock server và kiểm tra flow trên UI.
- Viết checklist review contract (status code, schema, example data).
