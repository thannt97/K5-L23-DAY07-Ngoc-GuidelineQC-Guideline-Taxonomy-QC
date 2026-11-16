# Guideline thành viên 194 — tổng hợp từ công việc chung — Đào Quang Tuấn — 2A202603194

- Họ và tên: Đào Quang Tuấn
- MSHV: 2A202603194
- Mã thành viên dùng trong file và script: 194

- Mã thành viên: 194
- Ngày tổng hợp: 2026-11-16
- Nguồn: guideline nhóm v1.0 và bộ nhãn YOLOv7 đã được 236 kiểm theo xác nhận người dùng.
- Phạm vi file nhãn bàn giao: 20 ảnh, 167 bbox.
- Hình thức: sản phẩm do cả bốn thành viên cùng xây dựng; bản theo mã thành viên dùng để phân chia hồ sơ bàn giao. Không quy riêng tác giả từng ca khó khi nhóm chưa cung cấp phân công chi tiết.
- Danh sách ảnh: img_002.jpg, img_004.jpg, img_006.jpg, img_008.jpg, img_009.jpg, img_012.jpg, img_014.jpg, img_015.jpg, img_018.jpg, img_020.jpg, img_021.jpg, img_024.jpg, img_025.jpg, img_028.jpg, img_030.jpg, img_031.jpg, img_033.jpg, img_036.jpg, img_037.jpg, img_039.jpg.

Quy tắc dưới đây là quy tắc chung được giữ cùng mã R-01…R-11 để kiểm lại phần việc. Phần ca khó là kết quả cả bốn thành viên cùng làm và thống nhất trong tài liệu chung.

## 1. Mục tiêu

Huấn luyện mô hình phát hiện **người và phương tiện thật** (ô tô, xe tải, xe thùng, xe máy) trên ảnh đường phố châu Á, dùng cho camera giao thông.

- Sai nguy hiểm nhất là **bỏ sót người**. Vì vậy người bị che nhiều vẫn gán.
- Sai thứ hai là dạy mô hình coi **vật không thật** là người hay xe.
- Nhầm giữa các lớp xe là lỗi lớp, ít nguy hiểm hơn.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm |
|---|---|---|---|
| `car` | Xe 4 bánh chở người ≤ 9 chỗ, thân liền: sedan, hatchback, SUV, taxi, MPV gia đình. | img_015, img_024, img_031 | Pickup → `truck`; xe 16 chỗ → `van`. |
| `truck` | Xe chở hàng có **cabin tách khỏi thùng hàng** (thùng hở/kín/bồn/camper), kể cả pickup và xe chở xe. | img_002, img_012, img_028, img_036, img_037 | Xe thân hộp liền, không cabin tách → `van`. |
| `van` | Xe 4 bánh thân hộp liền một khối: xe 16 chỗ, xe giao hàng thân liền, **xe khách cỡ nhỏ dài < 8 m**. | img_005, img_016, img_009 | MPV gia đình thấp → `car`; xe buýt lớn → không gán. |
| `motorcycle` | Xe 2 bánh có động cơ (xe số, xe ga, mô tô, xe điện), **kể cả xe có thùng bên**. | img_006, img_010, img_022 | Xe đạp, xích lô, tuk-tuk → không gán. |
| `person` | Người thật đi bộ, đứng, ngồi, lái hay ngồi trên xe (kể cả xe không thuộc lớp nào). | img_014, img_029, img_034 | Tượng, ma-nơ-canh, người in trên biển, phản chiếu → không gán. |
| `ignore_region` | Vùng đám đông / dòng xe dày đặc, quá nhỏ hoặc quá nhoè để gán từng vật. | img_021 (hậu cảnh), img_030 | Không dùng cho vật rõ, gần camera. |

### Vật ngoài taxonomy (chốt từ "chưa chốt" của bản cá nhân)

| Vật | Quyết định v1.0 | Lý do |
|---|---|---|
| Tuk-tuk, xe lam, xích lô (3 bánh) | **Không gán xe**; người lái / người ngồi vẫn gán `person` | Không có lớp; ép vào lớp khác làm bẩn lớp. Đề xuất `three_wheeler` cho phiên bản sau nếu nhiều ảnh. |
| Xe buýt lớn (dài ≥ 8 m) | **Không gán** | Ngoài phạm vi 4 lớp xe. |
| Xe đạp | **Không gán xe**, người gán `person` | Không có động cơ. |

Phép thử lớp / thuộc tính: mô hình có phải tự đoán ra điều này không? Nếu có thì đó là **lớp**. Nếu chỉ dùng để lọc hay QC thì đó là **thuộc tính**. `occluded_level`, `truncated`, `can_review` là thuộc tính.

### Vật không thật → không gán
Gồm tượng, mô hình, đồ chơi, xe điêu khắc / trang trí không chạy được, ma-nơ-canh, người hoặc xe in trên biển quảng cáo / màn hình, phản chiếu trong gương / kính / nước.
**Ngoại lệ:** xe trang trí **đang chạy, có biển số, có người lái** thì vẫn là xe thật, gán theo hình dạng + `can_review` (img_019).

### Thuộc tính

| Thuộc tính | Mức | Ngưỡng |
|---|---|---|
| `occluded_level` | `0` | bị che < 10% |
| | `1` | 10% – < 40% |
| | `2` | 40% – < 80% |
| | `3` | ≥ 80%, vẫn nhận ra lớp |
| `truncated` | `no` / `yes` | `yes` khi ≥ 10% vật nằm ngoài ảnh (box chạm mép) |
| `can_review` | tick | không chắc lớp, không chắc thật/giả, hoặc không chắc ranh giới |

Ước lượng che theo box của **toàn vật**. Phần bị cắt mép **không** tính vào `occluded_level`.

## 3. Quy tắc vẽ box

1. **R-01** — Ôm sát **phần nhìn thấy**, lệch ≤ 2 px. Không vẽ phần đoán sau vật che.
2. **R-02** — Vật bị chia đôi bởi vật che (cột, người) → **một** box bao cả hai mảnh.
3. **R-03** — Cắt mép: box dừng ở mép ảnh, `truncated = yes`.
4. **R-04** — Người trên xe máy → 2 box: `person` (phần người thấy được) + `motorcycle` (chỉ xe).
5. **R-05** — Người trong ô tô kín nhìn qua kính → không gán. Người trên xe hở (tuk-tuk, xích lô, xe đạp) → gán `person`.
6. **R-06** — Xe chở xe (img_002) → box xe chở + box riêng cho từng xe được chở.
7. **R-07** — Hàng chở lớn gắn liền xe → nằm trong box xe. Tượng hay vật không thuộc lớp chở trên xe (img_028) → không mở rộng box ra ngoài xe.
8. **R-08** — Box cao < 20 px (`min_box_height_px`) → không gán riêng; nếu là cụm → `ignore_region`.
9. **R-09** — ≥ 5 vật cùng lớp chồng nhau không tách được → một `ignore_region`; vật tách được bên ngoài vẫn gán riêng.
10. **R-10** — Nhoè / tối nhưng vẫn nhận ra lớp → gán + `can_review`; không nhận ra → không gán (hoặc `ignore_region` nếu là cụm).
11. **R-11** — Xe mang camera (ego) → không gán.

## 4. Ca khó (đã chốt)

| Ảnh | Tình huống | Quyết định |
|---|---|---|
| img_002 | Xe tải chở 8 ô tô | `truck` + 8 `car` |
| img_004, 016, 035, 040 | Tuk-tuk / xe lam | Không gán xe; người lái `person` + `can_review` |
| img_009 | Xe khách cỡ nhỏ | `van` (dài < 8 m) |
| img_011 | Xe điêu khắc phế liệu, tượng | Không gán |
| img_014 | Người/xe nhoè chuyển động | Nhận ra: gán + `can_review`; mép quá nhoè: `ignore_region` |
| img_017, 027 | Tượng người cưỡi ngựa, tượng xe trên mộ | Không gán |
| img_018 | Người trên biển quảng cáo; xe sau hàng rào | Không gán người in; xe gán `occluded_level=2` |
| img_019 | Xe diễu hành chạy được, có biển số | `car` + `can_review`; nhân vật hoá trang không gán |
| img_020, 033 | Gương chiếu hậu, ma-nơ-canh và phản chiếu | Không gán |
| img_021, 030 | Dòng/bãi xe máy dày đặc | Tiền cảnh gán từng vật, hậu cảnh `ignore_region` |
| img_022 | Xe máy có thùng bên | `motorcycle` bao cả thùng |
| img_023, 029 | Xích lô, xe đạp | Không gán xe, gán người |
| img_024 | Ô tô trong bảo tàng, xe trên nóc xe | `car` từng xe |
| img_028 | Bán tải chở tượng ngựa | `truck`, ngựa không gán |
| img_037 | Bán tải gắn thùng camper; xe buýt bên trái | `truck`; xe buýt không gán |
| img_038 | Xe đồ chơi | Không gán |

## 5. QC và phiên bản

- Ngưỡng lấy từ `taxonomy.json > rules` (không đổi): IAA IoU ≥ 0.85, đồng thuận lớp ≥ 95%; IoU box lớn (≥ 50 px) ≥ 0.85, box nhỏ ≥ 0.7; Release chỉ khi tỉ lệ ảnh lỗi ≤ 2%.
- Ca mơ hồ: người gán tick `can_review`; người chốt quyết ở calibration và ghi một dòng vào `calibration_decisions.csv`.
- Mọi thay đổi làm đổi nhãn → tăng phiên bản guideline + taxonomy.

| Phiên bản | Thay đổi | Lý do (ca nào) |
|---|---|---|
| v0 | Bản khởi đầu của giảng viên. | — |
| v0.1 | Bản cá nhân: định nghĩa + ví dụ/phản ví dụ, ngưỡng %, quy tắc box, ca khó. | v0 không trả lời các ca ở mục 4. |
| v1.0 | Chốt: tuk-tuk / xích lô / xe đạp / xe buýt lớn **không gán xe** nhưng gán người; `van` gồm xe khách nhỏ < 8 m; `motorcycle` gồm xe có thùng bên; xe trang trí chạy được = xe thật; giữ nguyên 6 lớp, cập nhật định nghĩa trong `taxonomy.json`. | img_004, 009, 016, 019, 022, 023, 029, 035, 037, 040 |

## Đối chiếu mã thành viên

| Mã thành viên | Họ và tên | MSHV |
|---|---|---|
| 202 | Nguyễn Thanh Thản | 2A202603202 |
| 231 | Đinh Tuấn Thành | 2A202603231 |
| 194 | Đào Quang Tuấn | 2A202603194 |
| 236 | Nguyễn Thị Ngọc | 2A202603236 |
