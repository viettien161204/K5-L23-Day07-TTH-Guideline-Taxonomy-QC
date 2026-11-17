# Báo cáo QC — Nhóm TTH

## 1. Mục đích, dữ liệu và phiên bản

- Mục đích và giới hạn sử dụng: Đặc tả và QC nhãn bounding box 2D cho bài toán Phát hiện đối tượng 2D (2D Object Detection) phục vụ ADAS và camera giám sát giao thông đô thị. Giới hạn đại diện: 40 ảnh chụp từ camera giao thông, phản ánh điều kiện đường phố Việt Nam và một số bối cảnh đô thị quốc tế.
- Nguồn và phép sử dụng dữ liệu: Xem data/ATTRIBUTION.csv để biết tác giả, giấy phép và URL nguồn của từng ảnh.
- Manifest và checksum nhãn cuối: batch_manifest.json và SHA-256 ghi nhận đầy đủ cho toàn bộ 40 ảnh và 156 annotation.
- Guideline và taxonomy khi gán / khi bàn giao: Khi gán áp dụng guideline_v1.0_agreed.md và taxonomy.json (v1.0, 8 lớp); khi bàn giao áp dụng guideline_v1.1_calibrated.md và taxonomy_v1.1.json (v1.1, 9 lớp — bổ sung lớp bus). Quyết định phạm vi toàn lô do thay đổi schema taxonomy.

## 2. Phân công và đồng thuận trước calibration

- Lượt hoàn thành / được giao: 80/80 (100%). Thành viên 101: 26 ảnh; 102: 27 ảnh; 103: 27 ảnh.
- Số cặp box ghép, IoU TB, đồng thuận lớp và mẫu số: 156 cặp box ghép, IoU TB = 0.953, Đồng thuận lớp = 96.2%, Cùng occluded_level = 96.2%, Cùng truncated = 96.8% (Mẫu số: 156 cặp).
- Box chưa ghép, loại do ignore, bất đồng vùng ignore: Box chưa ghép 0/0; loại do ignore: 0; bất đồng vùng ignore: 0.
- Ca hai người có thể cùng sai sau khi đối chiếu tham chiếu: 20 ảnh flagged với 24 ca bất thường gồm 6 ca sai lớp, 11 ca sai thuộc tính (tranh chấp occluded_level 0 vs 1 và truncated yes vs no ở cạnh ảnh), và 7 ca IoU thấp (vật ở cự ly xa hoặc box nhỏ dưới 50px). Đã xử lý toàn bộ qua phiên calibration.

## 3. Calibration và kiểm lại

- Ca đã xử lý, rule ID, bản sửa và người xác nhận: 24 ca (C001–C024) trong calibration_decisions.csv. Người chốt: 101, phiếu sửa: 101, kiểm lại: 102.
- Tác động v1.1 lên nhãn và ảnh: Bổ sung lớp bus vào taxonomy; không có đối tượng nào trong 40 ảnh cần đổi sang bus; quyết định phạm vi toàn lô (lo).
- Tham chiếu bằng chứng khi Rework: Không có vòng Rework — QC vòng 1 đạt Release ngay.

## 4. Kết quả duyệt

- Vòng QC: 1
- Người review và ngày: 102, 17/11/2026
- Đã duyệt / cả lô: 40/40 ảnh (100%)
- Ảnh lỗi / ảnh đã duyệt: 0/40 = 0.0%
- Annotation đối tượng hiện có bị lỗi / annotation đối tượng đã duyệt: 0/156 = 0.0%
- Vật bỏ sót, ảnh và vùng: Không có vật bỏ sót (0).
- Ca khó / phần còn lại, lỗi lặp và ví dụ: Ca khó (20 ảnh flagged): 20/20 ảnh lỗi 0/20 = 0.0%; phần còn lại (20 ảnh): 20/20 ảnh lỗi 0/20 = 0.0%. Không có lỗi lặp lại.
- Một lỗi máy chặn được và một lỗi cần người đọc ảnh: Máy chặn được box vượt biên ảnh hoặc cao dưới 20px; người đọc ảnh phân biệt tượng điêu khắc (img_017, img_027) hoặc đồ chơi trong tủ kính (img_038) với người và xe thật.

## 5. Quyết định bàn giao

- Quyết định: Release
- Lý do, người nhận và phạm vi sử dụng: IAA đạt chuẩn (IoU TB 0.953 >= 0.85; Đồng thuận lớp 96.2% >= 95%; cả 3 cặp đạt độc lập). QC vòng 1 toàn bộ 40 ảnh đạt tỷ lệ lỗi 0.0%. Bàn giao bộ nhãn labels_calibrated.json cho nhóm phát triển mô hình 2D Object Detection.
- Nếu còn việc: Không có việc tồn đọng.
- Kiểm ngược từ báo cáo tới ảnh, rule và nhãn: Ca C003 (img_004.jpg, other_vehicle vs car), mở work/viz/img_004_disagreement.png kiểm tra đối tượng tuk-tuk tại x=72, y=76 chốt other_vehicle theo quy tắc R-02 do là phương tiện ba bánh tuk-tuk. Trong labels_calibrated.json đối tượng id 9 mang category_id: 7 (other_vehicle v1.1).
