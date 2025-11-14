---
title: "Day 33 - Next.js App Router"
weight: 3
chapter: false
pre: "<b> 1.7.3. </b>"
---

**Date:** 2025-10-22 (Thứ Tư)  
**Status:** "Done"  

---

# **Lecture Notes**

## Next.js 16 App Router

- Tận dụng **Server Components** để fetch data trực tiếp từ server và tránh bundle dư thừa.
- Route `/app/books/[id]/page.tsx` xử lý data fetching, trả UI đã được render sẵn.
- `generateMetadata` cung cấp SEO meta dựa trên dữ liệu sách.

```tsx
// app/books/[id]/page.tsx
import { getBook } from "@/lib/api";

export default async function BookDetail({ params }) {
  const book = await getBook(params.id);
  if (!book) return notFound();
  return <BookPage book={book} />;
}
```

### Error & Not Found Handling

- Chỉ cần `not-found.tsx` cho case không tìm được sách → coi như expected flow.
- Không dùng `error.tsx` để tránh double handling; các lỗi còn lại log ở backend.
- Giữ UX nhất quán: hiển thị CTA quay lại danh sách và hotline hỗ trợ.

## Environment & Config

- Sử dụng biến môi trường rõ ràng: `NEXT_PUBLIC_API_URL` cho frontend, `API_URL` cho route handlers.
- Đảm bảo `.env.example` cập nhật khi thêm biến mới.
- Chuyển logic build URL vào helper (`lib/api.ts`) để tránh lặp.

## Insight

- Server Components giảm latency đáng kể khi render detail page.
- App Router giúp cấu trúc thư mục rõ ràng, dễ chia nhỏ khi thêm slice mới.
- Khi mock API, dùng fetch trực tiếp từ Prism, chỉ cần đổi base URL là xong.

---

# **Hands-On Labs**

- Tạo `not-found.tsx` tùy biến với CTA điều hướng.
- Viết helper `getBook(id)` tái sử dụng cho server components và tests.
- Kiểm tra build `npm run lint && npm run build` đảm bảo cấu hình Next.js sạch.
