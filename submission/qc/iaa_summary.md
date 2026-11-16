# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 147 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 147 | 0.881 | 95.9% | 91.8% | 93.9% | 0/0 | 0 | ĐẠT |
| 101|102 | 13 | 46 | 0.877 | 95.7% | 93.5% | 87.0% | 0/0 | 0 | ĐẠT |
| 101|103 | 13 | 50 | 0.887 | 96.0% | 94.0% | 98.0% | 0/0 | 0 | ĐẠT |
| 102|103 | 14 | 51 | 0.878 | 96.1% | 88.2% | 96.1% | 0/0 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 65 ({'iou_thap': 38, 'sai_thuoc_tinh': 21, 'sai_lop': 6}), trên 33 ảnh cần calibration.
