---
title: "Ngày 18 - AWS Snow Family & Hybrid Storage"
weight: 3
chapter: false
pre: "<b> 1.4.3. </b>"
---

**Ngày:** 2025-10-01 (Thứ Tư)  
**Trạng thái:** "Hoàn thành"  

---

# **Ghi chú Bài học**

## AWS Snow Family

Bộ thiết bị/dịch vụ chuyên dụng để di chuyển khối lượng dữ liệu lớn vào/ra AWS khi băng thông hạn chế hoặc dữ liệu quá khủng.

- **AWS Snowcone:** Thiết bị nhỏ gọn, chịu va đập (~8 TB). Phù hợp môi trường edge, vùng xa.  
- **AWS Snowball:**  
  - *Snowball Edge Storage Optimized:* Tối đa ~80 TB lưu trữ khả dụng.  
  - *Snowball Edge Compute Optimized:* Thêm khả năng compute mạnh với ~42 TB lưu trữ.  
- **AWS Snowmobile:** Trung tâm dữ liệu container hóa, phục vụ chuyển dữ liệu quy mô exabyte (tới 100 PB).

**So sánh Snow Family:**

| Thiết bị | Lưu trữ | Compute | Use case |
|----------|---------|---------|----------|
| Snowcone | 8 TB | 2 vCPU | Edge, IoT |
| Snowball Storage | 80 TB | 40 vCPU | Di chuyển dữ liệu |
| Snowball Compute | 42 TB | 52 vCPU | Edge computing |
| Snowmobile | 100 PB | N/A | Di dời datacenter |

**Khi nào dùng Snow Family:**

- Băng thông hạn chế hoặc chi phí cao.  
- Khối lượng dữ liệu rất lớn (TB tới PB).  
- Địa điểm xa xôi, khó kết nối.  
- Nhu cầu xử lý edge computing.  
- Yêu cầu tuân thủ lưu trữ dữ liệu tại chỗ.
