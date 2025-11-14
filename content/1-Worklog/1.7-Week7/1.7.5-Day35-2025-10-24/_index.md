---
title: "Day 35 - Contract Testing & Retrospective"
weight: 5
chapter: false
pre: "<b> 1.7.5. </b>"
---

**Date:** 2025-10-24 (Thứ Sáu)  
**Status:** "Done"  

---

# **Lecture Notes**

## Contract Testing với Schemathesis

- Dùng lệnh `schemathesis run --checks all --workers 4 --url http://127.0.0.1:8000/openapi.yaml` để tạo test tự động.
- Schemathesis sinh nhiều tình huống ngẫu nhiên (happy path, edge cases, missing fields).
- Giúp đảm bảo backend không trả sai schema khi frontend chuyển sang gọi API thật.

### Lợi ích

- Không cần viết tay test case phức tạp.
- Giảm rủi ro mismatch sau khi refactor.
- Hoạt động như quality gate trong pipeline CI.

## Mistakes & Fixes

| Mistake                               | Nguyên nhân        | Cách khắc phục                 |
| ------------------------------------- | ------------------ | ------------------------------ |
| Tạo cả `error.tsx` và `not-found.tsx` | Redundant          | Giữ lại `not-found.tsx` thôi   |
| Dùng `--base-url` trong Schemathesis  | Lệnh sai           | Dùng `--url` đúng chuẩn        |
| Timeout khi load `/openapi.json`      | CORS hoặc chậm     | Dùng file YAML trực tiếp       |
| Over-engineer backend                 | Quá nhiều file sớm | Bắt đầu đơn giản, refactor sau |

## Workflow Chuẩn Đã Validate

```
1. Define Contract (OpenAPI)
2. Mock API (Prism)
3. Build Frontend với mock data
4. Implement Backend theo spec
5. Switch sang real API
6. Contract Testing (Schemathesis)
```

- Hỗ trợ phát triển song song giữa team frontend và backend.
- Giảm xung đột, tăng tốc demo và giữ chất lượng ổn định.

## Key Insights

1. Contract-first development giữ spec đồng bộ và giảm lỗi integration.  
2. Vertical slice cho phép release từng phần, nhận feedback sớm.  
3. Automation (Prism, Schemathesis) giúp giảm effort test thủ công.  
4. Bắt đầu đơn giản, refactor dần khi cần mở rộng.

---

# **Hands-On Labs**

- Chạy Schemathesis với spec mới nhất và ghi nhận kết quả.
- Cập nhật README workflow cho team tham khảo.
- Chuẩn bị backlog cho vertical slice tiếp theo dựa trên feedback demo.
