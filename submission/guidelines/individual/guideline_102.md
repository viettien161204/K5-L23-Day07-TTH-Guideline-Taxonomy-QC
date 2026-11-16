# Guideline gán nhãn — v1.0-pre (Bản cá nhân) — Thành viên 102

> Bản guideline cá nhân phục vụ Pha 1 của lab Day 07 — Guideline, Taxonomy & QC.
> Văn bản này giải quyết các điểm mơ hồ, sơ sài của bản v0 nhằm đảm bảo hai người gán độc lập đạt độ đồng thuận cao nhất.

- Phiên bản: v1.0-pre-102
- Ngày: 2026-11-16
- Người viết: Thành viên 102
- Loại nhãn: Bounding box 2D (hộp chữ nhật bao quanh vật thể)
- Định dạng xuất: COCO 1.0

---

## 1. Mục tiêu

Tập dữ liệu nhãn được xây dựng để huấn luyện và đánh giá mô hình học sâu phát hiện vật thể 2D (2D Object Detection) phục vụ bài toán **giao thông thông minh và hỗ trợ lái xe tự động (ADAS/Autonomous Driving)** trên ảnh đường phố thực tế.

- **Mô hình cần đoán được:** Vị trí hộp bao 2D ôm sát (bounding box), phân loại lớp chính xác (phương tiện cơ giới đang lưu thông hoặc dừng/đỗ, người đi bộ, người tham gia giao thông), cùng các thuộc tính trạng thái che khuất (`occluded_level`) và cắt mép (`truncated`).
- **Mô hình không cần đoán:** Các vật thể không phải mục tiêu giao thông thực tế như tranh vẽ 2D trên biển quảng cáo/tường, tượng điêu khắc nghệ thuật, hình nộm/ma-nơ-canh trong tủ kính, đồ chơi mô hình thu nhỏ, hình ảnh ảo phản chiếu qua gương chiếu hậu/kính cửa sổ, hàng hóa/đòn gánh mang vác rời, và phương tiện thô sơ không động cơ (xe đạp, xích lô đạp).
- **Hậu quả nếu nhãn sai:**
  - *Âm tính giả (False Negative - Bỏ sót vật thể):* Cực kỳ nguy hiểm. Nếu mô hình tự lái bỏ sót người đi bộ qua đường (ví dụ `img_034.jpg`) hoặc người đi xe máy khuất sau xe tải (`img_005.jpg`), xe sẽ không phanh kịp và gây tai nạn trực tiếp.
  - *Dương tính giả (False Positive - Nhận nhầm vật không có thật):* Gây hiện tượng "phanh ma" (phantom braking) hoặc đánh lái khẩn cấp sai lầm nếu mô hình nhầm hình phản chiếu trong gương (`img_020.jpg`), tranh quảng cáo chân dung (`img_018.jpg`), hay tượng điêu khắc đá (`img_027.jpg`) là vật cản thật trên đường.
  - *Sai lệch ranh giới box (Bounding Box Jitter/Drift):* Box vẽ quá rộng bao cả bóng đổ hoặc hàng hóa cồng kềnh sẽ làm sai lệch kích thước vật thể, gây tính toán sai khoảng cách an toàn (Time-to-Collision).

---

## 2. Lớp và thuộc tính

### 2.1. Bảng định nghĩa lớp

*Áp dụng phép thử lớp vs thuộc tính:* "Mô hình phát hiện 2D có phải đoán ra thông tin này khi suy luận thực tế không?"
- Các loại phương tiện có kích thước, hình thái và đặc tính động học khác biệt hoàn toàn (như xe buýt lớn cần bán kính quay lớn, xe tuk-tuk 3 bánh luồn lách đặc thù ở Đông Nam Á) cần được xem xét là lớp riêng biệt thay vì gộp chung gây nhiễu phân phối đặc trưng của mô hình.

| Lớp | Định nghĩa (một câu) | Ví dụ (kèm tên ảnh thật) | Phản ví dụ / Dễ nhầm với (kèm tên ảnh thật) |
|---|---|---|---|
| `car` | Xe ô tô con từ 4 đến 9 chỗ ngồi (sedan, hatchback, SUV, crossover, wagon) dùng chở người cá nhân/gia đình/taxi. | Ô tô con trên xe chở xe (`img_002.jpg`), xe SUV Mazda (`img_012.jpg`), xe sedan Honda Civic (`img_023.jpg`), xe SUV Toyota RAV4 (`img_031.jpg`), xe taxi Bangkok (`img_040.jpg`). | Xe bán tải có cabin và thùng hàng tách rời (`img_001.jpg`, `img_012.jpg` — phân vào `truck`); xe mô hình đồ chơi thu nhỏ trong tủ kính (`img_038.jpg` — không gán); tượng đá hình xe (`img_027.jpg` — không gán). |
| `truck` | Xe tải cơ giới dùng để chở hàng hóa, bao gồm xe đầu kéo, xe tải thùng kín/hở, xe tải nhẹ (kei truck), và xe bán tải (pickup truck). | Xe chở ô tô chuyên dụng Renault (`img_002.jpg`), xe tải thùng kín Isuzu ELF (`img_007.jpg`), xe bán tải Suzuki Carry (`img_001.jpg`), xe tải nhẹ thùng bạt CMC Veryca (`img_036.jpg`), bán tải thùng cao Master 75 (`img_037.jpg`). | Xe ô tô con chở đồ trong cốp sau (`img_024.jpg` — là `car`); xe van thân liền chở khách (`img_005.jpg`, `img_016.jpg` — là `van`). |
| `van` | Xe ô tô chở người hoặc chở hàng có kết cấu thân xe liền khối một khoang (monocoque), nóc phẳng cao, không tách biệt rời cabin và thùng hàng. | Xe van chở khách MB 100 màu bạc (`img_005.jpg`), xe van Toyota HiAce màu bạc (`img_016.jpg`). | Xe tải nhẹ có cabin và sàn/thùng chở hàng tách biệt như CMC Veryca (`img_036.jpg` — là `truck`); xe bán tải pickup (`img_001.jpg`, `img_012.jpg` — là `truck`). |
| `motorcycle` | Phương tiện cơ giới hai hoặc ba bánh chạy bằng động cơ (xăng hoặc điện), bao gồm xe máy số, xe tay ga, xe phân khối lớn, xe máy điện và xe mô tô có thùng thuyền bên hông (sidecar). | Xe tay ga và xe số trên đường phố (`img_005.jpg`, `img_006.jpg`, `img_008.jpg`), xe Vespa cổ (`img_016.jpg`), xe mô tô có sidecar (`img_022.jpg`), xe máy điện Wanboru (`img_029.jpg`). | Xe đạp không động cơ (`img_006.jpg`, `img_029.jpg` — không gán phương tiện); xe tuk-tuk 3 bánh chuyên dụng có cabin chở khách (`img_004.jpg`, `img_016.jpg` — đề xuất `tuk_tuk`); mô hình xe máy điêu khắc kim loại (`img_011.jpg` — không gán). |
| `person` | Con người còn sống xuất hiện trong ảnh (người đi bộ, người lái xe, người ngồi sau xe, người đứng/ngồi trên vỉa hè hoặc lòng đường). | Người phụ nữ xăm mình và thực khách (`img_001.jpg`), người đi bộ qua đường (`img_014.jpg`, `img_034.jpg`), người điều khiển xe máy (`img_005.jpg`, `img_026.jpg`), hàng người xếp hàng đợi xe buýt (`img_032.jpg`). | Tượng đồng cưỡi ngựa (`img_017.jpg` — không gán); ma-nơ-canh nhựa trong tủ kính (`img_033.jpg` — không gán); ảnh chân dung 2D trên biển quảng cáo (`img_018.jpg` — không gán); tượng quái vật viễn tưởng Predator (`img_011.jpg` — không gán); búp bê/thú bông (`img_010.jpg`, `img_038.jpg` — không gán). |
| `bus` *(Đề xuất mới)* | Phương tiện giao thông công cộng chở khách cỡ lớn (từ 16–29 chỗ trở lên) chạy theo tuyến hoặc hợp đồng, có thân xe dài và trần cao vượt trội. | Xe khách 29 chỗ Hyundai County (`img_009.jpg`), xe buýt đô thị tuyến 62 (`img_031.jpg`), xe buýt cỡ lớn GO Transit (`img_037.jpg`), xe buýt Bangkok tuyến 511 (`img_040.jpg`). | Xe van chở khách 9–12 chỗ Toyota HiAce (`img_016.jpg` — là `van`); xe tải thùng dài chở hàng (`img_002.jpg` — là `truck`). |
| `tuk_tuk` *(Đề xuất mới)* | Xe cơ giới ba bánh có cabin hở hoặc khung che mái vòm, chuyên dùng chở khách hoặc hàng hóa tại khu vực đô thị (tuk-tuk, auto rickshaw, xe lam). | Xe tuk-tuk Bangkok màu xanh (`img_004.jpg`, `img_016.jpg`), xe auto rickshaw Hyderabad chở hàng cao (`img_035.jpg`), xe tuk-tuk vàng-xanh Bangkok (`img_040.jpg`). | Xe mô tô 2 bánh thông thường (`img_005.jpg` — là `motorcycle`); xe xích lô đạp chân không động cơ (`img_023.jpg` — không gán phương tiện); xe ô tô con mini (`img_015.jpg` — là `car`). |
| `ignore_region` | Vùng bao quanh các khu vực tập trung quá nhiều đối tượng nhỏ li ti, mật độ dày đặc hoặc sâu trong hậu cảnh không thể phân tách ranh giới chính xác từng vật. | Vùng biển xe máy ken đặc phía xa tại ngã tư đèn đỏ (`img_021.jpg`), khu vực các hàng xe máy đỗ sâu bên trong bãi gửi xe bị che khuất gần như toàn bộ (`img_030.jpg`). | Từng phương tiện ở cự ly gần và trung bình nhìn rõ biên dạng (bắt buộc gán box riêng lẻ với thuộc tính phù hợp). |

*Quy tắc xử lý vật ngoài taxonomy:*
- Phương tiện thô sơ không động cơ (xe đạp, xe xích lô đạp, xe đẩy tay hàng rong): KHÔNG gán box phương tiện (để tránh làm loãng dữ liệu huấn luyện xe cơ giới). Người điều khiển/đạp/dắt phương tiện thô sơ vẫn được gán box `person`.
- Nếu nhóm chưa thông qua việc thêm 2 lớp `bus` và `tuk_tuk` trong Phase 2: Thành viên 102 đề xuất tạm thời quy ước `bus` gán vào `truck` hoặc `van` kèm `can_review=true`; `tuk_tuk` gán vào `motorcycle` kèm `can_review=true`.

---

### 2.2. Thuộc tính với ngưỡng đo lường định lượng

Không sử dụng các tính từ định tính chung chung ("che ít", "che nhiều", "không rõ"). Áp dụng tỷ lệ diện tích bị che khuất hình học cụ thể như sau:

#### A. Thuộc tính `occluded_level` (Mức độ che khuất)
- **`0` — Không che khuất:** Nhìn thấy trọn vẹn 100% diện tích bề mặt khả kiến của vật thể (hoặc chỉ bị che viền mép không đáng kể < 1%). Ví dụ: người đi bộ đứng giữa đường vắng trong `img_034.jpg`.
- **`1` — Che khuất nhẹ (1% đến 30%):** Bị che lấp một phần nhỏ bởi vật thể khác, cành cây, rào chắn hoặc phụ kiện. Cấu trúc và hình dạng nhận dạng chính của vật thể vẫn hiển thị rõ ràng trên 70%. Ví dụ: xe ô tô sau hàng rào thưa trong `img_018.jpg`, người đứng sau bàn ăn trong `img_001.jpg`.
- **`2` — Che khuất trung bình / nặng (31% đến 70%):** Bị che khuất từ một phần ba đến hơn một nửa diện tích vật thể bởi các phương tiện khác hoặc công trình che chắn, nhưng các đặc trưng cốt lõi (đèn, bánh xe, mặt, thân trên) vẫn nhận diện chắc chắn không nhầm lẫn. Ví dụ: người lái xe máy bị thân xe và ghi đông che chân trong `img_005.jpg`, xe máy đỗ san sát trong `img_008.jpg`.
- **`3` — Che khuất cực nặng (trên 70% đến 90%):** Chỉ còn nhìn thấy từ 10% đến dưới 30% diện tích vật thể (ví dụ chỉ thấy đầu và vai người, hoặc chỉ thấy góc mui xe), nhưng ngữ cảnh xung quanh khẳng định chắc chắn 100% sự hiện diện của đối tượng. Ví dụ: người ngồi sâu trong hàng xe máy `img_021.jpg`.
- *Ngưỡng loại bỏ:* Nếu vật thể bị che khuất trên 90% diện tích và không thể nhận dạng chắc chắn loại đối tượng bằng mắt thường -> **KHÔNG GÁN** (tránh sinh nhãn nhiễu).

#### B. Thuộc tính `truncated` (Cắt mép ảnh)
- **`no`:** Vật thể nằm hoàn toàn trong khung hình ảnh, cả 4 cạnh của hộp bao không tiếp xúc hoặc không bị cắt đứt bởi cạnh viền của ảnh.
- **`yes`:** Vật thể có ít nhất một phần giải phẫu hoặc chi tiết thân vỏ bị cắt đứt bởi bất kỳ mép viền nào của bức ảnh (trên, dưới, trái, phải). Ví dụ: đầu xe ô tô đỏ ở mép trái và đuôi xe bạc ở mép phải trong `img_015.jpg`, phần thân dưới người lái xe máy ở góc dưới ảnh `img_005.jpg` và `img_040.jpg`.
- *Ngưỡng loại bỏ:* Nếu phần vật thể lọt vào mép ảnh quá nhỏ (< 10% diện tích cơ thể/xe) và không đủ cơ sở xác định đúng lớp -> KHÔNG gán.

#### C. Thuộc tính `can_review` (Đánh dấu cần xem lại)
- Giá trị boolean (`false` / `true`):
  - **`false`:** Ca gán rõ ràng, tuân thủ đúng định nghĩa và quy tắc đã xác lập.
  - **`true` (Tick):** Bắt buộc bật khi người gán nhãn gặp tình huống phân vân, độ tin cậy thấp, tranh chấp ranh giới box hoặc lớp giữa 2 đối tượng, hình ảnh bị nhòe mờ chuyển động nặng (motion blur như `img_014.jpg`), hoặc các ca nằm ở ranh giới taxonomy cần cả nhóm hoặc reviewer biểu quyết trong cuộc họp calibration.

---

## 3. Quy tắc vẽ box (Bounding Box Rules)

Mỗi quy tắc được định danh bằng mã `R-xx` để thuận tiện truy xuất và đối chiếu trong nhật ký QC.

### R-01: Quy tắc ôm sát phần nhìn thấy (Tight Visual Boundary)
- Hộp bao 2D phải vẽ sát theo đường biên ngoài cùng của **phần vật thể thực sự nhìn thấy** (visual extent).
- **Tuyệt đối KHÔNG** vẽ bao gồm bóng đổ của vật thể trên mặt đường (shadow) hoặc hình ảnh phản chiếu trên mặt đường ướt (reflections). Ví dụ: người đi bộ đổ bóng dài trên mặt đường trong `img_034.jpg`, vũng nước phản chiếu đèn trong `img_013.jpg` phải bị loại khỏi box.
- Không suy đoán "vẽ bù" phần vật thể bị che khuất khuất tầm mắt ra ngoài không gian ảnh trống.

### R-02: Quy tắc tách biệt người và phương tiện đi kèm (Rider & Vehicle Separation)
- **Với xe máy (motorcycle):** Luôn gán các box độc lập cho người và phương tiện.
  - Mỗi người ngồi trên xe máy (người lái và người ngồi sau/pillion) được gán **1 box `person` riêng biệt**, ôm sát phần cơ thể nhìn thấy của người đó (đầu, thân, chân tay). Ví dụ: `img_006.jpg`, `img_026.jpg` (1 người lái + 1 người ngồi sau = 2 box `person`).
  - Chiếc xe máy được gán **1 box `motorcycle` riêng**, ôm sát toàn bộ thân xe (bánh xe, yếm, tay lái, ống xả), kể cả phần thân xe bị chân và người lái che đè lên (đặt `occluded_level=2`).
  - *Tự kiểm tra:* Trên ảnh `img_026.jpg` (cặp đôi đi xe tay ga đỏ), quy tắc này cho ra chính xác 3 bounding box: 1 box person (nam lái), 1 box person (nữ ngồi sau), 1 box motorcycle (xe đỏ). Áp dụng lại trên `img_005.jpg` và `img_010.jpg` hoàn toàn nhất quán.
- **Với xe đạp và phương tiện thô sơ:**
  - Xe đạp/xích lô đạp không thuộc taxonomy phương tiện cơ giới -> **KHÔNG gán box phương tiện**.
  - Người đang đạp xe hoặc dắt xe đạp vẫn được gán **1 box `person`** (chỉ ôm cơ thể người, không trùm khung xe đạp). Ví dụ: `img_006.jpg`, `img_023.jpg`, `img_029.jpg`.

### R-03: Quy tắc xe chở xe (Piggyback / Car Carrier)
- Khi một phương tiện chuyên chở đang chở một hoặc nhiều phương tiện khác trên thùng xe hoặc rơ-moóc hai tầng:
  - Gán **1 box `truck`** bao quanh toàn bộ chiếc xe chuyên chở (gồm cabin đầu kéo và toàn bộ sàn rơ-moóc chở hàng).
  - Đồng thời gán **từng box `car` (hoặc phương tiện tương ứng)** riêng rẽ cho mỗi chiếc xe được chở trên các sàn xe nếu nhìn thấy rõ ràng hình dạng xe (đặt thuộc tính `occluded_level=1` hoặc `2` do khung kim loại của xe tải che).
  - *Tự kiểm tra:* Trên `img_002.jpg`, gán 1 box truck lớn cho xe Renault, cộng thêm các box car riêng biệt cho từng chiếc xe con trên tầng trên và tầng dưới.

### R-04: Quy tắc người bên trong phương tiện (Vehicle Occupants)
- **Phương tiện kín (ô tô con, xe tải kín cabin, xe buýt có kính):** KHÔNG gán box `person` cho tài xế và hành khách ngồi bên trong xe nhìn qua kính chắn gió hay cửa sổ (vì người ngồi trong xe kín là một thể thống nhất với phương tiện, không thể va chạm tách rời với xe trong bài toán giao thông ADAS). Ví dụ: `img_009.jpg` (không gán hành khách trong xe khách), `img_023.jpg` (không gán tài xế trong xe Honda Civic).
- **Phương tiện hở cabin (tuk-tuk, xe ba gác, mô tô sidecar):** Nếu tài xế hoặc hành khách lộ rõ cơ thể ra ngoài không gian mở (không có cửa và kính bao bọc), gán box `person` bình thường cho người đó. Ví dụ: `img_004.jpg`, `img_016.jpg`, `img_022.jpg`.

### R-05: Quy tắc ngưỡng kích thước tối thiểu (Minimum Size Threshold)
- Đối tượng đơn lẻ có cạnh lớn nhất < 20 pixels hoặc diện tích < 400 px² (nhỏ hơn 20x20 px) hoặc quá mờ nhạt ở hậu cảnh sâu: **KHÔNG gán nhãn**.
- Ngoại lệ: Nếu có đám đông xe cộ/người li ti tập trung thành dải lớn ở hậu cảnh ngã tư (như `img_021.jpg`) hoặc bãi đỗ xe sâu bên trong (`img_030.jpg`), sử dụng 1 vùng `ignore_region` bao phủ toàn bộ khu vực đó.

### R-06: Quy tắc loại trừ vật thể nhân tạo / không có thật (Non-real / Art / Replica Exclusion)
- **Tượng điêu khắc:** Tượng người, tượng quái vật, tượng động vật bằng đá, đồng, kim loại, thạch cao -> **TUYỆT ĐỐI KHÔNG GÁN** `person`. Ví dụ: tượng Predator (`img_011.jpg`), tượng Công tước Wellington (`img_017.jpg`), tượng gà/sư tử (`img_013.jpg`).
- **Tượng xe / bia mộ hình xe:** Tượng đài xe bằng đá hoặc mô hình tĩnh không có động cơ -> **KHÔNG GÁN** phương tiện. Ví dụ: bia mộ xe bán tải bằng đá granite (`img_027.jpg`).
- **Ma-nơ-canh (Mannequin):** Ma-nơ-canh bằng nhựa/sáp trong tủ kính hoặc cửa hàng -> **KHÔNG GÁN** `person`. Ví dụ: ma-nơ-canh nữ trong tủ kính (`img_033.jpg`), ma-nơ-canh treo đồ (`img_010.jpg`).
- **Hình in 2D / Tranh ảnh / Biển quảng cáo:** Ảnh chân dung người in trên áp phích, tranh vẽ tường, biển quảng cáo tấm lớn -> **KHÔNG GÁN** `person` hay phương tiện. Ví dụ: biển quảng cáo chân dung chính khách (`img_018.jpg`), tranh tường cảng biển (`img_026.jpg`), áp phích ca sĩ (`img_030.jpg`).
- **Đồ chơi / Mô hình thu nhỏ:** Xe đồ chơi trẻ em, mô hình ô tô tĩnh tỉ lệ nhỏ -> **KHÔNG GÁN** `car`. Ví dụ: xe đồ chơi màu vàng gắn trên nóc (`img_024.jpg`), tủ kính trưng bày hàng chục mô hình xe hơi đồ chơi (`img_038.jpg`).

### R-07: Quy tắc phản chiếu qua gương / kính (Reflections)
- Hình ảnh ảo xuất hiện do phản xạ qua gương chiếu hậu ô tô, gương xe máy, kính tủ trưng bày hoặc mặt đường ướt -> **TUYỆT ĐỐI KHÔNG GÁN BOX**.
- Việc gán nhãn cho ảnh ảo trong gương sẽ khiến thuật toán tự lái lầm tưởng có vật cản ngay trước đầu xe. Ví dụ: xe ô tô phản chiếu trong gương xe (`img_020.jpg`), bóng xe phản chiếu trên kính tủ (`img_033.jpg`), bóng người phản chiếu trên kính mũ bảo hiểm (`img_040.jpg`).

### R-08: Quy tắc hàng hóa và vật mang vác (Cargo & Luggage Handling)
- **Hàng hóa trên thùng xe / nóc xe:**
  - Hàng hóa nằm gọn trong thùng xe bán tải/xe tải hoặc buộc cố định trên nóc phương tiện: Hộp bao của phương tiện **bao trọn cả phần hàng hóa gắn liền** nếu hàng hóa chuyển động đồng nhất cùng xe và ảnh hưởng trực tiếp đến kích thước chiếm dụng làn đường của xe. Ví dụ: xe tuk-tuk chở bao tải chất cao trên nóc (`img_035.jpg`), xe bán tải chở hàng rau củ thùng sau (`img_012.jpg`).
  - *Ngoại lệ hàng quá khổ / trưng bày nghệ thuật:* Nếu thùng xe chở vật trưng bày độc lập nhô cao bất thường (như tượng ngựa trắng "yorse" trên thùng xe bán tải `img_028.jpg`), hộp bao `truck` chỉ bao quanh thân xe đến thành thùng và đỉnh cabin; KHÔNG kéo box cao vút bao cả tượng ngựa vì sẽ làm biến dạng nghiêm trọng tỷ lệ khung hình chuẩn của xe tải.
- **Hàng hóa do người mang vác rời:** Đòn gánh thúng hàng (`img_003.jpg`) hoặc thanh nhôm dài vác trên vai (`img_029.jpg`) **KHÔNG** được tính vào box `person`. Box `person` chỉ ôm sát giải phẫu cơ thể người.

### R-09: Quy tắc xử lý ảnh không có đối tượng hợp lệ (Empty Images)
- Bất kỳ bức ảnh nào sau khi áp dụng các quy tắc loại trừ (R-06, R-07, R-08) mà không còn đối tượng giao thông thực tế hợp lệ nào: **Giữ nguyên ảnh trong tập dữ liệu với 0 annotation** (danh sách annotations rỗng).
- Tuyệt đối không vẽ gượng ép box vào đồ chơi hay tượng đá để "cho có nhãn". Các ảnh âm tính (negative samples) này có giá trị cực cao để huấn luyện mô hình không bị báo động giả. Ví dụ: `img_017.jpg`, `img_020.jpg`, `img_027.jpg`, `img_038.jpg`.

---

## 4. Ca khó và tình huống biên (Edge Cases Table)

Bảng tổng hợp ít nhất 10 ca khó tiêu biểu từ tập ảnh 40 ảnh đã khảo sát, kèm quyết định và quy tắc dẫn chiếu:

| Ảnh | Tình huống quan sát trên ảnh | Rule ID | Quyết định cụ thể của 102 | Lý do giải thích | Phân loại |
|---|---|---|---|---|---|
| `img_002.jpg` | Xe tải Renault chở 8 ô tô con trên hai tầng sàn xe; có người đứng cạnh bánh sau. | R-03, R-01 | Gán 1 box `truck` cho toàn bộ xe tải Renault. Gán riêng từng box `car` cho mỗi ô tô con trên sàn xe (đặt `occluded_level=1` hoặc `2`). Gán 1 box `person` cho người đứng sau. | Xe con trên sàn là vật thể ô tô thực tế; phân tách rõ ràng giúp mô hình học được cả phương tiện chở và vật được chở. | Ca áp dụng quy tắc |
| `img_004.jpg` | Hai xe tuk-tuk màu xanh đỗ trên vỉa hè ban đêm, có tài xế ngồi bên trong cabin hở. | R-04, R-01 | Gán 2 box `tuk_tuk` cho 2 xe (hoặc `motorcycle` kèm `can_review=true` nếu nhóm chưa duyệt lớp mới). Gán box `person` cho tài xế ngồi trong cabin hở. | Xe ba bánh có động cơ là phương tiện cơ giới riêng biệt. Cabin hở nên người lái lộ rõ hình thể, cần gán box person. | Ca cần hỏi reviewer / họp nhóm |
| `img_011.jpg` | Tượng quái vật viễn tưởng Predator khổng lồ đứng cạnh xe máy điêu khắc kim loại; người thật phía xa. | R-06, R-05 | KHÔNG gán box `person` cho tượng Predator. KHÔNG gán xe máy điêu khắc. Chỉ gán các cá nhân người thật phía xa nếu kích thước > 20 px. | Tượng Predator và xe máy kim loại là tác phẩm điêu khắc nghệ thuật trưng bày, không phải người hay phương tiện giao thông. | Ca loại theo quy tắc |
| `img_014.jpg` | Hai người đi bộ rõ nét qua đường; nhiều người và xe máy xung quanh bị nhòe chuyển động (motion blur) rất nặng. | R-01, R-05 | Gán 2 box `person` nét rõ cho 2 người đi bộ (`occluded_level=0`). Với các vệt nhòe xe máy lướt qua: chỉ gán box khi còn nhận ra hình khối chính kèm `can_review=true`; các vệt mờ mất hẳn hình dạng thì bỏ qua. | Vệt nhòe chuyển động do chụp tốc độ chậm làm mất biên giới vật thể, dễ gây bất đồng IoU giữa các annotator. | Ca cần hỏi reviewer / họp nhóm |
| `img_017.jpg` | Tượng đài kim loại Công tước Wellington cưỡi ngựa đội nón chóp giao thông tại Glasgow. | R-06, R-09 | KHÔNG gán bất kỳ box nào trên toàn bộ bức ảnh (0 annotation). | Bức ảnh chỉ chứa tượng đài điêu khắc bằng kim loại và chim hải âu. Tượng không phải người thật, ngựa không phải phương tiện. | Ca loại theo quy tắc |
| `img_018.jpg` | Biển quảng cáo ngoài trời in ảnh chân dung chính khách Lukáš ZICHA; phía dưới có ô tô đỗ sau hàng rào sắt. | R-06, R-01 | KHÔNG gán box `person` cho ảnh in chân dung trên biển. Gán box `car` cho các ô tô sau hàng rào với `occluded_level=1` hoặc `2`. | Ảnh in 2D trên biển bảng không phải người thật. Ô tô sau hàng rào là xe thật bị rào che một phần diện tích. | Ca loại theo quy tắc |
| `img_019.jpg` | Xe diễu hành Danger Mouse màu vàng (biển số YR04 NUM) chở người lái thật và đầu mascot chuột bông; đám đông xem hội. | R-01, R-06 | Gán 1 box `car` cho xe Danger Mouse. Gán 1 box `person` cho người lái thật. KHÔNG gán box cho đầu mascot chuột nhồi bông. Gán box `person` cho các khán giả thấy rõ trong đám đông. | Chiếc xe diễu hành là xe cơ giới thật, có biển số, chở người lưu thông trên đường. Đầu mascot chuột bông là phụ kiện hóa trang, không phải người. | Ca áp dụng quy tắc |
| `img_020.jpg` | Gương chiếu hậu ô tô đọng nước phản chiếu hình ảnh mặt đường, cầu vồng và một ô tô chạy phía sau. | R-07, R-09 | KHÔNG gán box `car` cho chiếc xe ô tô phản chiếu trong gương (0 annotation hợp lệ trên ảnh). | Vật thể phản chiếu trong gương là ảnh ảo quang học, không tồn tại vật thể vật lý trước máy ảnh. Gán nhãn sẽ làm sai lệch mô hình tự lái. | Ca loại theo quy tắc |
| `img_027.jpg` | Bia mộ / tượng đài bằng đá granite đen nguyên khối tạc hình xe bán tải trong nghĩa trang Oak Hill. | R-06, R-09 | KHÔNG gán box phương tiện (`truck`/`car`) cho khối tượng đá bán tải (0 annotation). | Khối đá granite đặt cố định trên bệ trong nghĩa trang, không có động cơ, không phải phương tiện giao thông. | Ca loại theo quy tắc |
| `img_028.jpg` | Xe bán tải cổ màu nâu chở một bức tượng ngựa trắng lớn ("yorse") đứng thẳng trên thùng xe; xe SUV đỏ phía sau. | R-08, R-06 | Gán 1 box `truck` cho xe bán tải nâu với ranh giới hộp bao ôm thân xe đến mép thùng và đỉnh cabin (KHÔNG bao gồm chiều cao tượng ngựa). KHÔNG gán box cho tượng ngựa. Gán box `car` cho xe SUV đỏ. | Tượng ngựa là vật thể trưng bày độc lập nhô cao bất thường; nếu bao trùm cả tượng ngựa sẽ làm biến dạng tỷ lệ khung hình xe bán tải, gây lỗi cho mô hình nhận diện xe. | Ca cần hỏi reviewer / họp nhóm |
| `img_029.jpg` | Hai người (một đi xe đạp, một đi xe máy điện Wanboru) cùng vác chung một thanh nhôm dài hàng mét trên vai. | R-02, R-08 | Gán 1 box `person` cho người đi xe đạp (không gán xe đạp). Gán 1 box `person` và 1 box `motorcycle` cho người và xe máy điện. Ranh giới box người/xe KHÔNG bao trùm thanh nhôm dài. | Thanh nhôm dài vác trên vai là hàng hóa rời; không được kéo dãn box người/xe hàng mét để trùm thanh nhôm. Xe đạp là phương tiện thô sơ không gán. | Ca áp dụng quy tắc |
| `img_035.jpg` | Xe auto rickshaw / tuk-tuk chở hàng cồng kềnh chất cao vút 2-3 tầng bao tải trên nóc; người đi xe máy đối diện. | R-08, R-02 | Gán 1 box `tuk_tuk` bao trọn cả thân xe và khối bao tải hàng hóa chằng buộc trên nóc xe. Người đi xe máy đối diện gán 1 box `person` và 1 box `motorcycle`. | Khối hàng hóa chằng buộc cố định trên nóc xe tuk-tuk chuyển động cùng xe và tạo thành một khối kích thước vật lý duy nhất khi xe lưu thông. | Ca áp dụng quy tắc |
| `img_038.jpg` | Tủ kính cửa hàng đồ chơi ở Munich bày hàng chục mô hình xe hơi thu nhỏ (Porsche, Ferrari...), búp bê, bóng người phản chiếu. | R-06, R-07, R-09 | KHÔNG gán bất kỳ box nào trên ảnh (0 annotation). | Toàn bộ các xe trong ảnh là mô hình đồ chơi thu nhỏ (scale models) trong tủ kính, búp bê không phải người, bóng phản chiếu không gán. Đây là ảnh đối chứng âm tính. | Ca loại theo quy tắc |

---

## 5. QC và quản lý phiên bản

### 5.1. Tiêu chí kiểm soát chất lượng (QC Criteria)
1. **Kiểm tra tự động bằng mã lệnh (Script-based Validation):**
   - Hợp lệ cấu trúc COCO 1.0 (đủ mảng `images`, `categories`, `annotations`).
   - Tọa độ bounding box hợp lệ: `width > 0`, `height > 0`, không tràn ra ngoài kích thước ảnh.
   - Không chứa thuộc tính để trống hoặc giá trị ngoài taxonomy đã khóa.
2. **Kiểm tra trực quan bằng mắt (Visual Inspection):**
   - Độ ôm sát của bounding box (đạt chuẩn IoU kỳ vọng >= 0.70 với các ca rõ).
   - Kiểm tra việc tách biệt đúng box giữa người và xe máy (theo quy tắc R-02).
   - Kiểm tra triệt để các ca loại trừ (không gán nhãn cho tượng, ảnh in, phản chiếu gương, đồ chơi theo R-06, R-07).
3. **Thẩm quyền phê duyệt ca mơ hồ:**
   - Trong quá trình gán cá nhân: Annotator tự quyết định và đánh dấu `can_review=true`.
   - Trong buổi họp calibration (Pha 5): Cả nhóm cùng thảo luận dựa trên bằng chứng ảnh và đưa ra quyết định biểu quyết đồng thuận đa số để cập nhật vào bảng quyết định.

### 5.2. Lịch sử thay đổi tài liệu (Revision History)

| Phiên bản | Nội dung thay đổi chính | Lý do / Dẫn chiếu ca khó thực tế |
|---|---|---|
| `v0` | Bản khởi đầu sơ sài của ban tổ chức (5 lớp định nghĩa 1 câu, thuộc tính định tính không có ngưỡng, chưa có quy tắc ca biên). | Bản gốc ban đầu. |
| `v1.0-pre-102` | Viết lại toàn bộ 5 phần: <br>1. Xác định rõ mục tiêu bài toán ADAS, mô hình cần đoán gì và không đoán gì, nguy cơ âm tính giả/dương tính giả.<br>2. Bổ sung ví dụ và phản ví dụ thực tế cho từng lớp từ 40 ảnh thật.<br>3. Đề xuất bổ sung lớp `bus` (`img_009.jpg`, `img_031.jpg`, `img_037.jpg`) và `tuk_tuk` (`img_004.jpg`, `img_016.jpg`, `img_035.jpg`) theo phép thử lớp.<br>4. Định lượng ngưỡng `occluded_level` cụ thể theo tỷ lệ diện tích % (0, 1–30%, 31–70%, >70–90%).<br>5. Ban hành hệ thống quy tắc mã hóa R-01 đến R-10: chuẩn hóa box người đi xe máy (R-02), xe chở xe (R-03), loại trừ tượng/ma-nơ-canh/poster/đồ chơi (R-06), loại trừ phản chiếu gương (R-07), xử lý hàng hóa thùng xe (R-08), chuẩn hóa ảnh âm tính 0 box (R-09).<br>6. Xây dựng bảng 13 ca khó với ID ảnh thực tế và quyết định rõ ràng. | Khắc phục hoàn toàn sự mơ hồ của v0 sau khi khảo sát thực tế toàn bộ 40 bức ảnh đường phố của bài lab. |
