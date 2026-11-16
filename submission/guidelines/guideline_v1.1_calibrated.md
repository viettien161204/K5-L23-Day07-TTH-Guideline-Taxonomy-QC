# Guideline gán nhãn Bounding Box 2D — v1.1 (Calibrated) — Nhóm TTH

- **Phiên bản:** v1.1 (Bản hiệu chuẩn sau calibration của nhóm TTH, đã rà mâu thuẫn)
- **Ngày lập:** 16/11/2026
- **Thành viên:** 101 (điều phối), 102 (review bản cuối), 103
- **Công cụ áp dụng:** CVAT (xuất COCO 1.0)
- **Bộ nhãn:** 8 lớp đối tượng (`car`, `truck`, `van`, `bus`, `motorcycle`, `bicycle`, `other_vehicle`, `person`) + 1 vùng bỏ qua (`ignore_region`)

---

## 1. Mục tiêu

### 1.1. Bài toán duy nhất
Bộ nhãn bounding box 2D này phục vụ **một bài toán duy nhất**: hệ thống hỗ trợ lái xe nâng cao (ADAS) phát hiện và cảnh báo nguy cơ va chạm với vật cản thật trên đường phố đô thị, ở cự ly gần và trung bình (dưới khoảng 40–50 m).

Mô hình **cần đoán**: vị trí và lớp của người và phương tiện thật đang lưu thông hoặc dừng đỗ.
Mô hình **không cần đoán**: tượng, ma-nơ-canh, hình in, đồ chơi, hình phản chiếu, và các thông tin chỉ để lọc mẫu (mức che khuất, bị cắt mép, cờ cần xem lại), những thứ này thuộc thuộc tính chứ không phải lớp.

Hậu quả của từng loại lỗi:
- **Bỏ sót vật thật ở gần** (người, xe máy, xe đạp): hệ thống không phanh kịp, nguy hiểm nhất.
- **Gán thừa vật giả** (tượng, hình in, phản chiếu): hệ thống học sai và "phanh ma".
- **Gán sai lớp** (ví dụ tuk-tuk thành `car`): làm nhiễu đặc trưng hình học mà mô hình học.

### 1.2. Nguyên tắc chung
1. Chỉ gán vật **thật**, nhìn thấy rõ, đủ lớn (ngưỡng X ở R-05).
2. Một cách xử lý duy nhất cho mỗi tình huống, không có "A hoặc B". Ca lưỡng lự thì tick `can_review`, không tự đoán.
3. Vật ở xa/nhỏ/đám đông dày: xử lý theo R-05 và R-06, không bóc tách từng vật li ti.
4. Loại trừ dứt khoát vật không có thật (R-03, R-04) và không ôm hàng chìa ra ngoài thân xe (R-07).

---

## 2. Lớp và thuộc tính

### 2.1. Bảng định nghĩa lớp

| Lớp | Định nghĩa (1 câu) | Ví dụ | Phản ví dụ (KHÔNG gán lớp này) |
|---|---|---|---|
| `car` | Ô tô con 4–9 chỗ, thân kín, cabin liền khối (sedan, hatchback, SUV, wagon). | Các ô tô con trên sàn xe tải (`img_002.jpg`), xe wagon (`img_024.jpg`). | Tượng xe bán tải (`img_027.jpg`), ô tô đồ chơi trong tủ kính (`img_038.jpg`), tuk-tuk (`other_vehicle`). |
| `truck` | Xe tải hoặc bán tải chở hàng có cabin tách rời khỏi thùng hàng (thùng hở hoặc thùng có nắp), xe đầu kéo, xe chuyên dụng chở xe. | Xe tải chở ô tô (`img_002.jpg`), bán tải thùng hở (`img_001.jpg`, `img_012.jpg`), xe tải nhỏ (`img_036.jpg`). | Tượng xe tải (`img_027.jpg`), xe thân liền khối kín (`van`), xe lam 3 bánh (`other_vehicle`). |
| `van` | Xe thùng kín cỡ nhỏ hoặc vừa, thân liền khối một khoang, không thấy vách ngăn cabin–thùng từ bên ngoài. | Xe van chuyển phát (`img_007.jpg`), xe Master 75 (`img_037.jpg`, ca cần hỏi reviewer, xem mục 4). | Xe có cabin và thùng tách rời (`truck`), xe buýt (`bus`). |
| `bus` | Xe buýt hoặc xe khách cỡ lớn chở nhiều người, thân dài một khối với dãy cửa sổ hành khách liên tiếp (từ 3 ô cửa sổ hành khách trở lên trên một bên thân xe). | Xe buýt (`img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg`). | Xe van thân ngắn ít cửa sổ (`van`), xe tải (`truck`). |
| `motorcycle` | Xe 2 bánh có động cơ (xe số, xe ga, mô tô, xe máy điện, xe đạp điện) hoặc xe 2 bánh có sidecar. | Xe máy (`img_005.jpg`, `img_026.jpg`), mô tô sidecar (`img_022.jpg`). | Xe đạp không động cơ (`bicycle`), xích lô và tuk-tuk (`other_vehicle`). |
| `bicycle` | Xe đạp 2 bánh, không động cơ, không có thùng hoặc ghế khách gắn cố định (có thể chở hàng đặt trên yên hoặc giỏ). | Xe đạp chở mít (`img_003.jpg`), xe đạp bán quả phật thủ (`img_010.jpg`), xe đạp bên cạnh xe máy (`img_029.jpg`). | Xe có động cơ (`motorcycle`), xe đạp 3 bánh chở hàng và xích lô (`other_vehicle`). |
| `other_vehicle` | Xe 3 bánh có cabin, thùng hàng hoặc ghế khách gắn cố định: tuk-tuk, xe lam, auto rickshaw, xích lô đạp chân, xe đạp 3 bánh chở hàng. | Tuk-tuk (`img_004.jpg`, `img_016.jpg`), auto rickshaw chở hàng (`img_035.jpg`), xích lô (`img_006.jpg`, `img_023.jpg`, `img_034.jpg`, `img_039.jpg`). | Ô tô con (`car`), xe máy 2 bánh (`motorcycle`), xe đạp 2 bánh (`bicycle`). |
| `person` | Người thật đi bộ, đứng, hoặc ngồi/đứng trên phương tiện **hở** (xe máy, xe đạp, xích lô, tuk-tuk hở). | Người đi xe máy (`img_026.jpg`), người bán hàng rong (`img_003.jpg`), người đạp và khách xích lô (`img_006.jpg`). | Tượng đồng (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), hình người trên quảng cáo (`img_018.jpg`), búp bê (`img_038.jpg`), người ngồi trong cabin kín. |
| `ignore_region` | Vùng bỏ qua: 1 box ôm cụm người/xe nhỏ ở hậu cảnh mà không gán riêng từng vật (R-06). | Cụm xe máy kẹt xe xa (`img_030.jpg`), đám đông chờ xe buýt (`img_032.jpg`). | Người hoặc xe đơn lẻ, nhìn rõ, cao từ X trở lên. |

### 2.2. Bảng thuộc tính

| Thuộc tính | Giá trị | Ngưỡng định lượng | Mặc định |
|---|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | `0`: nhìn rõ toàn bộ (che 0%). `1`: che 1–30% diện tích. `2`: che 31–70%. `3`: **không dùng** trong lô này, vì vật che hơn 70% bị bỏ theo R-10. Ước lượng bằng mắt trên toàn vật; ca sát ngưỡng (±5%) thì tick `can_review`. | `0` |
| `truncated` | `no`, `yes` | `yes` khi box chạm hoặc bị cắt bởi mép ảnh (trên, dưới, trái, phải); còn lại `no`. | `no` |
| `can_review` | `true`, `false` | `true` khi: sát ngưỡng che khuất (±5%), chiều cao box 35–45 px (sát ngưỡng X), không chắc vật thật hay giả, phân vân giữa hai lớp. Còn lại `false`. | `false` |

---

## 3. Quy tắc vẽ box (R-01 đến R-11)

- **`R-01` (Ôm sát phần nhìn thấy)**: Box ôm sát mép nhìn thấy của vật. Tính cả gương chiếu hậu và phần thân xe; **không** tính ăng-ten, cờ, dây treo, bóng đổ, vệt phản chiếu trên đường. Không vẽ phần dự đoán bị khuất.
- **`R-02` (Tách người và phương tiện)**:
  - Xe máy: 1 box `motorcycle` ôm thân xe + 1 box `person` cho **mỗi** người ngồi trên xe. `img_026.jpg`: 2 `person` + 1 `motorcycle`.
  - Xe đạp: 1 box `bicycle` + 1 box `person` cho người đạp.
  - Xích lô, tuk-tuk hở: 1 box `other_vehicle` + 1 box `person` cho mỗi người **lộ rõ**, tức là thấy ít nhất đầu và vai.
  - Box `person` **được phép chồng** lên box xe (chân hoặc thân bị xe che một phần vẫn ôm từ đầu đến phần chân còn nhìn thấy).
  - Người ngồi trong cabin kín (ô tô, xe buýt, tuk-tuk có kính che): **không gán** `person`.
- **`R-03` (Vật không có sự sống)**: **Không gán** tượng đồng/đá (`img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), hình người in trên quảng cáo (`img_018.jpg`), đồ chơi hoặc mô hình trong tủ kính (`img_038.jpg`). Không chắc là thật hay giả thì tick `can_review`, không tự đoán.
- **`R-04` (Hình phản chiếu)**: **Không gán** mọi hình phản chiếu trên bề mặt phản chiếu: gương chiếu hậu (`img_020.jpg`), kính cửa hàng, kính xe, mặt nước hoặc mặt đường ướt (`img_013.jpg`).
- **`R-05` (Vật nhỏ, ngưỡng X)**: **X = 40 px** chiều cao box, đo trên ảnh gốc (ước lượng thô: khoảng 1/25 chiều cao ảnh khi ảnh cao 1024 px). Vật có chiều cao box **dưới X thì KHÔNG gán**. Hiểu là vật không phân biệt nổi lớp (kể cả do nhòe chuyển động) thì cũng không gán. Vật cao 35–45 px thì nếu gán, tick `can_review`.
- **`R-06` (Đám đông và `ignore_region`)**: Gán **mọi** vật có chiều cao ≥ X và che ≤ 70%. Phần còn lại xử lý như sau:
  - Cụm từ **3 vật trở lên** nhỏ hơn X (hoặc bị che > 70%) nằm sát nhau: vẽ **1 box `ignore_region`** ôm cả cụm; hai cụm tách biệt thì hai box.
  - Vật nhỏ lẻ đơn, không thuộc cụm: bỏ qua, **không** vẽ `ignore_region`.
  - `ignore_region` **không được phủ lên** box vật đã gán (vật cao ≥ X nằm trong vùng thì giữ nguyên box của vật, vẽ ignore tránh vật đó).
  - Áp dụng cho `img_005.jpg`, `img_030.jpg`, `img_032.jpg`, `img_040.jpg`.
- **`R-07` (Hàng hóa cồng kềnh)**: Box xe **ôm thân xe và thùng xe**, **không** ôm hàng chìa ra ngoài thùng/khung xe (bao tải xốp `img_029.jpg`, bao tải chất cao trên tuk-tuk `img_035.jpg`, hàng đầy thùng `img_012.jpg`). Hàng che một phần xe thì mức `occluded_level` tính trên chính xe.
- **`R-08` (Xe chở xe, `img_002.jpg`)**: 1 box `truck` cho xe tải + 1 box `car` cho **từng** ô tô con trên sàn có **phần nhìn thấy ≥ 30%** so với ước lượng toàn xe (tương đương che ≤ 70%). Mức `occluded_level` ghi theo ước lượng thực tế của từng xe (thường là `1` hoặc `2`). Xe nhìn thấy dưới 30% thì bỏ (R-10).
- **`R-09` (Phân loại xe 2–3 bánh, thứ tự xét)**:
  1. 2 bánh có động cơ, hoặc có sidecar (kể cả xe máy điện, xe đạp điện) → `motorcycle`.
  2. 2 bánh không động cơ, không thùng/ghế khách → `bicycle`.
  3. 3 bánh có cabin, thùng hoặc ghế khách gắn cố định (tuk-tuk, xe lam, xích lô, xe đạp 3 bánh chở hàng) → `other_vehicle`.
- **`R-10` (Che khuất quá nặng)**: Vật bị che **hơn 70%** diện tích (phần nhìn thấy dưới 30%): **bỏ, không gán**. Đây là ngưỡng duy nhất cho cả R-08 và `occluded_level`. Ca sát ngưỡng (65–75%) thì nếu gán, tick `can_review`.
- **`R-11` (Phân loại xe lớn: `truck` / `van` / `bus`)**: Nhìn từ bên ngoài:
  1. Dãy cửa sổ hành khách liên tiếp, thân dài một khối → `bus`.
  2. Có vách ngăn cabin–thùng nhìn thấy, hoặc cabin nhô rõ khỏi thùng (kể cả bán tải có nắp thùng) → `truck`.
  3. Thân liền khối một khoang, thùng kín, không vách ngăn nhìn thấy → `van`.
  Phân vân giữa hai lớp thì chọn theo thứ tự trên và tick `can_review`.

---

## 4. Ca khó tiêu biểu

| Ảnh | Tình huống | Rule | Quyết định thống nhất | Lý do | Loại |
|---|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở 8 ô tô con trên hai sàn | `R-08`, `R-10` | 1 `truck` + từng `car` nhìn thấy ≥ 30%, `occluded_level` theo thực tế. | Xe trên sàn là vật thật cần tránh; ngưỡng 30% nhìn thấy tránh hai người chọn khác nhau. | Loại theo quy tắc |
| `img_003.jpg` | Người bán mít bằng xe đạp chở nặng | `R-02`, `R-09` | 1 `bicycle` + 1 `person`; không gán thúng mít. | Xe đạp 2 bánh không thùng cố định → `bicycle`; hàng không ôm vào box. | Loại theo quy tắc |
| `img_004.jpg` | Hai tuk-tuk xanh đỗ vỉa hè ban đêm | `R-02`, `R-09` | 2 `other_vehicle`; tài xế gán `person` chỉ khi lộ rõ đầu và vai. | Tuk-tuk 3 bánh có cabin riêng → `other_vehicle`. | Loại theo quy tắc |
| `img_006.jpg` | Đoàn xích lô du lịch | `R-02`, `R-09` | Mỗi xích lô 1 `other_vehicle`; người đạp và khách mỗi người 1 `person`. | Xích lô 3 bánh có ghế khách gắn cố định → `other_vehicle`. | Loại theo quy tắc |
| `img_009.jpg`, `img_031.jpg`, `img_040.jpg` | Xe buýt cỡ lớn | `R-11` | 1 `bus` cho mỗi xe buýt nhìn thấy ≥ 30%. | Dãy cửa sổ hành khách, thân dài một khối → `bus`, không gộp vào `truck`. | Loại theo quy tắc |
| `img_037.jpg` | Xe Master 75 thùng cao kín (cạnh xe buýt) | `R-11` | `van` nếu thân liền khối một khoang, không thấy vách cabin–thùng; nếu thấy vách thì `truck`. Tick `can_review`. | Ranh giới `van` và `truck`, dễ lệch giữa hai người gán. | Cần hỏi reviewer |
| `img_017.jpg` | Tượng đồng tướng cưỡi ngựa | `R-03` | Không gán (0 annotation, cả người và ngựa). | Vật tĩnh không tham gia giao thông; gán là lỗi thừa. | Loại theo quy tắc |
| `img_020.jpg` | Xe cộ phản chiếu qua gương chiếu hậu | `R-04` | Không gán vật trong gương. | Hình ảo, không có vật cản thật ở vị trí đó. | Loại theo quy tắc |
| `img_033.jpg`, `img_038.jpg` | Ma-nơ-canh, đồ chơi trong tủ kính | `R-03` | Không gán, ảnh có 0 annotation nhưng vẫn giữ trong Task CVAT. | Vật không có sự sống, không phải vật cản thật. | Loại theo quy tắc |
| `img_026.jpg` | Cặp đôi đèo nhau trên xe máy trong hầm chui | `R-02` | 2 `person` + 1 `motorcycle`; box `person` được phép chồng box xe. | Tách người và xe, box người ôm đến phần chân còn thấy. | Loại theo quy tắc |
| `img_029.jpg` | Xe chở bao tải xốp cạnh người đi xe đạp vác nhôm | `R-02`, `R-07`, `R-09` | Xe máy: 1 `motorcycle` (không ôm bao tải) + 1 `person`; xe đạp: 1 `bicycle` + 1 `person`. | Box không phình theo hàng; phân loại xe theo R-09. | Cần hỏi reviewer (xác nhận lại cảnh trong ảnh) |
| `img_030.jpg` | Ngã tư kẹt hàng trăm xe máy | `R-05`, `R-06` | Gán mọi vật cao ≥ 40 px và che ≤ 70%; cụm vật nhỏ phía sau vẽ `ignore_region` ôm cụm, không phủ lên box đã gán. | Một tiêu chí đo được, hai người chọn cùng tập vật. | Loại theo quy tắc |
| `img_035.jpg` | Auto rickshaw chở bao tải cao trên nóc | `R-07`, `R-09` | 1 `other_vehicle` ôm thân xe, không ôm bao tải. | Ôm cả khối hàng làm tụt IoU giữa hai người gán. | Loại theo quy tắc |
| `img_014.jpg`, `img_015.jpg` | Ảnh nhòe chuyển động | `R-05` | Vật không phân biệt nổi lớp thì không gán; vật còn rõ thì xét theo ngưỡng X. | "Mờ" đo bằng khả năng phân biệt lớp, không bằng cảm nhận. | Cần hỏi reviewer |

---

## 5. QC và lịch sử thống nhất

### 5.1. Ngưỡng nghiệm thu
- **Độ phủ phân công:** đủ 100% (80/80 lượt gán cho 40 ảnh, K=2).
- **Kích thước tối thiểu:** mọi box vật thể có chiều cao ≥ X = 40 px (trừ `ignore_region`).
- **IAA (ghép IoU ≥ 0.5):** IoU trung bình ≥ 0.85; thống nhất lớp ≥ 95%.
- **Tỉ lệ lỗi QC toàn lô:** ảnh có lỗi ≤ 2.0% (trên 40 ảnh, chỉ cần 1 ảnh lỗi là vượt ngưỡng).

### 5.2. Lịch sử v0 → v1.0

| Phiên bản | Nội dung | Lý do |
|---|---|---|
| **v0** | Bản khởi đầu của ban tổ chức: 5 lớp, thuộc tính mô tả định tính. | Không đủ dùng với xe đạp, xích lô, tuk-tuk, xe buýt, tượng, phản chiếu. |
| **v1.0** | Chốt bài toán ADAS. Thêm 3 lớp `bicycle`, `other_vehicle`, `bus`. Định lượng `occluded_level` (0 / 1–30% / 31–70% / bỏ khi >70%). Một ngưỡng nhỏ X = 40 px và quy tắc `ignore_region` đo được. Ban hành `R-01` đến `R-11`. | Bản của 103 đề xuất không gán xe đạp và xích lô, bản của 102 đề xuất thêm lớp riêng cho tuk-tuk. Nhóm chốt `bicycle` cho xe đạp 2 bánh và `other_vehicle` cho xe 3 bánh. Hai lớp `motorcycle` và `other_vehicle` được tách theo số bánh và cabin/thùng. Ngưỡng che khuất dùng một con số duy nhất (70%) cho R-08, R-10 và `occluded_level`. |
