---
title: "Ngày 04 - Tối ưu Chi phí trên AWS"
weight: 4
chapter: false
pre: "<b> 1.1.4. </b>"
---

**Ngày:** 2025-09-11 (Thứ Năm)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Tối ưu chi phí trên AWS

### Chiến lược tối ưu chi phí

- Chọn đúng loại tài nguyên và Region phù hợp.  
- Sử dụng các mô hình giá như Reserved Instances, Savings Plans, Spot Instances.  
- Tắt hoặc lên lịch các tài nguyên không dùng.  
- Tận dụng kiến trúc serverless để giảm chi phí vận hành.  
- Liên tục rà soát hiệu quả chi phí bằng AWS Budgets và Cost Explorer.  
- Gắn thẻ chi phí (Cost Allocation Tags) để theo dõi theo phòng ban.

### AWS Pricing Calculator

[calculator.aws](https://calculator.aws/#/)

- Tạo và chia sẻ ước tính chi phí cho các dịch vụ phổ biến.  
- Giá thay đổi tùy Region.

![image](/images/1-Worklog/Week1/image%202.png)

**Tính năng chính:**

- Ước tính chi phí trước khi triển khai.  
- So sánh giá giữa các Region.  
- Xuất và chia sẻ bảng dự toán.  
- Có sẵn template cho từng workload.

---

## Gói hỗ trợ AWS Support Plans

- Bốn cấp độ: **Basic**, **Developer**, **Business**, **Enterprise**.  
- Có thể nâng cấp tạm thời khi gặp sự cố nghiêm trọng.

### So sánh các gói hỗ trợ

| Tính năng | Basic | Developer | Business | Enterprise |
|-----------|-------|-----------|----------|------------|
| Chi phí | Miễn phí | 29 USD/tháng | 100 USD/tháng | 15.000 USD/tháng |
| Thời gian phản hồi | Không áp dụng | 12-24 giờ | 1 giờ (khẩn) | 15 phút (nghiêm trọng) |
| Hỗ trợ kỹ thuật | Diễn đàn | Giờ hành chính | 24/7 | 24/7 + TAM |

---

# **Hands-On Labs**

### Lab 07 – AWS Budgets & Cost Management

1. Tạo Budget từ template → *07-01*  
2. Hướng dẫn tạo Cost Budget → *07-02*  
3. Tạo Usage Budget → *07-03*  
4. Tạo Budget cho Reserved Instance (RI) → *07-04*  
5. Tạo Budget cho Savings Plans → *07-05*  
6. Dọn dẹp các Budget → *07-06*

### Lab 09 – AWS Support Plans

1. Các gói hỗ trợ AWS → *09-01*  
2. Phân loại yêu cầu hỗ trợ → *09-02*  
3. Thay đổi gói hỗ trợ → *09-03*  
4. Quản lý ticket hỗ trợ → *09-04*
