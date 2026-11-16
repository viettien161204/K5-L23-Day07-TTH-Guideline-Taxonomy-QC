# Guideline gán nhãn Bounding Box 2D — v1.0 — Nhóm TTH

- **Phiên bản:** v1.0 (Bản thống nhất chốt của nhóm TTH)
- **Ngày lập:** 16/11/2026
- **Đại diện nhóm:** Nguyễn Viết Tiến (101 - Điều phối), Phạm Thanh Tùng (102 - Reviewer), Trần Đức Hòa (103)
- **Công cụ áp dụng:** CVAT (định dạng xuất COCO 1.0)
- **Bộ nhãn sử dụng:** 7 lớp đối tượng (`car`, `truck`, `van`, `motorcycle`, `bicycle`, `other_vehicle`, `person`) + 1 vùng bỏ qua (`ignore_region`)

---

## 1. Mục tiêu và Xác định bài toán duy nhất

### 1.1. Bài toán kỹ thuật duy nhất
Tập dữ liệu nhãn bounding box 2D này được xây dựng cho **một bài toán duy nhất**: **Hệ thống hỗ trợ lái xe nâng cao (ADAS / Autonomous Driving cấp độ 2+) nhằm phát hiện và cảnh báo nguy cơ va chạm chướng ngại vật thực tế trên đường phố đô thị**.

- **Mục tiêu nhận diện:** Mô hình cần phát hiện chính xác vị trí và phân loại các đối tượng có nguy cơ va chạm trực tiếp với xe tự hành (người đi bộ, xe máy, xe đạp, ô tô con, xe tải, xe van, phương tiện đặc thù khác) đang lưu thông hoặc dừng/đỗ ở cự ly gần và trung bình (khoảng cách dưới 40–50 mét).
- **Hậu quả của các loại lỗi:**
  - *Bỏ sót đối tượng ở cự ly gần (False Negative — Lỗi chí mạng):* Bỏ sót người đi bộ, người đi xe máy hoặc xe đạp qua đường ở cự ly gần sẽ khiến hệ thống không kịp kích hoạt phanh tự động khẩn cấp (AEB), trực tiếp dẫn đến tai nạn chết người.
  - *Báo thừa trên vật thể tĩnh / giả mạo (False Positive — Gây "phanh ma"):* Gán nhãn các đối tượng vô tri như tượng đài, tranh ảnh in trên biển quảng cáo, ma-nơ-canh, hay bóng phản chiếu qua gương/mặt đường ướt sẽ làm hệ thống học sai và tự động phanh gấp vô cớ, gây tai nạn liên hoàn cho các xe phía sau.
  - *Gán ép nhãn sai lệch:* Việc gán ép xe ba bánh, xích lô vào ô tô hoặc xe máy sẽ gây nhiễu phân phối đặc trưng hình học của mô hình.

### 1.2. Tiêu chí cốt lõi: Đơn giản — Thực dụng — Không khắt khe
Để 3 thành viên gán nhãn nhanh chóng, đồng nhất và đạt độ đồng thuận (IAA) cao:
1. **Ưu tiên tiền cảnh:** Chỉ tập trung gán các đối tượng thật, nhìn thấy rõ ràng, có nguy cơ va chạm ở khu vực tiền cảnh và trung cảnh.
2. **Không khắt khe với vật ở xa hoặc mờ nhòe:** Các vật thể ở xa có chiều cao $< 20\text{ px}$ hoặc bị nhòe chuyển động (motion blur), thiếu sáng không nhìn rõ đường viền $\rightarrow$ **BỎ QUA KHÔNG CẦN GÁN**.
3. **Không khắt khe với mật độ đông / chỗ đông người:** Tại các điểm kẹt xe ngã tư hoặc chỗ đông người chen chúc (`img_005.jpg`, `img_030.jpg`, `img_040.jpg`), **chỉ gán 3–5 đối tượng rõ nhất ở ngay phía trước**. Toàn bộ cụm người/xe chen chúc dính vào nhau phía sau **GOM VÀO `ignore_region` HOẶC BỎ QUA**, không ngồi tỉ mỉ bóc tách từng xe li ti.
4. **Loại trừ dứt khoát:** Không gán tượng, tranh ảnh, ma-nơ-canh, gương phản chiếu, và không bao trùm hàng hóa cồng kềnh chìa ra ngoài thân xe.

---

## 2. Lớp và Thuộc tính

### 2.1. Bảng định nghĩa lớp

Nhóm thống nhất bộ **7 lớp đối tượng** (bổ sung `bicycle` và `other_vehicle` để bao quát toàn diện các phương tiện thực tế) và **1 lớp vùng bỏ qua**:

| Lớp | Tên tiếng Việt | Định nghĩa (chuẩn 1 câu) | Ví dụ tiêu biểu trong 40 ảnh | Phản ví dụ / Tuyệt đối KHÔNG gán |
|---|---|---|---|---|
| `car` | Ô tô con | Ô tô con từ 4 đến 9 chỗ ngồi chở người (sedan, hatchback, SUV, crossover, station wagon). | Ô tô đỗ bên đường (`img_001.jpg`, `img_015.jpg`), các ô tô con chở trên sàn xe tải (`img_002.jpg`), xe wagon (`img_024.jpg`). | Tượng xe bán tải bằng đá (`img_027.jpg`), ô tô đồ chơi trong tủ kính (`img_038.jpg`), xe ba bánh tuk-tuk (`other_vehicle`). |
| `truck` | Xe tải | Xe cơ giới chở hàng có kết cấu thùng hàng riêng biệt, xe đầu kéo, xe chuyên dụng chở xe, xe bán tải thùng hở (pickup). | Xe tải chở ô tô (`img_002.jpg`), xe tải nhỏ (`img_036.jpg`), xe bán tải chở hàng thùng hở (`img_001.jpg`, `img_012.jpg`). | Tượng đài xe tải bằng đá (`img_027.jpg`), xe thùng kín chở khách (`van`), xe lam 3 bánh chở hàng (`other_vehicle`). |
| `van` | Xe van | Xe thùng kín cỡ vừa/nhỏ có thân liền khối một khoang (không tách rời cabin và thùng). | Xe van chuyển phát nhanh (`img_007.jpg`), xe bán tải có nắp thùng cao kín Master 75 (`img_037.jpg`). | Xe tải có cabin và thùng hàng tách rời (`truck`), xe buýt cỡ lớn. |
| `motorcycle` | Xe máy / Mô tô | Phương tiện cơ giới 2–3 bánh chạy bằng động cơ (xe số, xe ga, xe phân khối lớn, xe mô tô có thùng phụ sidecar). | Xe máy đang chạy rõ nét (`img_005.jpg`, `img_026.jpg`), mô tô có thùng phụ sidecar (`img_022.jpg`). | Xe đạp không động cơ (`bicycle`), xích lô đạp chân (`other_vehicle`), xe tuk-tuk (`other_vehicle`). |
| `bicycle` | Xe đạp | Phương tiện thô sơ 2 hoặc 3 bánh vận hành bằng bàn đạp chân của con người, không có động cơ cơ giới. | Xe đạp của người bán hàng rong chở mít (`img_003.jpg`), xe đạp bán quả phật thủ (`img_010.jpg`), xe đạp đi cùng người vác nhôm (`img_029.jpg`). | Xe máy, xe máy điện (`motorcycle`); xe xích lô chở khách (`other_vehicle`). |
| `other_vehicle` | Phương tiện khác | Các phương tiện giao thông đường bộ khác không thuộc các lớp trên: xe ba bánh có động cơ (tuk-tuk, xe lam, auto rickshaw) và xe thô sơ chở khách/hàng đặc thù (xích lô đạp chân). | Xe tuk-tuk màu xanh (`img_004.jpg`, `img_016.jpg`), xích lô chở khách (`img_006.jpg`, `img_009.jpg`, `img_023.jpg`, `img_034.jpg`, `img_039.jpg`), xe auto rickshaw chở hàng (`img_035.jpg`). | Xe ô tô con (`car`), xe máy 2 bánh (`motorcycle`), xe đạp 2 bánh thường (`bicycle`). |
| `person` | Người | Con người thật đang đi bộ, đứng, ngồi hoặc đang điều khiển / ngồi trên bất kỳ phương tiện giao thông nào. | Người đi bộ qua đường (`img_014.jpg`), người lái xe máy (`img_026.jpg`), người bán hàng rong (`img_003.jpg`), người đạp và khách ngồi xích lô (`img_006.jpg`). | Tượng đồng tướng cưỡi ngựa (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), hình người in trên quảng cáo (`img_018.jpg`), búp bê đồ chơi (`img_038.jpg`). |
| `ignore_region` | Vùng bỏ qua | Vùng bỏ qua: khoanh trùm đám đông chen chúc hoặc cụm xe nhỏ ở hậu cảnh xa để mô hình không bị tính lỗi sót. | Cụm xe máy kẹt xe ở xa (`img_030.jpg`), đám đông chờ xe buýt (`img_032.jpg`), dòng xe xa tít (`img_040.jpg`). | Người hoặc xe đứng đơn lẻ nhìn rõ ở cự ly gần. |

**Quy tắc loại trừ vật ngoài thực tế:**
- Tuyệt đối không gán bất kỳ lớp nào cho mô hình xe đồ chơi trong tủ kính (`img_038.jpg`), tượng điêu khắc (`img_017.jpg`, `img_027.jpg`), ảnh in quảng cáo (`img_018.jpg`), hay bóng phản chiếu qua gương (`img_020.jpg`).

---

### 2.2. Bảng thuộc tính định lượng

| Thuộc tính | Các mức cố định | Ý nghĩa & Ngưỡng định lượng cụ thể | Mặc định |
|---|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | **`0`**: Nhìn rõ toàn bộ (che khuất 0%).<br>**`1`**: Che khuất nhẹ (che từ **1% đến 30%** diện tích).<br>**`2`**: Che khuất trung bình/nặng (che từ **31% đến 70%** diện tích).<br>**`3`**: Che khuất rất nặng (che **> 70%** diện tích nhưng vẫn nhận diện chắc chắn 100%). | `"0"` |
| `truncated` | `no`, `yes` | **`no`**: Vật thể nằm trọn vẹn trong khung ảnh.<br>**`yes`**: Bị cắt cụt bởi bất kỳ mép viền nào của bức ảnh (trên, dưới, trái, phải). | `"no"` |
| `can_review` | `true`, `false` | **`false`**: Tự tin với box.<br>**`true`**: Tình huống phân vân, đánh dấu để cả nhóm xem lại trong buổi họp calibration. | `"false"` |

---

## 3. Quy tắc vẽ box (R-01 đến R-10)

- **`R-01` (Ôm sát phần nhìn thấy)**: Box bao sát các mép nhìn thấy thực tế của vật thể. Không vẽ box dự đoán phần vô hình bị khuất sau vật khác; không bao gồm bóng đổ hoặc vệt phản chiếu trên mặt đường.
- **`R-02` (Tách Người và Phương tiện ở cự ly gần)**:
  - **Với xe máy (`motorcycle`)**: Gán **1 box `motorcycle`** ôm thân xe và **1 box `person` cho MỖI người** trên xe (từ đầu đến chân còn nhìn thấy). Xe chở 2 người (`img_026.jpg`) gán 2 `person` + 1 `motorcycle`.
  - **Với xe đạp (`bicycle`)**: Gán **1 box `bicycle`** ôm khung thân và bánh xe đạp; gán **1 box `person`** cho người đạp xe (`img_003.jpg`, `img_010.jpg`).
  - **Với phương tiện khác (`other_vehicle` — xích lô, tuk-tuk hở)**: Gán **1 box `other_vehicle`** ôm toàn bộ thân xe; gán **box `person` cho từng người** (người điều khiển và hành khách) nếu lộ rõ cơ thể ra ngoài không gian mở (`img_004.jpg`, `img_006.jpg`).
- **`R-03` (BỎ QUA VẬT THỂ KHÔNG CÓ SỰ SỐNG)**: Tuyệt đối **KHÔNG GÁN** tượng đồng/đá (`img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), hình người in trên biển quảng cáo (`img_018.jpg`), đồ chơi/mô hình trong tủ kính (`img_038.jpg`).
- **`R-04` (BỎ QUA HÌNH ẢNH PHẢN CHIẾU)**: Tuyệt đối **KHÔNG GÁN** hình ảnh xe cộ hoặc người phản chiếu qua gương chiếu hậu (`img_020.jpg`) hoặc bóng phản chiếu trên mặt đường ướt mưa (`img_013.jpg`).
- **`R-05` (BỎ QUA VẬT Ở XA / NHỎ < 20 px / MỜ NÉT)**: Bất kỳ người hoặc xe nào có **chiều cao box nhỏ hơn 20 pixels** ($h < 20\text{ px}$) hoặc bị nhòe chuyển động (motion blur) không rõ biên dạng: **BỎ QUA KHÔNG GÁN** (`img_014.jpg`, `img_025.jpg`, `img_040.jpg`).
- **`R-06` (KHÔNG KHẮT KHE VỚI MẬT ĐỘ ĐÔNG — DÙNG `ignore_region`)**: Tại các điểm kẹt xe đông đúc hoặc trạm xe buýt tập trung đông người (`img_005.jpg`, `img_030.jpg`, `img_032.jpg`, `img_040.jpg`), **chỉ gán 3–5 đối tượng rõ nhất ở ngay tiền cảnh**. Toàn bộ cụm người/xe dính chùm phía sau vẽ **1–2 box lớn gán `ignore_region`**, không ngồi soi gán từng xe nhỏ lẻ.
- **`R-07` (BỎ QUA HÀNG HÓA CỒNG KỀNH)**: Xe chở hàng cồng kềnh (xe máy chở bao tải xốp `img_029.jpg`, xe tuk-tuk chất bao tải cao vút `img_035.jpg`, xe bán tải chở hàng đầy thùng `img_012.jpg`): **Box phương tiện CHỈ ÔM KHUNG THÂN VỎ CƠ BẢN CỦA XE**, không kéo dãn ôm thùng xốp/bao tải chìa ra ngoài.
- **`R-08` (XE CHỞ XE `img_002.jpg`)**: Gán **1 box `truck`** cho xe tải chở hàng; gán từng box `car` cho các ô tô con trên sàn nhìn rõ $\ge 30\%$ diện tích. Xe bị khuất sâu tầng dưới $> 70\%$ thì bỏ qua.
- **`R-09` (QUY TẮC CHO `bicycle` VÀ `other_vehicle`)**:
  - Gán `bicycle` cho toàn bộ xe đạp 2–3 bánh đạp chân thông thường.
  - Gán `other_vehicle` cho xe tuk-tuk, xe auto rickshaw, xe xích lô du lịch.
- **`R-10` (BỎ QUA CHE KHUẤT QUÁ NẶNG)**: Vật thể bị che $> 70\%$ diện tích mà chỉ lộ 1 mẩu nhỏ rời rạc (1 cánh tay, góc bánh xe): **BỎ QUA KHÔNG GÁN**.

---

## 4. Ca khó tiêu biểu

Bảng các ca tiêu biểu minh họa quy tắc phân lớp rõ ràng, nhất quán:

| Ảnh | Tình huống | Rule ID | Quyết định thống nhất | Lý do kỹ thuật |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chuyên dụng chở 8 ô tô con trên hai tầng sàn | `R-08` | Gán **1 box `truck`** cho xe tải chở hàng; gán từng box **`car`** (`occluded=1` hoặc `2`) cho các xe con trên sàn nhìn rõ $\ge 30\%$. | Ô tô con trên sàn là vật thể thật kích thước lớn, xe tự hành cần nhận diện để tránh va chạm. |
| `img_003.jpg` | Người phụ nữ bán mít bằng xe đạp chở nặng | `R-02`, `R-09` | Gán **1 box `bicycle`** cho xe đạp; gán **1 box `person`** cho người phụ nữ bán hàng. Không gán thúng mít. | Phân định đúng lớp xe đạp (`bicycle`) và tách riêng người điều khiển (`person`). |
| `img_004.jpg` | Hai xe tuk-tuk màu xanh đỗ trên vỉa hè ban đêm | `R-02`, `R-09` | Gán **2 box `other_vehicle`** cho 2 xe tuk-tuk; tài xế ngồi trong cabin hở gán `person` nếu nhìn rõ. | Xe tuk-tuk thuộc lớp phương tiện khác (`other_vehicle`), không gán ép vào `car` hay `motorcycle`. |
| `img_006.jpg` | Đoàn xích lô du lịch chở khách trên đường phố Huế | `R-02`, `R-09` | Gán **box `other_vehicle`** cho từng chiếc xích lô; gán **box `person`** riêng cho người đạp và khách ngồi. | Xích lô chở khách là phương tiện đặc thù thuộc `other_vehicle`. |
| `img_017.jpg` | Tượng đồng tướng Duke of Wellington cưỡi ngựa tại Glasgow | `R-03` | **TUYỆT ĐỐI KHÔNG GÁN BOX** (0 annotation cho cả người và ngựa). | Tượng kim loại tĩnh không tham gia giao thông; gán sẽ là lỗi thừa (`thua`) gây phanh ma nguy hiểm. |
| `img_020.jpg` | Xe cộ và cảnh quan phản chiếu qua gương chiếu hậu ô tô | `R-04` | **TUYỆT ĐỐI KHÔNG GÁN BOX** các vật thể trong gương. | Hình ảnh phản chiếu ảo, không có chướng ngại vật vật lý tại vị trí mặt gương. |
| `img_026.jpg` | Cặp đôi đèo nhau trên xe máy trong hầm chui | `R-02` | Gán **2 box `person`** riêng biệt cho 2 người; gán **1 box `motorcycle`** ôm toàn bộ thân xe. | Chuẩn hóa quy tắc tách người và xe máy rõ nét ở cự ly gần; box người ôm từ đầu đến chân còn nhìn thấy. |
| `img_029.jpg` | Xe máy chở bao tải xốp cồng kềnh đi cạnh người đi xe đạp vác nhôm | `R-02`, `R-07`, `R-09` | Gán 1 `motorcycle` **chỉ ôm khung/bánh xe** (không ôm bao tải); gán 1 `person` cho người lái xe máy. Với xe đạp bên cạnh: gán 1 `bicycle` và 1 `person`. | Box xe không phình to theo hàng hóa cồng kềnh; tách đúng xe đạp và xe máy. |
| `img_030.jpg` | Ngã tư đường phố đặc kín hàng trăm xe máy chen chúc | `R-02`, `R-06` | Gán 4–5 xe máy và người nhìn rõ nhất ở tiền cảnh. Toàn bộ dòng xe chen chúc phía sau: khoanh 1–2 box **`ignore_region`**. | Tiết kiệm công sức, không khắt khe với nơi kẹt xe đông đúc, đảm bảo độ phủ và chất lượng nhãn. |
| `img_035.jpg` | Xe tuk-tuk / auto rickshaw chở bao tải chất cao ngất trên nóc | `R-07`, `R-09` | Gán 1 box **`other_vehicle`** ôm khung thân xe tuk-tuk. **KHÔNG KÉO BOX TRÙM BAO TẢI TRÊN NÓC**. | Phân lớp đúng `other_vehicle`; giữ nguyên tỷ lệ khung hình xe để tránh tụt IoU. |

---

## 5. QC và Lịch sử thống nhất nhóm

### 5.1. Ngưỡng QC nghiệm thu
- **Độ phủ phân công**: Đủ $100\%$ ($80/80$ lượt gán cho 40 ảnh với $K=2$).
- **Kích thước tối thiểu**: $100\%$ box có chiều cao $\ge 20\text{ px}$ (trừ `ignore_region`).
- **Độ đồng thuận IAA (IoU ghép $\ge 0.5$)**: IoU trung bình $\ge 0.85$; Thống nhất phân lớp $\ge 95\%$.
- **Tỉ lệ lỗi QC toàn lô**: Tỉ lệ ảnh có lỗi $\le 2.0\%$ (trên 40 ảnh đã duyệt, chỉ cần $\ge 1$ ảnh lỗi là vượt ngưỡng).

### 5.2. Lịch sử thống nhất và giải quyết bất đồng cá nhân (v0 $\rightarrow$ v1.0)

| Phiên bản | Nội dung thống nhất | Lý do & Giải quyết bất đồng giữa các thành viên |
|---|---|---|
| **v0** | Bản khởi đầu của ban tổ chức (5 lớp định tính sơ sài, bỏ ngỏ xe đạp, xích lô, tuk-tuk). | Chưa đủ dùng; người gán lúng túng khi gặp xích lô, xe đạp, tuk-tuk (không biết gán vào đâu hay bỏ qua). |
| **v1.0** | - **Chốt 1 bài toán duy nhất**: ADAS phát hiện và cảnh báo va chạm chướng ngại vật.<br>- **Bổ sung 2 lớp mới**: `bicycle` (xe đạp) và `other_vehicle` (xe khác: xích lô, tuk-tuk, xe ba bánh).<br>- **Bộ nhãn hoàn chỉnh**: 7 lớp đối tượng (`car`, `truck`, `van`, `motorcycle`, `bicycle`, `other_vehicle`, `person`) + 1 vùng bỏ qua (`ignore_region`).<br>- **Quy định nới lỏng thực dụng**: Bỏ qua xe/người ở xa ($< 20\text{ px}$), mờ nhòe; cho phép dùng `ignore_region` cho các điểm kẹt xe, chỗ đông người.<br>- **Định lượng ngưỡng `occluded_level`**: 0%, 1–30%, 31–70%, >70%.<br>- **Ban hành 10 mã quy tắc `R-01` đến `R-10`**. | - **Giải quyết bất đồng về xe ba bánh và xe đạp**: Trước đó bản của Hòa đề xuất không gán xe đạp/xích lô, bản của 102 đề xuất thêm `tuk_tuk`. Nhóm thống nhất: bổ sung lớp `bicycle` cho xe đạp và lớp bao quát `other_vehicle` cho toàn bộ xích lô, tuk-tuk, xe lam. Điều này giúp không bỏ sót phương tiện thật trên đường mà không làm vỡ cấu trúc taxonomy.<br>- **Thống nhất về hàng cồng kềnh (`img_029.jpg`, `img_035.jpg`)**: Box của `motorcycle` hay `other_vehicle` chỉ ôm khung xe, không bao hàng hóa nhô cao.<br>- **Thống nhất về đối tượng ở xa/đám đông (`img_030.jpg`, `img_040.jpg`)**: Nâng ngưỡng lên $\ge 20\text{ px}$ và dùng `ignore_region` cho hậu cảnh đông đúc để giảm tải tối đa cho nhóm 3 người. |
