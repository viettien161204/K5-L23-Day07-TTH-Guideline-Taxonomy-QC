# Guideline gán nhãn — v0.1 — 101

- Phiên bản: v0.1 (Bản cá nhân Pha 1 — Tinh giản, dễ thực hiện)
- Ngày: 16/11/2026
- Người viết / duyệt: 101 (Nguyễn Viết Tiến)

---

## 1. Mục tiêu

### 1.1. Mô hình sử dụng và Bài toán kỹ thuật
Nhãn bounding box 2D này dùng để huấn luyện và đánh giá mô hình **Phát hiện đối tượng 2D (2D Object Detection)** phục vụ:
- **Hệ thống hỗ trợ lái xe an toàn (ADAS / Xe tự hành cấp độ 2+)**: Phát hiện các chướng ngại vật thực tế đang di chuyển trên đường (người đi bộ, xe máy, ô tô, xe tải) ở cự ly gần và trung bình để kịp thời phanh khẩn cấp (AEB) hoặc giữ khoảng cách an toàn.
- **Camera giám sát giao thông đô thị**: Đếm lưu lượng và nhận diện phương tiện cơ bản tại các nút giao.

### 1.2. Sai ở đâu thì nguy hiểm?
- **Bỏ sót đối tượng ở gần (False Negative — Cực kỳ nguy hiểm)**: Nếu bỏ sót người đi bộ qua đường hoặc xe máy cắt ngang ở cự ly gần (< 30 mét), hệ thống sẽ không phanh, gây tai nạn chết người.
- **Báo thừa trên vật tĩnh / giả mạo (False Positive — Gây phanh ma / Phantom Braking)**: Nếu gắn nhãn tượng đài, hình người in trên biển quảng cáo, ma-nơ-canh hoặc bóng phản chiếu, xe tự hành sẽ phanh gấp vô cớ giữa đường, gây tai nạn liên hoàn cho các xe phía sau.
- **Nhầm lẫn lớp**: Nhầm người với xe máy hoặc xe con với xe tải lớn sẽ làm hệ thống tính toán sai lệch quán tính và khoảng cách dừng xe.

### 1.3. Tiêu chí cốt lõi cho nhóm 3 người: Đơn giản — Dễ hiểu — Không khắt khe
Vì nhóm chỉ có 3 người nên quy trình gán nhãn phải nhanh gọn, thực tế và tránh gây mệt mỏi:
- **Tập trung vào tiền cảnh**: Chỉ gán các đối tượng thực sự rõ ràng, ở gần máy quay (tiền cảnh và trung cảnh).
- **Không khắt khe với vật ở xa, mờ nhòe**: Các đối tượng ở xa, mờ tối do thiếu sáng, nhòe do chuyển động (motion blur) mà mắt thường khó nhận rõ đường biên $\rightarrow$ **BỎ QUA KHÔNG CẦN GÁN**.
- **Không khắt khe với mật độ đông / chỗ đông người**: Tại các ngã tư kẹt xe, trạm dừng xe buýt đông đúc, chỉ gán vài đối tượng rõ nét nhất ở phía trước; toàn bộ đám đông san sát nhau phía sau **GOM VÀO `ignore_region` HOẶC BỎ QUA**, không ngồi soi gán từng người hay từng chiếc xe nhỏ li ti.
- **Bỏ qua hoàn toàn ca khó phi thực tế**: Tượng, tranh quảng cáo, ma-nơ-canh, bóng phản chiếu, vật bị che khuất $> 70\%$, hoặc vật có chiều cao $< 20\text{ px}$ $\rightarrow$ **KHÔNG GÁN**.

---

## 2. Lớp và thuộc tính

### Lớp
Mỗi vật thể thuộc đúng một lớp duy nhất:

| Lớp | Định nghĩa (đơn giản, dễ nhớ) | Ví dụ | Phản ví dụ / Tuyệt đối KHÔNG gán |
|---|---|---|---|
| `car` | Ô tô con 4–9 chỗ, sedan, SUV, hatchback, station wagon. | Ô tô con đỗ bên đường (`img_001.jpg`), ô tô con chở trên sàn xe tải (`img_002.jpg`). | Tượng xe bán tải bằng đá (`img_027.jpg`), ô tô đồ chơi trong tủ kính (`img_038.jpg`). |
| `truck` | Xe tải chở hàng có thùng riêng, xe đầu kéo, xe bán tải thùng hở. | Xe tải chở xe (`img_002.jpg`), xe tải nhỏ (`img_036.jpg`), xe bán tải thùng hở (`img_001.jpg`, `img_012.jpg`). | Tượng xe tải (`img_027.jpg`), xe thùng kín chở khách (`van`). |
| `van` | Xe thùng kín cỡ vừa/nhỏ có thân liền khối (không tách cabin và thùng). | Xe van chuyển phát nhanh, xe bán tải có nắp thùng cao kín Master 75 (`img_037.jpg`). | Xe tải có cabin và thùng hàng tách rời (`truck`). |
| `motorcycle` | Xe máy, xe tay ga, mô tô 2–3 bánh có động cơ. | Xe máy đang chạy rõ nét (`img_005.jpg`, `img_026.jpg`), mô tô có thùng phụ (`img_022.jpg`). | Xe đạp thường (`img_003.jpg`), xích lô đạp chân (`img_006.jpg`). |
| `person` | Con người thật đang đi bộ, đứng hoặc đang lái / ngồi trên xe. | Người đi bộ qua đường (`img_014.jpg`), người ngồi lái xe máy (`img_026.jpg`), người bán hàng rong (`img_003.jpg`). | Tượng đồng tướng cưỡi ngựa (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), hình người in trên quảng cáo (`img_018.jpg`). |
| `ignore_region` | Vùng bỏ qua: khoanh trùm đám đông chen chúc hoặc cụm xe nhỏ ở hậu cảnh xa để không bị tính lỗi sót. | Đám đông xe máy kẹt xe ở xa (`img_030.jpg`), đám đông chờ xe buýt (`img_032.jpg`), dòng xe xa tít (`img_040.jpg`). | Người hoặc xe đứng đơn lẻ nhìn rõ ở gần. |

**Xử lý đối tượng ngoài danh mục:**
- **Xe đạp, Xích lô** (`img_003.jpg`, `img_006.jpg`): Không có trong taxonomy $\rightarrow$ **KHÔNG GÁN PHƯƠNG TIỆN**. Người điều khiển nếu nhìn rõ thì gán `person`.
- **Xe tuk-tuk / xe 3 bánh** (`img_004.jpg`, `img_035.jpg`): Không có lớp riêng $\rightarrow$ **BỎ QUA KHÔNG GÁN hoặc gom `ignore_region`**. Không gán ép vào `car` hay `motorcycle`.

### Thuộc tính

| Thuộc tính | Các mức cố định | Ngưỡng định lượng dễ áp dụng | Mặc định |
|---|---|---|---|
| `occluded_level` | `0`, `1`, `2`, `3` | **`0`**: Nhìn rõ trọn vẹn (0% bị che).<br>**`1`**: Bị che nhẹ (**1% – 30%** diện tích).<br>**`2`**: Bị che vừa đến nặng (**31% – 70%** diện tích).<br>**`3`**: Bị che rất nặng (**> 70%** diện tích nhưng vẫn nhận diện chắc chắn 100%). | `"0"` |
| `truncated` | `no`, `yes` | **`no`**: Nằm trọn trong ảnh.<br>**`yes`**: Bị mép ảnh cắt cụt một phần thân. | `"no"` |
| `can_review` | `true`, `false` | **`false`**: Tự tin với box.<br>**`true`**: Phân vân, đánh dấu để nhóm họp chốt ở Pha 5. | `"false"` |

---

## 3. Quy tắc vẽ box — Tối giản & Dễ thực hiện

- **`R-01` (Ôm sát phần nhìn thấy)**: Box chỉ bao quanh phần nhìn thấy thực tế của vật thể. Không cần đoán phần vô hình bị khuất sau xe khác.
- **`R-02` (Tách Người và Xe máy ở tiền cảnh)**: Khi xe máy ở gần nhìn rõ: gán **1 box `motorcycle`** ôm thân xe (bánh chạm đất, tay lái) và **1 box `person` cho MỖI người** trên xe (từ đầu đến chân còn nhìn thấy). Xe chở 2 người (`img_026.jpg`) gán 2 `person` + 1 `motorcycle`.
- **`R-03` (BỎ QUA VẬT TĨNH KHÔNG CÓ SỰ SỐNG)**: Tuyệt đối **KHÔNG GÁN** tượng đồng/đá (`img_017.jpg`, `img_027.jpg`), ma-nơ-canh (`img_033.jpg`), hình in quảng cáo (`img_018.jpg`), đồ chơi tủ kính (`img_038.jpg`).
- **`R-04` (BỎ QUA HÌNH PHẢN CHIẾU)**: Tuyệt đối **KHÔNG GÁN** hình ảnh phản chiếu qua gương chiếu hậu (`img_020.jpg`) hoặc bóng phản chiếu trên mặt đường ướt mưa (`img_013.jpg`).
- **`R-05` (BỎ QUA VẬT Ở XA / NHỎ < 20 px / MỜ NÉT)**: Bất kỳ người hoặc xe nào ở xa có **chiều cao box nhỏ hơn 20 pixels** ($h < 20\text{ px}$) hoặc quá mờ nhòe không rõ đường nét: **BỎ QUA KHÔNG CẦN GÁN** (`img_025.jpg`, `img_040.jpg`).
- **`R-06` (KHÔNG KHẮT KHE VỚI ĐÁM ĐÔNG / KẸT XE — DÙNG `ignore_region`)**: Tại các điểm kẹt xe đông đúc hoặc trạm dừng tập trung đông người (`img_005.jpg`, `img_030.jpg`, `img_032.jpg`, `img_040.jpg`), **chỉ gán 3–5 đối tượng rõ ràng nhất ở ngay phía trước**. Toàn bộ cụm người/xe dày đặc dính vào nhau ở phía sau vẽ **1–2 box lớn gán `ignore_region`**, không ngồi soi gán từng xe nhỏ lẻ.
- **`R-07` (BỎ QUA HÀNG HÓA CỒNG KỀNH)**: Xe máy chở thùng xốp to (`img_029.jpg`), xe bán tải chở hàng cao (`img_012.jpg`): **Box xe CHỈ ÔM KHUNG THÂN VỎ CƠ BẢN CỦA XE**, không kéo dãn ôm hàng hóa cồng kềnh chìa ra ngoài.
- **`R-08` (XE CHỞ XE `img_002.jpg`)**: Gán **1 box `truck`** cho xe tải chở hàng; gán từng box `car` cho các ô tô con trên sàn nhìn rõ $\ge 30\%$. Xe bị khuất sâu tầng dưới $> 70\%$ thì bỏ qua.
- **`R-09` (ĐỐI TƯỢNG NGOÀI DANH MỤC)**: Xe đạp, xích lô không gán xe; chỉ gán người `person` nếu nhìn rõ.
- **`R-10` (BỎ QUA CHE KHUẤT QUÁ NẶNG)**: Vật thể bị che $> 70\%$ diện tích mà chỉ lộ một mẩu nhỏ rời rạc: **BỎ QUA KHÔNG GÁN**.

---

## 4. Ca khó

Bảng 6 ca tiêu biểu minh họa quy tắc đơn giản, dễ áp dụng:

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chuyên dụng chở 8 ô tô con trên hai tầng sàn | `R-08` | Gán **1 box `truck`** cho toàn bộ xe tải; gán từng box **`car`** (`occluded=1` hoặc `2`) cho các xe con trên sàn nhìn rõ $\ge 30\%$. | Ô tô con trên sàn là vật thể thật kích thước lớn, cần nhận diện. |
| `img_017.jpg` | Tượng đồng tướng Duke of Wellington cưỡi ngựa tại Glasgow | `R-03` | **TUYỆT ĐỐI KHÔNG GÁN BOX** (0 annotation cho cả người và ngựa). | Tượng kim loại tĩnh không tham gia giao thông; gán sẽ là lỗi thừa (`thua`) gây phanh ma. |
| `img_020.jpg` | Xe cộ và cảnh quan phản chiếu qua gương chiếu hậu ô tô | `R-04` | **TUYỆT ĐỐI KHÔNG GÁN BOX** các vật thể trong gương. | Hình ảnh phản chiếu ảo, không có chướng ngại vật vật lý tại mặt gương. |
| `img_026.jpg` | Cặp đôi đèo nhau trên xe máy trong hầm chui | `R-02` | Gán **2 box `person`** riêng biệt cho 2 người; gán **1 box `motorcycle`** ôm toàn bộ thân xe. | Chuẩn hóa quy tắc tách người và xe máy rõ nét ở cự ly gần. |
| `img_029.jpg` | Xe máy chở bao tải xốp cồng kềnh gấp 3 lần xe | `R-02`, `R-07` | Gán 1 `motorcycle` **chỉ ôm khung/bánh xe**; gán 1 `person` cho người lái. **KHÔNG BAO HÀNG HÓA**. | Tránh làm phình to box xe gây lệch kích thước và tụt IoU giữa các thành viên. |
| `img_030.jpg` | Ngã tư đường phố đặc kín hàng trăm xe máy chen chúc | `R-02`, `R-06` | Gán 4–5 xe máy và người nhìn rõ nhất ở tiền cảnh. Toàn bộ dòng xe chen chúc phía sau: khoanh 1–2 box **`ignore_region`**. | Tiết kiệm công sức, không khắt khe với nơi kẹt xe đông đúc, đúng chuẩn kỹ thuật. |

---

## 5. QC và phiên bản

### 5.1. Ngưỡng QC áp dụng
- **Độ phủ phân công**: Đủ $100\%$ ($80/80$ lượt gán cho 40 ảnh với $K=2$).
- **Kích thước tối thiểu**: $100\%$ box có chiều cao $\ge 20\text{ px}$ (trừ `ignore_region`).
- **Độ đồng thuận IAA (IoU ghép $\ge 0.5$)**:
  - IoU trung bình: **$\ge 0.85$**.
  - Thống nhất phân lớp (Class Agreement): **$\ge 95\%$**.
- **Tỉ lệ lỗi QC toàn lô**: Tỉ lệ ảnh có lỗi trên 40 ảnh đã duyệt **$\le 2.0\%$** (chỉ cần $\ge 1$ ảnh lỗi là vượt ngưỡng).

### 5.2. Ai quyết định ca mơ hồ?
- Khi gán độc lập: nếu phân vân, tick cờ `can_review = true` trên box đó.
- Trong buổi họp calibration: Người điều phối (101 - Nguyễn Viết Tiến) chủ trì thảo luận, chốt quyết định theo biểu quyết đa số (2/3 phiếu) và đối chiếu với Người review (102 - Phạm Thanh Tùng).

### 5.3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| **v0** | Bản khởi đầu của giảng viên (5 lớp định tính, không có ngưỡng đo). | Quá sơ sài; hai người đọc hiểu ra hai cách khác nhau. |
| **v0.1** | - Định lượng hóa `occluded_level` ($0\%, 1-30\%, 31-70\%, >70\%$).<br>- Đưa ra 10 mã quy tắc `R-01` đến `R-10`.<br>- Tách người đi xe máy thành 2 box riêng biệt ở cự ly gần (`R-02`).<br>- Loại bỏ 100% tượng, ma-nơ-canh, biển quảng cáo, gương phản chiếu (`R-03`, `R-04`).<br>- Box xe chỉ ôm thân vỏ, không bao hàng cồng kềnh (`R-07`).<br>- **Tinh giản thực dụng**: Không khắt khe với vật ở xa/mờ nhòe; cho phép gom `ignore_region` cho kẹt xe, đám đông (`R-05`, `R-06`). | - Tránh bất đồng kích thước box xe chở hàng (`img_029.jpg`).<br>- Chống lỗi phanh ma trong bài toán ADAS (`img_017.jpg`, `img_020.jpg`).<br>- Giảm nhẹ khối lượng gán nhãn, giúp nhóm 3 người hoàn thành nhanh và đồng thuận cao. |
