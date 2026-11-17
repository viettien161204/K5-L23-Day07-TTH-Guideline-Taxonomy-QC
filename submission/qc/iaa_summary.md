# Tóm tắt IAA (taxonomy v1.0)

- Ảnh được so sánh (có >=2 người gán): **40**; số người gán: 3
- Lượt gán đủ **80/80**.
- IoU trung bình và % cùng lớp **chỉ tính trên các cặp box đã ghép** (IoU >= 0.5, ghép 1-1 không phân biệt lớp). Box chưa ghép được thống kê riêng ở cột bỏ sót/thừa.
- Mẫu số: 156 cặp box ghép (tổng mọi cặp người gán, ảnh chung).

| Cặp | Ảnh | Cặp box ghép | IoU TB | Cùng lớp | Cùng occluded_level | Cùng truncated | Chưa ghép (A/B) | Bỏ qua do ignore | Đạt IAA |
|---|---|---|---|---|---|---|---|---|---|
| **TỔNG** | 40 | 156 | 0.953 | 96.2% | 96.2% | 96.8% | 0/0 | 0 | ĐẠT |
| 101|102 | 13 | 48 | 0.961 | 95.8% | 95.8% | 95.8% | 0/0 | 0 | ĐẠT |
| 101|103 | 13 | 65 | 0.948 | 96.9% | 96.9% | 98.5% | 0/0 | 0 | ĐẠT |
| 102|103 | 14 | 43 | 0.953 | 95.3% | 95.3% | 95.3% | 0/0 | 0 | ĐẠT |

Ngưỡng IAA: IoU TB >= 0.85, đồng thuận lớp >= 95%.

Số ca bất thường: 24 ({'sai_thuoc_tinh': 11, 'sai_lop': 6, 'iou_thap': 7}), trên 20 ảnh cần calibration.
