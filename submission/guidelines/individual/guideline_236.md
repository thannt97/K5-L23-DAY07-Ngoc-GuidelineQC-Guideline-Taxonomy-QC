# Guideline gán nhãn cá nhân — v0.1

> Bản đề xuất cá nhân trước buổi họp chốt guideline nhóm. Các đề xuất thêm lớp cần được nhóm thống nhất và cập nhật taxonomy trước khi gán nhãn v1.0.

- Phiên bản: v0.1-cá-nhân
- Ngày: 2026-11-16
- Mã thành viên: 236
- Loại nhãn: bounding box 2D trên ảnh đường phố
- Công cụ và định dạng dự kiến: CVAT, COCO 1.0

## 1. Mục tiêu

Gán nhãn người và phương tiện thật đang hiện diện trong ảnh để huấn luyện mô hình phát hiện đối tượng 2D. Nhãn phải phản ánh những gì quan sát được trên ảnh, không suy diễn phần bị che hoặc nằm ngoài khung. Bỏ sót, gán nhầm vật không thật và không thống nhất cách tách các đối tượng trên xe chở xe có thể làm sai dữ liệu huấn luyện; các ca chưa chắc chắn cần được đánh dấu để reviewer xem lại.

## 2. Lớp và thuộc tính

### Lớp

| Lớp | Định nghĩa | Ví dụ | Phản ví dụ / dễ nhầm với |
|---|---|---|---|
| `car` | Ô tô chở người, gồm sedan và SUV; xe con được chở trên xe chở xe là một đối tượng riêng nếu nhận diện được đầy đủ. | Các xe con trên hai tầng xe chở ở `img_002.jpg`. | Vỏ thân xe rời, bộ phận xe hoặc hình xe trên biển quảng cáo không phải `car`. |
| `truck` | Phương tiện có thân/thùng hàng hoặc sàn chở hàng rõ ràng; gồm xe tải và xe đầu kéo chở xe. | Xe chở xe ở `img_002.jpg`. | Xe buýt chở hành khách không phải `truck`; xe thùng kín cỡ nhỏ thuộc `van`. |
| `van` | Phương tiện chở người/hàng có khoang thân kín, liền khối và nhỏ hơn xe tải hàng thông thường. | Xe thùng kín cỡ nhỏ. | Xe tải có sàn/thùng hàng tách biệt thuộc `truck`. Bán tải gắn nắp thùng cao cần reviewer xem nếu không rõ đó là khoang kín hay thùng hàng. |
| `motorcycle` | Xe máy hoặc mô tô thật, có hai hoặc ba bánh và nhận diện được là phương tiện có động cơ. | Xe máy trong ảnh đường phố. | Xe đạp, xích lô không động cơ, xe máy đồ chơi và xe máy trong gương không gán lớp này. |
| `person` | Người thật đang đi bộ, đứng, ngồi hoặc lái xe; mỗi người là một đối tượng riêng. | Người đi bộ và người ngồi trên xe máy. | Tượng, ma-nơ-canh, người in trên biển và người trong ảnh phản chiếu không phải `person`. |
| `bus` *(đề xuất thêm)* | Phương tiện có thân dài, khoang hành khách đóng và được nhận diện rõ ràng là xe buýt. | Xe buýt ở `img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg`. | Không gồm xe tải thùng kín hoặc xe van. |
| `tuk_tuk` *(đề xuất thêm)* | Phương tiện ba bánh có động cơ, dạng xe tuk-tuk/xe ba gác chở người trên đường. | Tuk-tuk ở `img_004.jpg`, `img_016.jpg`, `img_035.jpg`. | Không gồm xe đạp, xích lô không động cơ hoặc xe máy đồ chơi. |
| `ignore_region` | Vùng bỏ qua, không phải lớp đối tượng; dùng cho đối tượng quá nhỏ hoặc đám đông không thể tách box theo quy tắc bên dưới. | Vùng chứa các đối tượng cao dưới 20 px không thể gán box riêng. | Không dùng thay cho đối tượng rõ ràng chỉ để tránh xử lý ca khó. |

`bus` và `tuk_tuk` chưa có trong taxonomy v0. Tôi đề xuất thêm hai lớp này khi nhóm chốt v1.0; không tự ý gán một lớp khác cho chúng. Nếu taxonomy không được mở rộng, các ca này cần được nhóm/reviewer quyết định trước khi gán nhãn.

### Thuộc tính

| Thuộc tính | Giá trị và ngưỡng | Cách gán |
|---|---|---|
| `occluded_level` | `0`: không bị che; `1`: bị che 1–30% diện tích đối tượng; `2`: bị che 31–70%; `3`: bị che trên 70% nhưng vẫn nhận diện chắc chắn. | Ước lượng phần diện tích thật bị vật khác che, không tính phần nằm ngoài khung ảnh. Nếu không thể nhận diện lớp chắc chắn thì không gán lớp đó; ghi vấn đề để review. |
| `truncated` | `no` / `yes` | `yes` chỉ khi đối tượng bị cắt bởi cạnh ảnh; vật khác che không làm `truncated=yes`. |
| `can_review` | `true` / `false` | Bật khi còn bất đồng hợp lý về lớp, ranh giới box hoặc tính thật của đối tượng. Tắt nếu quy tắc đã cho kết luận rõ ràng. |

Giá trị mặc định khi đối tượng không bị che và không bị cắt: `occluded_level=0`, `truncated=no`, `can_review=false`.

## 3. Quy tắc vẽ box

1. Mỗi đối tượng thật, nhìn thấy đủ và thuộc lớp trong taxonomy phải có một box riêng. Box ôm sát phần nhìn thấy; không nới rộng để đoán phần bị che hoặc phần nằm ngoài ảnh.
2. Nếu đối tượng chạm/bị cắt bởi mép ảnh, kéo box đến mép ảnh và đặt `truncated=yes`. Nếu đối tượng chỉ bị vật khác che, box vẫn ôm phần nhìn thấy và `truncated=no`.
3. Người và phương tiện là hai đối tượng khác nhau. Người lái/ngồi trên xe máy có một box `person` và một box `motorcycle`; không gộp người và xe vào một box. Áp dụng tương tự cho người ngồi trên phương tiện khác.
4. Trên xe chở xe, gán một box `truck` cho xe chở và một box `car` cho từng xe con hoàn chỉnh, nhìn thấy và nhận diện được. Không gán box riêng cho sàn chở, bánh xe bị che, vỏ thân rời hoặc bộ phận không còn là một xe hoàn chỉnh. Áp dụng cho `img_002.jpg`; thân vỏ trên nóc ở `img_024.jpg` không được tính là xe con.
5. Box xe tuk-tuk chỉ bao phương tiện. Hàng hóa/bao tải chồng trên xe không nằm trong box tuk-tuk và không được gán lớp phương tiện riêng. Ở `img_035.jpg`, không kéo box tuk-tuk lên bao trùm khối hàng quá khổ.
6. Chỉ gán người/phương tiện thật trong cảnh. Loại tượng, tượng xe, ma-nơ-canh, đồ chơi, hình in/quảng cáo và phản chiếu trong gương. Ví dụ: tượng ở `img_011.jpg`, `img_017.jpg`, `img_027.jpg`; hình quảng cáo ở `img_018.jpg`; gương ở `img_020.jpg`; ma-nơ-canh ở `img_033.jpg`; đồ chơi ở `img_038.jpg`. Ảnh không còn đối tượng hợp lệ vẫn phải có trong COCO với 0 annotation.
7. Đối tượng có chiều cao dưới 20 px không tạo box lớp riêng; đánh dấu vùng bỏ qua `ignore_region`. Nếu nhiều người/xe chen lẫn đến mức không thể tách từng đối tượng, dùng một `ignore_region` cho đúng vùng không thể gán riêng; không dùng nó cho các đối tượng rõ ràng ở bên cạnh.
8. Xe buýt được gán `bus`, không gán `truck` chỉ vì xe có kích thước lớn. Tuk-tuk được gán `tuk_tuk`. Xe đạp và xích lô không động cơ không thuộc các lớp phương tiện hiện có; không ép chúng vào `motorcycle`.
9. Nếu ảnh mờ nhưng vẫn nhận diện chắc chắn được đối tượng và lớp, gán nhãn theo phần nhìn thấy. Nếu không thể kết luận lớp/ranh giới box có căn cứ, bật `can_review` nếu đã vẽ box hoặc ghi vấn đề cấp ảnh nếu chưa thể vẽ.

## 4. Ca khó và quyết định cá nhân

| Ảnh | Tình huống | Rule ID | Quyết định ban đầu | Lý do / xử lý |
|---|---|---|---|---|
| `img_002.jpg` | Xe tải chở nhiều xe con trên hai tầng | R-01 | Một box `truck` cho xe chở và một box `car` cho từng xe con hoàn chỉnh nhìn thấy; không vẽ box cho sàn/bộ phận xe. | Các xe con là đối tượng riêng, không gộp chung vào box xe tải. |
| `img_004.jpg`, `img_016.jpg` | Tuk-tuk/xe ba bánh có động cơ | R-02 | Đề xuất lớp `tuk_tuk`; mỗi xe một box. | Taxonomy v0 không có lớp phù hợp; cần nhóm chấp thuận lớp mới trước khi gán chính thức. |
| `img_009.jpg`, `img_031.jpg`, `img_037.jpg`, `img_040.jpg` | Xe buýt | R-03 | Đề xuất lớp `bus`; mỗi xe một box. | Không ép xe buýt vào `truck` hay `van`. |
| `img_011.jpg`, `img_017.jpg`, `img_027.jpg` | Tượng người/tượng xe | R-04 | Không gán nhãn đối tượng. | Vật không phải người/phương tiện thật đang hiện diện. |
| `img_018.jpg`, `img_020.jpg` | Người trên biển quảng cáo / phản chiếu trong gương | R-04 | Không gán `person` hoặc phương tiện. | Hình in và ảnh phản chiếu không phải đối tượng thật trong cảnh. |
| `img_024.jpg` | Thân vỏ xe trên nóc phương tiện | R-05 | Không gán `car` cho thân vỏ rời; chỉ gán phương tiện nếu nhận diện được. | Thân vỏ không phải một xe hoàn chỉnh. |
| `img_027.jpg` | Tượng có dáng xe bán tải | R-04 | Không gán lớp xe. | Hình dáng giống xe không biến vật phi thực thành phương tiện thật. |
| `img_033.jpg`, `img_038.jpg` | Ma-nơ-canh và xe đồ chơi trong tủ kính | R-04 | Không gán `person`/lớp phương tiện. | Không phải đối tượng thật đang tham gia giao thông. |
| `img_035.jpg` | Tuk-tuk chở bao tải/hàng hóa cao | R-06 | Box chỉ ôm tuk-tuk; không gồm khối hàng trên nóc. | Hàng hóa không phải phương tiện; box cần nhất quán giữa các ảnh. |
| `img_037.jpg` | Bán tải có nắp thùng cao, dễ nhầm `truck`/`van` | R-07 | Nếu khoang hàng được đóng kín liên tục bởi thân cứng thì tạm xếp `van`; nếu là sàn/thùng hàng rõ ràng thì `truck`. Bật `can_review` để nhóm chốt. | Ranh giới lớp chưa rõ từ mô tả hiện có; không tự ý kết luận taxonomy cả nhóm. |
| `img_006.jpg`, `img_029.jpg`, `img_039.jpg` | Xe đạp/xích lô không động cơ | R-08 | Không gán `motorcycle`. | Không phải xe máy; ghi vấn đề nếu nhóm muốn mở rộng phạm vi lớp. |
| `img_014.jpg`, `img_015.jpg` | Phương tiện/đối tượng bị mờ do chuyển động | R-09 | Gán lớp nếu hình dạng còn đủ để nhận diện chắc chắn; nếu không, đánh dấu `can_review`/ghi vấn đề ảnh. | Mờ không tự động là lý do loại; cần phân biệt không rõ lớp với chỉ kém nét. |

Nếu nhóm quyết định khác các đề xuất trên, ghi lại lý do trong guideline nhóm và sử dụng thống nhất quy tắc đã khóa trong tất cả lượt gán.

## 5. QC và phiên bản

- Trước khi khóa guideline, đối chiếu taxonomy và bảo đảm mọi lớp được đề xuất đã được nhóm chấp thuận. Không thay đổi các ngưỡng trong `taxonomy.json` của lab.
- Box đối tượng cao từ 50 px trở lên: đối chiếu IoU với ngưỡng `0.85`; box nhỏ hơn 50 px: `0.70`. Khi ghép box để tính IAA, ghép 1–1 với ngưỡng IoU `0.50`; IAA đạt mục tiêu IoU trung bình `0.85` và cùng lớp `95%`.
- Chỉ đọc IAA khi đủ 100% lượt gán được phân công. Box không ghép được và bất đồng `ignore_region` phải được xem riêng; IoU cao không chứng minh rằng cả hai người không cùng sai.
- QC Release theo ngưỡng lab: ảnh có lỗi / ảnh đã duyệt không quá `2%`; lab yêu cầu review cả lô để Release. Nếu chỉ review lấy mẫu thì kết quả bàn giao là Hold. Lỗi lặp lại theo cùng quy tắc cần xử lý Rework.
- Ghi rõ ca chưa chắc, ảnh/vùng liên quan, rule ID, người chốt và kết quả kiểm lại. Dùng `can_review` để yêu cầu xem lại, không dùng nó thay cho ghi chú khi chưa thể tạo box.

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v0.1-cá-nhân | Viết quy tắc phân lớp, box, thuộc tính và các ca biên từ bộ 40 ảnh; đề xuất lớp `bus` và `tuk_tuk`. | Làm rõ các điểm chưa được quy định trong guideline v0; để nhóm xem xét trước khi khóa v1.0. |
