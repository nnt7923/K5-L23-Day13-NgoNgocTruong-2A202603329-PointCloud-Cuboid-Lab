# Báo cáo Day 13 — Ngô Ngọc Trường, 2A202603329 (MSSV thật: 2A202600329)

## 1. Kết luận trước
Mã số trong `configs/lab.ini`: **2A202603329**. Đây là mã gõ nhầm lúc tạo task; MSSV thật của tôi là **2A202600329**. Toàn bộ cuboid đã vẽ trên task `D13-2A202603329`, nên bài nộp theo mã này và tôi đã báo coach. 10 frame: s01f12, s01f42, s02f04, s02f20, s03f05, s03f20, s04f17, s04f36, s05f04, s05f27. Máy Windows, PowerShell 5.1, trình duyệt Edge; CVAT server 2.74.1 (SDK 2.83.0 báo lệch phiên bản nhưng `sync` vẫn chạy). Bạn ngồi cạnh: không đổi kiểm được (xem mục 5).

Tổng **133 cuboid** (105 Car, 13 Pedestrian, 9 Cyclist, 3 Truck, 3 Bus). Frame ít nhất là s02f04 và s05f27 với 9 hộp: cảnh ít vật rõ trong 50 m. `selfcheck` cờ **18/133** cuboid, 12 trong số đó ở frame đầu s01f12. Tôi tin nhất phần đáy hộp và hướng của xe cùng chiều (z đáy trung vị −1,79 m, yaw xe con phía trước ≈ 270°). Yếu nhất là **kích thước xe ở xa và xe bị che** trong frame đầu: nhiều hộp Car chỉ dài 2–2,7 m vì tôi vẽ theo phần điểm nhìn thấy thay vì suy ra cả thân xe.

Cách dùng CVAT: xoay khung 3D bằng chuột trái, dời bằng chuột phải, lăn để phóng; nháy đúp chỗ trống để về góc nhìn mặc định. Vẽ cuboid: chọn label → **Shape** → nháy đúp lên cụm điểm, rồi chỉnh tâm, dài, rộng, yaw ở **Top** và đáy, chiều cao ở **Side/Front**. Bật Appearance → **Cuboid orientation** để thấy mũi tên hướng; xem cuboid trên ảnh camera bằng `python scripts/lab.py preview` (ảnh trong `work/xem/`).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ. Ảnh: `t1-s01f12-3d`, `t1-s01f12-top`, `t1-s01f42-3d`, `t1-s01f42-top`.

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f12 | Phía trước (−y) ở cạnh trái ảnh; +x xuống dưới (bên trái xe), +y sang phải (phía sau) | Phẳng; thấp hơn LiDAR ~1,8 m | ~44 m (xe con, chỉ ~18 điểm) | Xe con, xe tải, người đi bộ | 8/12 hộp ở phía sau (+y), xe tải ~32 m phía trước | Cụm rất thưa ~54 m phía trước: có thể là xe nhưng chỉ 17 điểm |
| s01f42 | Như trên: trước ở cạnh trái; +x xuống, +y sang phải | Phẳng; thấp hơn LiDAR ~1,8 m | ~40 m | Xe con, xe đạp/xe máy, người đi bộ | Xe hai bánh và người đi bộ ở hai bên đường | Các vật ~40 m chỉ còn vài chục điểm, khó phân biệt xe con với xe van |

Chỉ camera cho biết: màu xe và loại xe (sedan, van, xe buýt hay xe tải), người có đang lái xe máy hay dắt xe, và nội dung biển báo, đèn tín hiệu. Chỉ LiDAR cho biết: khoảng cách thật theo mét tới từng xe, độ cao mặt đường so với cảm biến (~−1,8 m), và vật ở phía sau, hai bên trong vùng các camera trước không thấy.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s02f04 | Cạnh trái | Có xe buýt; ít vật rõ (9 hộp) |
| s02f20 | Cạnh trái | Nhiều xe ngược chiều (yaw ≈ 90°) |
| s03f05 | Cạnh trái | Chỉ có xe con, vật xa nhất ~34 m |
| s03f20 | Cạnh trái | Cảnh đông (25 hộp), nhiều người đi bộ đứng sát nhau |
| s04f17 | Cạnh trái | Dòng xe con dày ở cả trước và sau |
| s04f36 | Cạnh trái | Có xe tải; vật xa ~48 m rất thưa |
| s05f04 | Cạnh trái | Có xe tải |
| s05f27 | Cạnh trái | Mặt đường thấp hơn các frame khác (z đáy trung vị −1,93 m); khoảng hở với mặt đường chỉ −0,01 đến 0,12 m nên hộp vẫn chạm đường |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f12, 1 | Car | Có, yaw 270,5° (xe cùng chiều phía trước) | Có: z đáy −1,79 m, khoảng hở với mặt đường −0,03 m | Có, `t2-a-car` | Có: 3,82 × 1,54 × 1,61 m (đã sửa width từ 1,29 m) |
| (b) xe buýt hoặc xe tải | s01f12, 5 | Truck | Có, yaw 285,1° | Có nhưng sát ngưỡng: z đáy −1,43 m, khoảng hở với mặt đường quanh hộp 0,26 m (ngưỡng 0,3 m) | Có, `t2-b-bus-truck` | Dài 10,17 m hợp lý; rộng 3,80 m vượt khoảng 1,6–3,2 m, có 334 điểm sát ngoài mép: hộp có thể lệch ngang khỏi xe |
| (c) người đi bộ hoặc xe hai bánh | s01f12, 8 | Pedestrian | Chưa đúng: yaw để 0°, chưa xoay theo hướng người đi | Có: z đáy −1,62 m, khoảng hở 0,10 m | Có, `t2-c-ped-cyclist` | Cao 1,94 m hợp lý; dài 0,98 m, rộng 1,00 m vượt khoảng 0,5–0,9 m của người lớn, hộp ôm cả điểm vật bên cạnh (421 điểm sát mép) |

Vòng hiệu chuẩn: tôi đã vẽ cuboid trong task chung `D13-HC485087` nhưng chưa có kết quả so sánh với các bạn để ghi điểm lệch cụ thể. Vật bỏ qua vì không đủ cơ sở: không bỏ hẳn vật nào, nhưng s01f12 #7 (~54,5 m, 17 điểm) lẽ ra nên bỏ qua vì ngoài 50 m và LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s02f00, `t4-1-bong-che`, task chung D13-HC485087 | Giả thuyết: xe buýt sát bên trái (~4 m) chặn tia LiDAR | Các vòng quét dừng ở thân xe buýt, phía sau nó gần như không có điểm | Vùng trống là chưa quan sát được, không phải đường trống; vật bị che thì bật occluded và vẽ phần suy luận được |
| 2. Thưa ở xa | s01f12 #6, `t4-2-thua-o-xa`, task của tôi | Giả thuyết: LiDAR 40 tia, càng xa khoảng cách giữa các tia càng lớn | Xe phía sau cách 35,9 m chỉ có 47 điểm trong hộp (xe ở 5,9 m có 2270 điểm); Top/Side/Front gần như trống | Chỉ vẽ khi thấy khoảng 15–20% thể tích; không thì bỏ qua, ghi LOW_VISIBILITY |
| 3. Phản xạ thấp / ít điểm bất thường | s04f02, `t4-3-phan-xa-thap`, task chung D13-HC485087 | Xe van tối màu, nhiều kính phản xạ ít tia (giả thuyết) | Xe van sát phía trước bên phải có ít điểm dù rất gần; biển báo bên trái thì điểm sáng hẳn | Ghi giả thuyết, không khẳng định; vẫn vẽ theo hình dạng suy ra từ camera |
| 4. Điểm lạ | s02f35, `t4-4-diem-la`, task chung D13-HC485087 | Giả thuyết: tia phản xạ nhiều lần (multipath) hoặc nhiễu cảm biến, chưa rõ | Cụm ~55 điểm phía sau bên trái, cách ~29 m, ở z ≈ −13,3 m, thấp hơn mặt đường quanh đó (~−1,9 m) khoảng 11 m; camera hướng đó không thấy vật | Không gán thành vật |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603329-bangA.csv`; số điểm từ cột `points_in` của `-tukiem.csv`.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f12, 1, Car | −0,984 | 1,608 | −1,788 | 5,88 | 5,96 | 270,5 | Chạm đường |
| s01f12, 5, Truck | −0,070 | 2,722 | −1,431 | 31,94 | 31,94 | 285,1 | Cao hơn mặt đường ~0,36 m, hơi lơ lửng |
| s01f12, 8, Pedestrian | −0,647 | 1,939 | −1,616 | 7,69 | 7,72 | 0,0 | Chạm đường |

(a) Gần nhất: s02f04 #1 Car, dist_m 3,18 m, r 3,35 m. Xa nhất: s01f12 #7 Car, dist_m 54,51 m, r 54,52 m; hộp này ngoài giới hạn 50 m.
(b) Người đi bộ s01f12 #8: 0,98 × 1,00 × 1,94 m; cao hợp lý, rộng 1,00 m hơi lớn vì hộp ôm cả tay chân và vài điểm vật bên cạnh. Xe con s01f12 #1: 3,82 × 1,54 × 1,61 m, nằm trong khoảng của bảng 4.3.
(c) Xe cùng chiều phía trước s01f12 #1 yaw 270,5° (gần 270°); xe có yaw ngược lại: s02f20 #4 yaw 90,0°, tôi hiểu là xe ngược chiều.

Mật độ theo khoảng cách (4 cuboid ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s02f04, 1, Car | 3,18 | 2294 |
| s04f17, 4, Car | 16,84 | 682 |
| s01f42, 5, Car | 21,49 | 300 |
| s04f17, 13, Car | 33,52 | 135 |

Quy luật: xa gấp 2 lần (16,84 m → 33,52 m) thì số điểm còn khoảng 1/5 (682 → 135); từ 3,2 m ra 33,5 m số điểm giảm khoảng 17 lần, nhanh hơn tỉ lệ 1/khoảng cách. Trung vị xe con không bị cờ của tôi là ~2282 điểm ở 0–10 m, 682 ở 10–20 m, 300 ở 20–30 m, 122 ở 30–40 m, 54 ở 40–50 m; khá gần đường cong slide (1968, 718, 214, 72), chỉ cao hơn một chút ở 20–40 m.

Tự kiểm: tôi đã sửa các hộp Car ở s01f12 sau lần `sync` đầu: #1 (width 1,29 → 1,54 m, hết cờ), #10 (đáy bị chìm 0,69 m → đã nâng lên), và kéo dài #3, #4. Số hộp có cờ giảm từ 19 xuống 18. Giữ nguyên: s01f12 #9 và #10 (length < width, yaw 0°) vì chỉ thấy một mặt xe bị che, tôi chưa chắc hướng thân xe; các cờ "điểm sát ngoài mép" (s01f12 #8, s01f42 #10, s03f20 #18) vì có thể do vật khác đứng sát cạnh, tôi chưa kiểm hết. Hộp s03f20 #18 và #19 chồng nhau cần xem lại là một hay hai người.

Đổi kiểm: chưa thực hiện.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model học trên point cloud có tọa độ thật, nên nhãn đo bằng mét mới cho model biết kích thước và khoảng cách thật của vật, không đổi dù vật ở gần hay xa như pixel; nhờ vậy xe tự hành tính được khoảng cách an toàn. Vật thưa điểm cần quy tắc riêng vì nếu ép model học hộp đầy đủ từ vài điểm, model sẽ học "bịa" vật; quy tắc 15–20% thể tích và LOW_VISIBILITY giữ nhãn nhất quán và trung thực.

(b) Dùng AI: không dùng

(c) Điều còn chưa chắc: kích thước các xe ở xa hoặc bị che trong s01f12 (nhiều hộp Car dài dưới 3 m), hướng của s01f12 #9 và #10, và hộp s01f12 #7 ở ngoài 50 m. Việc tạo task bằng mã gõ nhầm (2A202603329 thay vì 2A202600329) là lỗi của tôi lúc cấu hình.
