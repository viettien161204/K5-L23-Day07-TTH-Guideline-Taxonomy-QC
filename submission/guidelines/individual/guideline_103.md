# Guideline gán nhãn — v1 — TTH

- Phiên bản: v1
- Ngày: 16/11/2026
- Người viết: Trần Đức Hòa
- Loại nhãn: bounding box (hộp chữ nhật) trên ảnh đường phố
- Công cụ: CVAT, xuất COCO 1.0

## 1. Mục tiêu

Nhãn dùng để huấn luyện mô hình **Object Detection** phát hiện **người và phương tiện** trên ảnh đường phố, phục vụ nhận biết đối tượng xung quanh trong hệ thống hỗ trợ lái xe (ADAS) và giám sát giao thông. Mô hình cần xác định đúng loại đối tượng và vị trí bounding box của từng đối tượng. Bỏ sót người đi đường hoặc xe đang di chuyển có thể khiến hệ thống không cảnh báo nguy cơ va chạm; nhận nhầm tượng, hình in hoặc phản chiếu là đối tượng thật có thể gây cảnh báo sai. Bộ nhãn chỉ gồm năm class được chỉ định; không tự bổ sung class khác.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa (một câu) | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Xe cơ giới bốn bánh hoặc nhiều hơn có thân xe chủ yếu thiết kế để chở người cá nhân, không có khoang chở hàng dạng thùng tách biệt làm công năng chính. | Sedan, hatchback, SUV; xe con trong `img_015.jpg`. | `van` thân cao khoang rộng; `truck` chở hàng; xe đồ chơi `img_038.jpg`. |
| `truck` | Xe cơ giới có thiết kế chính để chở hàng, thể hiện qua thùng hàng, sàn tải hoặc kết cấu chuyên chở hàng hóa/phương tiện. | Xe tải thùng `img_036.jpg`, xe chuyên chở ô tô `img_002.jpg`, xe bán tải có thùng hàng `img_012.jpg`. | Xe `van` thùng kín liền thân; `car` chở ít hành lý; xe tải mô hình. |
| `van` | Xe cơ giới thân hộp cao với khoang chở người hoặc hàng dạng kín tích hợp trong thân xe, không có sàn tải hoặc thùng hàng hở độc lập phía sau cabin. | Xe van kín trong `img_007.jpg`, xe van thân hộp trong `img_037.jpg` (nếu xác nhận rõ kết cấu). | `truck` thùng kín có cabin và thùng tách biệt; xe buýt lớn; pickup có nắp thùng cao nhưng vẫn là thùng hàng tách cabin. |
| `motorcycle` | Phương tiện hai bánh có động cơ, gồm xe số, xe tay ga và mô tô, dù đang chạy hay đỗ. | Xe tay ga và mô tô tại `img_005.jpg`, `img_006.jpg`, `img_030.jpg`. | Xe đạp, xích lô, tuk-tuk/xe ba bánh `img_004.jpg`, mô hình xe. |
| `person` | Người thật có cơ thể xuất hiện trực tiếp trong cảnh, bao gồm người đi bộ, đứng, ngồi, điều khiển hoặc ngồi sau phương tiện. | Người đi bộ `img_014.jpg`, người đi xe máy `img_005.jpg`. | Tượng `img_017.jpg`, ma-nơ-canh `img_033.jpg`, người trên hình in/bảng quảng cáo. |

Vật **không thuộc lớp nào**: **không gán box và không ép vào class gần giống**. Ví dụ xe đạp, xích lô, tuk-tuk, xe buýt không rõ là van, tượng, mô hình, ảnh in và hình phản chiếu đều không được tạo box theo năm class này. Nếu một khu vực đông hoặc mờ khiến không thể phân biệt từng đối tượng, ghi chú vấn đề cấp ảnh và chuyển review; chỉ sử dụng `ignore_region` khi dự án CVAT có cấu hình vùng bỏ qua riêng, và **không xuất `ignore_region` thành một trong năm class đối tượng**. Người thật đi cùng phương tiện ngoài taxonomy vẫn được gán `person` nếu đủ điều kiện quan sát. Xe bán tải có nắp thùng cao vẫn là `truck` khi cabin và thùng hàng có cấu trúc tách biệt; xe chở ô tô: gán riêng xe chuyên chở và từng ô tô thật nhìn thấy đủ rõ, không gộp thành một box.

### Thuộc tính

| Thuộc tính | Mức cố định | Ý nghĩa và ngưỡng áp dụng |
|---|---|---|
| `occluded_level` | `0` | Không bị **đối tượng khác che** (`0%` diện tích hình dáng ước tính bị che). |
| `occluded_level` | `1` | Bị che ít: lớn hơn `0%` đến hết `25%` diện tích hình dáng ước tính. |
| `occluded_level` | `2` | Bị che trung bình/nhiều: lớn hơn `25%` đến hết `60%`. |
| `occluded_level` | `3` | Bị che rất nhiều: lớn hơn `60%` nhưng vẫn còn phần nhìn thấy đủ để nhận ra class. Không vẽ đối tượng hoàn toàn vô hình (`100%`). |
| `truncated` | `no` | Không có phần cơ thể/thân xe bị cắt bởi **mép khung ảnh**. |
| `truncated` | `yes` | Có bất kỳ phần nào của đối tượng vượt ra ngoài mép khung ảnh; không phụ thuộc mức phần trăm bị cắt. |
| `can_review` | Không tick | Class, số lượng box, đường bao và thuộc tính đều đủ rõ theo guideline. |
| `can_review` | Tick | Còn nghi ngờ class, ranh giới box hoặc mức che; ghi kèm lý do để Reviewer kiểm tra. |

Ước lượng `occluded_level` bằng tỷ lệ **diện tích hình dáng vật thể bị đối tượng khác che / diện tích toàn bộ hình dáng vật thể ước tính × 100%**; đây là ước lượng trực quan nhất quán, không phải tỷ lệ diện tích của box. `truncated` và `occluded_level` độc lập: xe vừa bị vật khác che vừa bị cắt mép ảnh có thể mang cả hai thuộc tính. Tối, mờ do chuyển động và ra ngoài mép ảnh **không tự động** được tính là `occluded_level`. Khi không đủ bằng chứng nhận diện class, không tạo box đoán mò; ghi chú ảnh để review.

## 3. Quy tắc vẽ box

1. **[R01 — Box ôm sát]** Dùng rectangle nhỏ nhất bao hết **phần nhìn thấy** của từng đối tượng thuộc năm class (visible box), không tự suy diễn phần bị che. Box phải bao các phần thân chính nhìn thấy, không lấy bóng đổ, biển số phản chiếu hoặc vật chở theo làm thân xe. Cho phép box của các đối tượng chồng lên nhau.
2. **[R02 — Bị che khuất]** Nếu còn nhìn thấy và nhận diện được class, vẽ box ôm các phần nhìn thấy cùng thuộc một đối tượng, kể cả các phần bị ngắt bởi vật che; gán `occluded_level` theo bảng ở mục 2. Không gán box nếu hoàn toàn bị che hoặc không xác định được class.
3. **[R03 — Cắt mép ảnh]** Giới hạn tọa độ box trong khung ảnh, không kéo box ra ngoài ảnh hoặc ước lượng phần ngoài ảnh; đặt `truncated=yes` khi thân vật bị mép ảnh cắt. Vật vừa che khuất vừa cắt mép được đánh dấu cả hai thuộc tính độc lập.
4. **[R04 — Phản chiếu, hình in, tượng, đồ chơi]** Không vẽ box cho đối tượng chỉ xuất hiện trong gương (`img_020.jpg`), hình in/biển quảng cáo (`img_018.jpg`), tượng (`img_017.jpg`), ma-nơ-canh (`img_033.jpg`), xe mô hình (`img_038.jpg`). Đối tượng thật nhìn trực tiếp qua kính trong suốt vẫn được gán nếu phân biệt được với hình phản chiếu.
5. **[R05 — Xe chủ/ego]** Không gán box cho chính phần đầu/nắp capo, gương hoặc nội thất của xe gắn camera khi chúng lọt vào khung hình; chỉ gán các xe khác trong cảnh.
6. **[R06 — Vật nhỏ/mờ]** Chỉ gán khi phần nhìn thấy có cả chiều rộng và chiều cao **tối thiểu 10 pixel** và nhận diện được class; vật nhỏ hơn ngưỡng hoặc mờ đến mức không nhận diện được thì bỏ qua, có thể ghi chú ảnh nếu quan trọng. Ngưỡng này là lựa chọn QC của v1, không phải yêu cầu từ v0.
7. **[R07 — Đám đông và xe đỗ sát nhau]** Mỗi người thật và mỗi phương tiện hợp lệ là **một instance, một box riêng** khi nhận diện được; không vẽ một box bao cả đám đông hoặc hàng xe (`img_021.jpg`, `img_030.jpg`, `img_032.jpg`). Vùng không tách được từng vật thì bỏ qua và ghi chú review thay vì gán box gộp.
8. **[R08 — Người và xe]** Người lái xe máy, người ngồi sau và bản thân xe máy là **các đối tượng độc lập**: một xe có một người lái → 2 box (`person` + `motorcycle`); một xe có người lái và người ngồi sau → 3 box (2 `person` + 1 `motorcycle`). Box `person` ôm phần cơ thể nhìn thấy, box `motorcycle` ôm phần xe nhìn thấy, không lấy người làm một phần của box xe; đánh giá che khuất riêng cho từng box (`img_005.jpg`, `img_006.jpg`). Người đứng cạnh hoặc dắt xe vẫn có box người và box xe riêng.
9. **[R09 — Vật chở / phần gắn thêm]** Xe tải chở xe con (`img_002.jpg`) được gán một box cho xe tải và box riêng cho từng xe con thật đủ điều kiện R06; không gộp các ô tô chở trên sàn vào box của một chiếc `car`. Hàng hóa, vali, sọt hàng không có box; phụ kiện rời nhô ra như ăng-ten, bạt, hàng chằng buộc không làm nới box thân xe. Xe có nhiều bộ phận nhưng là cùng một chiếc xe vẫn chỉ tạo một box xe.
10. **[R10 — Không thuộc taxonomy]** Không thêm class `bus`, `bicycle`, `tuk-tuk` hoặc chuyển chúng sang class khác chỉ vì hình dáng gần giống. Với xe minibus/van, dựa vào kết cấu thân hộp kín tích hợp (R01 và định nghĩa `van`); trường hợp khó phân biệt được ghi cấp ảnh để Reviewer xác nhận.

## 4. Ca khó

| Ảnh | Tình huống | Rule ID | Quyết định | Lý do |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở nhiều ô tô con trên sàn | R09 | Một box `truck` cho xe chở; các ô tô thật đủ rõ có box `car` riêng. | Các phương tiện là các instance vật lý độc lập. |
| `img_004.jpg` | Tuk-tuk/xe ba bánh | R10 | Không gán `motorcycle`, `car` hay `van`. | Không thuộc năm class định nghĩa; không ép class. |
| `img_005.jpg` | Người lái/ngồi sau xe máy bị chồng lấp | R08, R02 | Box `person` cho từng người nhìn thấy rõ và `motorcycle` cho xe; tính che khuất riêng. | Tránh gộp người và xe thành một object. |
| `img_006.jpg` | Xe máy và xe đạp trong cùng cảnh | R08, R10 | Xe máy và người đủ điều kiện được gán; xe đạp không gán. | Xe đạp nằm ngoài taxonomy, người thật vẫn thuộc `person`. |
| `img_009.jpg` | Phương tiện chở khách lớn/dạng minibus | R10 | Không tự gán `van` nếu không xác định được là thân van kín; ghi chú review khi phân biệt khó. | Không có class `bus`, cần tránh ép nhãn. |
| `img_012.jpg` | Xe bán tải chở hàng | R09; định nghĩa `truck` | Box `truck` cho xe, không gán box cho hàng trên thùng. | Công năng chở hàng và thùng tải tách biệt. |
| `img_015.jpg` | Ô tô bị cắt ở mép ảnh | R03 | Box phần xe trong khung, `truncated=yes`. | Không suy đoán phần nằm ngoài ảnh. |
| `img_017.jpg` | Tượng có hình người | R04 | Không gán `person`. | Không phải người thật trong cảnh. |
| `img_018.jpg` | Biển quảng cáo có hình ảnh | R04 | Không gán vật thể chỉ được thể hiện trong nội dung biển. | Hình in không phải đối tượng giao thông trực tiếp. |
| `img_020.jpg` | Xe xuất hiện trong gương chiếu hậu | R04 | Không tạo box cho ảnh phản chiếu. | Tránh tính trùng và nhầm vật ảnh với đối tượng trực tiếp. |
| `img_021.jpg` | Mật độ xe/người rất đông, che lấp nhau | R07, R02 | Box riêng từng object nhận diện được; bỏ qua object không đủ bằng chứng, không box gộp. | Giữ nguyên tính instance và tránh box suy đoán. |
| `img_023.jpg` | Xích lô/xe đạp ba bánh có người điều khiển | R10, R08 | Không gán box cho xích lô; người thật đủ rõ vẫn gán `person`. | Phương tiện ngoài taxonomy nhưng người không bị loại trừ. |
| `img_027.jpg` | Xe trưng bày/tác phẩm hình xe | R04 | Không gán nếu xác định là tác phẩm mô phỏng không phải xe hoạt động; nếu là xe thật trưng bày, gán theo class và tick `can_review` nếu khó xác nhận. | Phải phân biệt vật mô phỏng với xe thật, không quyết theo vị trí trưng bày đơn thuần. |
| `img_030.jpg` | Nhiều xe máy đỗ sát/chồng nhau | R07 | Một box cho mỗi xe máy đủ rõ, không gộp cả hàng xe. | Tránh biến nhiều instance thành một nhãn. |
| `img_033.jpg` | Ma-nơ-canh trong cửa hàng | R04 | Không gán `person`. | Không phải người thật. |
| `img_035.jpg` | Phương tiện ba bánh chở hàng | R10 | Không gán vào `motorcycle` hay `truck` chỉ dựa công năng. | Taxonomy không bao gồm phương tiện ba bánh. |
| `img_036.jpg` | Xe tải thùng | Định nghĩa `truck`, R01 | Một box `truck` ôm phần xe nhìn thấy, không bao các xe xung quanh. | Kết cấu chở hàng nhận diện được. |
| `img_038.jpg` | Ô tô mô hình trong tủ kính | R04 | Không gán `car`. | Xe thu nhỏ/đồ chơi không phải phương tiện vận hành trong cảnh. |
| `img_040.jpg` | Xe ba bánh cạnh người và ô tô | R10, R08, R03 | Bỏ qua xe ba bánh; gán riêng `person`/`car` đủ điều kiện và đánh dấu `truncated` nếu bị cắt mép. | Phân biệt class và đánh giá từng instance độc lập. |

Nếu chưa thể chốt, tick `can_review` trên **box đã có đủ bằng chứng để tạo**; nếu chưa thể quyết định có tạo box hay không, ghi vấn đề cấp ảnh kèm tên ảnh, vị trí, Rule ID, hai phương án và lý do. Không tạo box giả chỉ để tick `can_review`. Mỗi quyết định cần trích Rule ID trong nhật ký QC.

## 5. QC và phiên bản

- **Ngưỡng QC áp dụng:** kiểm tra lại **100% ảnh** trước khi nộp; người viết tự kiểm tra toàn bộ các box đã tạo và rà soát lại ảnh sau khi bật/tắt hiển thị nhãn. Reviewer chọn ngẫu nhiên **tối thiểu 10% số ảnh** (làm tròn lên) để kiểm tra độc lập, đồng thời kiểm tra **100% ca có `can_review`/ghi chú cấp ảnh**.
- **Tiêu chí đạt:** đúng một trong năm class; đúng nguyên tắc tách instance; box không vượt khung; box chỉ bao phần nhìn thấy; `occluded_level` theo ngưỡng; `truncated` nhất quán; mọi ca mơ hồ đều có ghi chú. **Không có lỗi nghiêm trọng còn tồn** về nhầm class, bỏ sót người/xe đủ điều kiện hoặc gộp sai instance trong mẫu QC. Nếu phát hiện lỗi nghiêm trọng, sửa và kiểm tra lại toàn bộ các ảnh có cùng loại lỗi.
- **Người quyết ca mơ hồ:** Trần Đức Hòa tổng hợp và đề xuất; Reviewer/nhóm Pha 2 phê duyệt khi có xung đột quy tắc. Không tự thay đổi nhãn đã thống nhất nếu chưa ghi phiên bản mới.
- **Nhật ký QC:** ghi `ảnh | vị trí | Rule ID | lỗi/vấn đề | quyết định | người duyệt | ngày sửa` để truy vết.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| `v0` | Bản đầu: năm class, `occluded_level`, `truncated`, `can_review`, quy tắc box định tính. | Chưa đủ ngưỡng định lượng, ranh giới class và quy tắc ca biên. |
| `v1` — 16/11/2026 | Chốt class; box phần nhìn thấy; bốn ngưỡng occlusion; tách người–xe; xử lý đối tượng ngoài taxonomy, vật giả, phản chiếu, vật cắt viền; thêm R01–R10 và ngưỡng QC. | Các tình huống `img_002`, `img_004`, `img_005`, `img_017`, `img_020`, `img_021`, `img_033`, `img_038` và các ca tương tự. |
