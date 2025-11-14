---
title: "Ngày 19 - Disaster Recovery trên AWS"
weight: 4
chapter: false
pre: "<b> 1.4.4. </b>"
---

**Ngày:** 2025-10-02 (Thứ Năm)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## Disaster Recovery (DR) trên AWS

**Disaster Recovery** là quá trình khôi phục dịch vụ CNTT sau sự cố lớn (mất điện, thiên tai, phần cứng hỏng, tấn công mạng).

- **RTO (Recovery Time Objective):** Thời gian cần để khôi phục dịch vụ.  
- **RPO (Recovery Point Objective):** Mức dữ liệu tối đa có thể mất (theo thời gian).

### Chiến lược DR (tăng dần theo độ phức tạp & chi phí)

**Backup & Restore**

- Chỉ lưu backup (snapshot EBS/RDS, S3/Glacier).  
- Khôi phục hạ tầng mới khi gặp sự cố.  
- **RTO:** vài giờ tới vài ngày. **RPO:** phụ thuộc tần suất backup. **Chi phí:** thấp nhất.

**Pilot Light**

- Duy trì các dịch vụ lõi ở trạng thái thu nhỏ trên AWS.  
- Scale lên toàn bộ sản xuất khi DR.  
- **RTO:** hàng giờ. **RPO:** vài phút. **Chi phí:** trung bình.

**Warm Standby**

- Hệ thống hoàn chỉnh chạy ở quy mô giảm trên AWS.  
- Scale lên khi failover.  
- **RTO:** phút – giờ. **RPO:** giây – phút. **Chi phí:** cao hơn.

**Multi-Site (Active/Active hoặc Active/Passive)**

- Môi trường production chạy song song giữa on-prem và AWS, hoặc giữa nhiều Region AWS.  
- Có thể chuyển hướng traffic ngay lập tức (Route 53, Global Accelerator).  
- **RTO/RPO:** gần như bằng 0. **Chi phí:** cao nhất.

**So sánh chiến lược DR:**

| Chiến lược | RTO | RPO | Chi phí | Độ phức tạp |
|------------|-----|-----|---------|-------------|
| Backup & Restore | Giờ – Ngày | Giờ | $ | Thấp |
| Pilot Light | Giờ | Phút | $$ | Trung bình |
| Warm Standby | Phút | Giây | $$$ | Trung bình-Cao |
| Multi-Site | Giây | Gần 0 | $$$$ | Cao |

---

## Best Practices cho DR

### Lập kế hoạch

- Xác định yêu cầu RTO và RPO.  
- Tài liệu hóa quy trình khôi phục.  
- Nhận diện hệ thống và phụ thuộc quan trọng.  
- Thiết lập kế hoạch truyền thông.

### Triển khai

- Tự động hóa quy trình khôi phục.  
- Sử dụng nhiều AZ và Region.  
- Triển khai cơ chế sao chép dữ liệu.  
- Kiểm thử backup định kỳ.

### Kiểm thử

- Thực hiện diễn tập DR thường xuyên.  
- Thử nghiệm quy trình khôi phục.  
- Đo lường RTO/RPO thực tế.  
- Cập nhật tài liệu.

---

# **Hands-On Labs**

## Lab 14 – AWS VM Import/Export (Phần 2)

4. Import máy ảo lên AWS → *14-02.3*  
5. Deploy instance từ AMI → *14-02.4*  
6. Thiết lập ACL cho S3 Bucket → *14-03.1*  
7. Export máy ảo từ instance → *14-03.2*  
8. Dọn dẹp tài nguyên trên AWS → *14-05*
