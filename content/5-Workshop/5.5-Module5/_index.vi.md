---
title: "Module 5: Tìm kiếm & Khám phá"
date: "2025-01-15T13:00:00+07:00"
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

## Tổng quan

Trong module này, bạn sẽ triển khai chức năng tìm kiếm cho Thư viện Online sử dụng DynamoDB Global Secondary Indexes (GSI). Bạn sẽ xây dựng các truy vấn hiệu quả cho tìm kiếm theo tiêu đề và tác giả, triển khai phân trang, và tạo giao diện tìm kiếm thân thiện với người dùng có bộ lọc.

**Thời lượng:** ~60 phút

**Dịch vụ sử dụng:**
- Amazon DynamoDB (GSI cho tìm kiếm)
- AWS Lambda (logic tìm kiếm)
- Amazon API Gateway (search endpoint)

---

## Những gì bạn sẽ học

- Thiết kế DynamoDB GSI cho tìm kiếm hiệu quả
- Query GSI thay vì Scan operations tốn kém
- Triển khai phân trang với cursor-based navigation
- Xây dựng UI tìm kiếm với autocomplete
- Thêm bộ lọc và khả năng sắp xếp

---

## Kiến trúc cho Module này

```
User → Form tìm kiếm → API Gateway → Lambda (searchBooks)
                                    ↓
                              Query GSI1 (Title) hoặc GSI2 (Author)
                                    ↓
                              Trả kết quả đã lọc & phân trang
                                    ↓
User ← Kết quả tìm kiếm ← Response
```

---

## Xác minh & Kiểm thử

### Checklist

- DynamoDB GSI được tạo cho title và author
- Search Lambda queries GSI hiệu quả
- Tìm kiếm theo title trả kết quả đúng
- Tìm kiếm theo author trả kết quả đúng
- Tìm kiếm kết hợp (title VÀ author) hoạt động
- Phân trang với nextToken chức năng
- UI tìm kiếm debounces input
- Nút load more tải thêm kết quả

### Các vấn đề thường gặp & Giải pháp

**Vấn đề:** Tìm kiếm trả quá nhiều kết quả  
**Giải pháp:** Triển khai begins_with matching chặt chẽ hơn hoặc thêm bộ lọc bổ sung

**Vấn đề:** Hiệu suất tìm kiếm chậm  
**Giải pháp:** Đảm bảo GSI được cấu hình đúng và sử dụng Query không phải Scan

**Vấn đề:** Phân trang không hoạt động đúng  
**Giải pháp:** Xác minh LastEvaluatedKey được encode/decode đúng cách

---

## Dọn dẹp

```bash
cdk destroy
```

---

## Bước tiếp theo

Chúc mừng! Bạn đã hoàn thành Module 5. Bây giờ bạn có:
- Tìm kiếm hiệu quả sử dụng DynamoDB GSI
- Tìm kiếm theo title và author với prefix matching
- Phân trang cho tập kết quả lớn
- Giao diện tìm kiếm thân thiện với debouncing

**Tiếp tục đến:** [Module 6: Triển khai & Vận hành](../5.6-module6/)

---

## Tài nguyên bổ sung

- [Tài liệu DynamoDB GSI](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GSI.html)
- [DynamoDB Query Operations](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.html)
- [Pagination Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Query.Pagination.html)
