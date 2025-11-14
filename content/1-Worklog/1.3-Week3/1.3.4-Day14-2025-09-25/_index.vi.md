---
title: "Ngày 14 - EC2 Auto Scaling"
weight: 4
chapter: false
pre: "<b> 1.3.4. </b>"
---

**Ngày:** 2025-09-25 (Thứ Năm)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Amazon EC2 Auto Scaling

- **EC2 Auto Scaling** tự động điều chỉnh số lượng instance EC2 dựa trên nhu cầu.

**Lợi ích**

- Co giãn năng lực linh hoạt.  
- Tăng tính sẵn sàng cho ứng dụng.  
- Tối ưu chi phí.

**Thành phần chính**

- **Auto Scaling Group (ASG):** nhóm logic chứa các EC2 instance.  
- **Launch Template / Configuration:** định nghĩa thông số instance.  
- **Scaling Policy:** quy tắc thêm/bớt instance.

### Scaling Policy

- **Simple / Step Scaling:** thêm/bớt instance khi vượt ngưỡng.  
- **Target Tracking:** duy trì một metric (ví dụ CPU = 50%).  
- **Scheduled Scaling:** scale theo lịch định sẵn.  
- **Predictive Scaling:** dùng ML dự đoán và scale chủ động.

**Ví dụ Target Tracking:**

```json
{
  "TargetTrackingScalingPolicyConfiguration": {
    "PredefinedMetricSpecification": {
      "PredefinedMetricType": "ASGAverageCPUUtilization"
    },
    "TargetValue": 50.0
  }
}
```

### Tích hợp với Load Balancer

- ASG thường đi kèm **Elastic Load Balancer (ELB)**.  
- Instance mới sẽ tự đăng ký, instance bị hủy sẽ tự hủy đăng ký.

**Best practices cho Auto Scaling:**

- Dùng nhiều AZ để tăng độ sẵn sàng.  
- Thiết lập cooldown hợp lý.  
- Giám sát metric trên CloudWatch.  
- Sử dụng lifecycle hook cho tác vụ tùy chỉnh.  
- Kiểm thử policy trước khi đưa vào production.

---

## Các mô hình giá của EC2

- **On-Demand:** Trả theo giờ/giây. Linh hoạt nhất nhưng chi phí cao.  
- **Reserved Instances:** Cam kết 1 hoặc 3 năm để được giảm giá; gắn với loại/family cụ thể.  
- **Savings Plans:** Cam kết 1 hoặc 3 năm; linh hoạt hơn giữa các family.  
- **Spot Instances:** Dùng công suất dư thừa, giảm tới 90%; có thể bị thu hồi sau 2 phút báo trước.

> Nên kết hợp nhiều mô hình giá trong Auto Scaling Group để tối ưu chi phí.

**So sánh giá:**

| Mô hình | Giảm giá | Linh hoạt | Cam kết |
|---------|----------|-----------|---------|
| On-Demand | 0% | Cao | Không |
| Reserved | 40-60% | Thấp | 1-3 năm |
| Savings Plans | 40-60% | Trung bình | 1-3 năm |
| Spot | 50-90% | Thấp | Không |

---

# **Hands-On Labs**

## Lab 09 – AWS Support Plans

1. Các gói hỗ trợ AWS → *09-01*  
2. Phân loại yêu cầu hỗ trợ → *09-02*  
3. Thay đổi gói hỗ trợ → *09-03*  
4. Quản lý ticket hỗ trợ → *09-04*
