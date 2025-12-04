---
title: "Ngày 49 - Giao diện đọc sách & Triển khai tìm kiếm"
weight: 4
chapter: false
pre: "<b> 1.10.4. </b>"
---

**Ngày:** 2025-11-13 (Thứ Năm)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú bài giảng**

## Trải nghiệm đọc sách

### CloudFront Signed URLs cho phân phối nội dung

**Chiến lược bảo mật:**
- Sách lưu trong S3 `public/books/` nhưng **không thể truy cập công khai**
- CloudFront cấu hình với **Origin Access Control (OAC)**
- S3 bucket policy chỉ cho phép CloudFront truy cập objects
- Lambda tạo **signed GET URLs** với TTL ngắn (15 phút)
- Người dùng truy cập nội dung chỉ qua signed URLs

**Luồng Signed URL:**
```
User nhấp "Đọc" → Frontend gọi API /books/{id}/read → 
Lambda xác thực user authentication → Lambda kiểm tra book status (APPROVED) →
Lambda tạo CloudFront signed URL → Trả URL cho frontend →
Frontend load PDF/ePub qua signed URL
```

### Render PDF/ePub

**Tùy chọn triển khai:**
- **react-pdf**: Render PDF trong browser với canvas
- **PDF.js**: PDF viewer của Mozilla (nhiều tính năng hơn)
- **epub.js**: ePub reader với tùy chỉnh
- **iframe**: Đơn giản nhưng ít kiểm soát

**Tính năng Reader:**
- Điều hướng trang (trước/sau)
- Phóng to/thu nhỏ
- Chế độ toàn màn hình
- Hỗ trợ bookmark (lưu trong DynamoDB)
- Theo dõi tiến trình đọc

## Tìm kiếm & Khám phá

### Tìm kiếm dựa trên DynamoDB GSI

**Chiến lược tìm kiếm:**
- Tạo **Global Secondary Indexes (GSI)** cho các trường có thể tìm kiếm
- GSI1: `PK=TITLE#{normalizedTitle}, SK=BOOK#{bookId}`
- GSI2: `PK=AUTHOR#{normalizedAuthor}, SK=BOOK#{bookId}`
- Query GSI thay vì Scan operations tốn kém

**Luồng tìm kiếm:**
```
User nhập query tìm kiếm → Frontend gọi API /search với query params →
Lambda xác định GSI nào cần query →
Query GSI1 (title) và/hoặc GSI2 (author) →
Intersection kết quả nếu có nhiều params →
BatchGetItem để lấy full book metadata →
Trả kết quả cho frontend
```

### Triển khai tìm kiếm Frontend

**Search Components:**
- Thanh tìm kiếm với autocomplete
- Bảng lọc (theo tác giả, thể loại, ngày upload)
- Tùy chọn sắp xếp (mới nhất, cũ nhất, phổ biến)
- Chuyển đổi xem grid/list
- Phân trang hoặc infinite scroll

**Tối ưu tìm kiếm:**
- Debounce search input (chờ 300ms sau khi gõ)
- Cache kết quả tìm kiếm gần đây (5 phút)
- Hiển thị loading skeleton trong quá trình tìm kiếm
- Hiển thị "No results" với suggestions

## Những hiểu biết quan trọng

- Signed URLs thiết yếu cho bảo mật nội dung - không bao giờ expose S3 trực tiếp
- TTL ngắn trên signed URLs ngăn chia sẻ/lạm dụng
- Query GSI rẻ hơn và nhanh hơn nhiều so với Scan
- Normalized fields (chữ thường, không ký tự đặc biệt) cải thiện độ chính xác tìm kiếm
- Caching client-side giảm API calls và cải thiện UX

---

# **Công việc đã hoàn thành**

1. **Trang chi tiết sách**
   - Tạo view chi tiết sách với hiển thị metadata
   - Triển khai loading ảnh bìa từ S3
   - Thêm nút "Read Now" (chỉ cho sách APPROVED)
   - Xây dựng phần sách liên quan

2. **Tích hợp PDF Reader**
   - Tích hợp thư viện react-pdf để render PDF
   - Tạo component reader với navigation controls
   - Triển khai zoom controls (fit-width, fit-page, custom zoom)
   - Thêm chuyển đổi chế độ fullscreen

3. **Tích hợp ePub Reader**
   - Tích hợp thư viện epub.js để render ePub
   - Tạo ePub reader với page turn animations
   - Triển khai text selection và note-taking
   - Thêm font size và theme điều chỉnh được

4. **Triển khai Signed URL**
   - Tạo phương thức API client `getReadUrl`
   - Triển khai fetching signed URL khi nhấp "Read"
   - Thêm xử lý URL expiration (re-fetch khi hết hạn)
   - Xây dựng trạng thái loading khi fetching URL

5. **Chức năng tìm kiếm**
   - Tạo component thanh tìm kiếm với autocomplete
   - Triển khai tích hợp API tìm kiếm (endpoint `/search`)
   - Xây dựng trang kết quả tìm kiếm với grid/list view
   - Thêm sidebar lọc (tác giả, thể loại, trạng thái)

6. **Tối ưu tìm kiếm**
   - Triển khai debounced search input (delay 300ms)
   - Thêm caching kết quả client-side với React Query
   - Tạo lịch sử tìm kiếm (lưu trong localStorage)
   - Xây dựng dropdown "Recent Searches"

7. **Tính năng khám phá**
   - Tạo trang "Browse Books" với tất cả sách đã duyệt
   - Triển khai lọc category/genre
   - Thêm tùy chọn sắp xếp (mới nhất, title A-Z, author A-Z)
   - Xây dựng phần "Recommended for You" (tương lai: dựa trên ML)

---

# **Thách thức & Giải pháp**

**Thách thức:** Vấn đề hiệu suất render PDF với files lớn  
**Giải pháp:** Triển khai lazy loading của pages (chỉ render pages visible) và sử dụng Web Workers cho PDF parsing

**Thách thức:** Signed URLs hết hạn trong phiên đọc  
**Giải pháp:** Triển khai cơ chế auto-refresh yêu cầu URL mới 2 phút trước khi hết hạn

**Thách thức:** Tìm kiếm trả quá nhiều kết quả gây UI chậm  
**Giải pháp:** Triển khai phân trang server-side với cursor-based navigation và virtual scrolling

**Thách thức:** Text selection ePub can thiệp page turns  
**Giải pháp:** Thêm touch/click handlers với gesture detection để phân biệt selection và navigation
