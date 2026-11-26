# Báo cáo Day 13 — Trương Tuấn Vũ, 2A202603078

## 1. Kết luận trước

Task cá nhân: `D13-2A202603078` (task/job 23). Môi trường: Windows, Google Chrome, CVAT 2.74.1. Bộ frame: `s01f10`, `s01f41`, `s02f01`, `s02f28`, `s03f06`, `s03f53`, `s04f31`, `s04f56`, `s05f02`, `s05f46`.

Bảng A có **62 cuboid Shape**: 52 Car, 3 Truck, 3 Bus và 4 Cyclist. Số hộp theo thứ tự 10 frame là **8, 9, 5, 5, 6, 6, 6, 6, 6, 5**; ít nhất là `s02f01`, `s02f28`, `s05f46`, cùng 5 hộp. Bản gán hiện tại ưu tiên vật gần, nhận ra được; số hộp chưa chứng minh đã tìm hết vật trong cảnh. Phiếu tự kiểm cờ **3/62 cuboid**; tâm và đáy của vật gần có cơ sở kiểm tra tốt hơn, còn hướng đầu xe, kích thước vật bị che và vật xa cần rà soát.

Cách dùng CVAT:

- Kéo chuột trái ở vùng trống để xoay và nhìn từ trên xuống; kéo chuột phải để dời, con lăn để phóng/thu cảnh.
- Chọn class và **Shape**, nháy đúp để đặt hộp; chỉnh tâm, dài/rộng/yaw ở Top, chỉnh chiều cao và đáy ở Side/Front.
- Bật **Cuboid orientation**, căn vạch đỏ về đầu xe; đối chiếu camera nhận đầu/đuôi và dùng `preview` để xem hộp chiếu lên ảnh.

## 2. Đọc frame (Bước 2)

Ảnh: `t1-s01f10-3d.png`, `t1-s01f10-top.png`, `t1-s01f41-3d.png`, `t1-s01f41-top.png`. Hệ VF: +x sang trái xe thu dữ liệu, +y ra sau, +z lên; phía trước là −y. Hai ảnh top hiện có còn nghiêng và chưa đánh dấu trục theo yêu cầu, cần kiểm lại trước khi nộp.

| Frame | Hướng trên ảnh top hiện có | Mặt đường | Vật xa nhất đã gán/đọc được theo bảng A | Loại vật thấy | Phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f10 | Trước −y lên trái; +y xuống phải; +x xuống trái | Khá phẳng ở vùng gần; đáy Car gần −1,831 m | Bus ở 46,11 m | Car, Bus, Truck | Hai Car phía sau ở 37,36–38,63 m; Truck sau-trái | Chưa có ghi chép xác nhận class của các cụm thưa ngoài hộp |
| s01f41 | Trước −y lên trái; +y xuống phải; +x xuống trái | Vùng Car gần có đáy −1,827 m; nền vùng Cyclist có cao độ khác | Car ở 26,54 m | Car, Cyclist | Có Car phía sau và Cyclist bên phải; cần camera bên/sau | Các cụm sát lề và sau vật che còn cần đối chiếu camera |

Camera cho biết màu xe, chi tiết bề ngoài như kính/cửa, và chi tiết người đang lái hay dắt xe hai bánh khi ảnh đủ rõ. LiDAR cho phép đọc vị trí tương đối theo mét, hình dạng/độ cao mặt đường, và hình học của vật phía sau hoặc hai bên ngoài cam01. Hai nguồn bổ sung cho nhau; ảnh camera đơn lẻ không trực tiếp cung cấp các số đo 3D này.

Tám frame còn lại được đọc theo cùng hệ trục: khi quy về nhìn thẳng từ trên xuống, đặt −y lên trên, +x sang trái, +y xuống dưới. Đây là quy ước đọc; cạnh màn hình thực tế đổi khi xoay cảnh.

| Frame | Phía trước khi quy về nhìn từ trên xuống | Điểm cần lưu ý |
|---|---|---|
| s02f01 | −y, đặt ở cạnh trên | Bus gần bên trái tạo vùng thiếu điểm phía sau nó |
| s02f28 | −y, đặt ở cạnh trên | Có xe ngược chiều; phân biệt đầu/đuôi bằng camera |
| s03f06 | −y, đặt ở cạnh trên | Cụm điểm tách rời dưới mặt đường, không gán thành vật |
| s03f53 | −y, đặt ở cạnh trên | Bus phía sau ở 41,71 m bị che, điểm thưa |
| s04f31 | −y, đặt ở cạnh trên | Cyclist bên phải/phía sau, cần camera ngoài cam01 |
| s04f56 | −y, đặt ở cạnh trên | Car ở 34,40 m và 38,48 m; kiểm đáy theo đường cục bộ |
| s05f02 | −y, đặt ở cạnh trên | Car trước-trái có cảnh báo đáy, xem lại nền bị che |
| s05f46 | −y, đặt ở cạnh trên | Hai Car bên trái có cảnh báo đáy; có Truck sau-trái |

## 3. Ba cuboid có hướng dẫn (Bước 3)

`idx` dưới đây là số thứ tự trong **bảng A**, không phải ID trên thẻ Objects của CVAT.

| Vật / ảnh | Frame, idx, class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Top/Side/Front và camera | Kích thước so bảng 4.3 |
|---|---|---|---|---|---|
| Xe con gần, `t2-a-car.png` | s01f41, #1, Car | 89,2°; cần xác nhận đầu/đuôi, chưa kết luận hướng đúng | −1,827 m; gap −0,01 m, sát nền cục bộ | Ảnh có hộp ở ba view và camera | 4,528 × 2,024 × 1,533 m; rộng vượt mốc sedan 1,90 m, cần kiểm gương/phần hộp bao dư |
| Xe buýt, `t2-b-bus-truck.png` | s02f01, #1, Bus | 270,0°, phù hợp −y; cần đối chiếu đầu xe trên ảnh | −1,807 m; gap +0,03 m | Ảnh có hộp ở ba view và camera | 11,600 × 2,731 × 3,200 m; dài/cao phù hợp, rộng vượt mốc 2,60 m |
| Xe hai bánh, `t2-c-ped-cyclist.png` | s01f41, #2, Cyclist | 272,8°, gần −y; cần đối chiếu ảnh thấy người lái | −1,400 m; gap +0,02 m, sát nền cục bộ | Ảnh hiện chưa thấy rõ hộp chọn ở cả ba view, cần rà lại | 1,695 × 0,599 × 1,823 m; trong khoảng xe đạp có người |

Ảnh Bus hiện ở frame thứ ba; yêu cầu vật mẫu ưu tiên frame đầu, thiếu loại mới lấy frame hai, nên ảnh này còn cần đối chiếu lại yêu cầu. Vật bỏ qua vì thiếu cơ sở: chưa có danh sách được học viên xác nhận; không suy diễn class cho các cụm chưa nhận dạng rõ ở mục 2.

## 4. Bốn hiện tượng chất lượng (Bước 4)

Tất cả ảnh dưới đây thuộc **task cá nhân `D13-2A202603078`**, không dùng frame mượn. Nguyên nhân chỉ là giả thuyết.

| Hiện tượng | Frame / tên ảnh | Nguyên nhân (giả thuyết) | Quan sát | Hệ quả khi gán nhãn |
|---|---|---|---|---|
| Bóng che | s02f01 / `t4-1-bong-che.png` | Bus chắn tia LiDAR | Sau Bus có vùng thiếu điểm, hai bên vẫn có điểm | Vùng trống có thể chưa được quan sát; bật occluded cho vật bị che, bao phần suy luận có cơ sở |
| Thưa ở xa | s01f10 / `t4-2-thua-o-xa.png` | Mật độ lấy mẫu góc làm điểm trên vật giảm khi xa | Hai Car phía sau ở 37,36 m và 38,63 m có 65 và 49 điểm trong hộp | Dùng quy tắc khoảng 15–20% thể tích hoặc bằng chứng camera; thiếu cơ sở thì bỏ qua, ghi LOW_VISIBILITY |
| Phản xạ thấp / ít điểm bất thường | s01f10 / `t4-3-phan-xa-thap.png` | Kính, vật liệu tối và góc tới có thể làm phản xạ yếu | Xe tối màu gần bên trái có mật độ điểm không đều giữa các bề mặt; ảnh chưa đủ khẳng định nguyên nhân | Đối chiếu camera và các mặt hộp; không thu hộp chỉ theo mảng điểm sáng hoặc khẳng định vật liệu gây mất điểm |
| Điểm lạ | s03f06 / `t4-4-diem-la.png` | Có thể là sai số đo hoặc phản xạ nhiều đường | Cụm nhỏ tách rời dưới dải mặt đường khi xoay nhìn ngang | Không tạo cuboid cho cụm này, không gộp vào vật trên mặt đường |

## 5. Số từ cuboid của tôi (Bước 5)

Nguồn: `D13-2A202603078-bangA.csv` và `D13-2A202603078-tukiem.csv`. Hai file khớp nhau và khớp 62 cuboid đã lưu trên CVAT tại lúc soạn. `idx` theo bảng A; khoảng cách đo đến tâm hộp.

| Cuboid (frame, idx, class) | z (m) | h (m) | z − h/2 (m) | dist_m (m) | r (m) | yaw (°) | Nhận xét |
|---|---|---|---|---|---|---|---|
| s01f10, #2, Car | −1,006 | 1,650 | −1,831 | 5,47 | 5,565 | 269,9 | Sát đường; gap −0,07 m |
| s02f01, #1, Bus | −0,207 | 3,200 | −1,807 | 5,97 | 5,972 | 270,0 | Sát đường; gap +0,03 m |
| s01f41, #3, Cyclist | −0,450 | 1,580 | −1,240 | 11,27 | 11,283 | 272,8 | Cao hơn mốc −1,7 m nhưng gap cục bộ xấp xỉ 0,00 m; cần dùng nền cục bộ để kết luận |

**Gần/xa nhất:** Car `s03f06` #1 có `dist_m = 3,14 m`, `r = √(3,052² + (−0,722)² + (−0,974)²) ≈ 3,284 m`. Bus `s01f10` #6 xa nhất, `dist_m = 46,11 m`, `r ≈ 46,112 m`. Tính r từ x, y, z trong bảng A, không suy ngược từ `dist_m` đã làm tròn.

**Kích thước và yaw:** Cyclist `s01f41` #2 phù hợp khoảng xe đạp có người ở bảng 4.3. Car `s01f41` #1 rộng 2,024 m, ngoài khoảng sedan 1,70–1,90 m; cần kiểm gương và phạm vi hộp. Xe cùng chiều phía trước `s01f10` #2 có yaw 269,9° gần 270°; xe được gán hướng ngược chiều `s02f28` #1 có yaw 90,0° gần 90°. Số yaw phù hợp quy ước vẫn cần đối chiếu đầu xe trên camera.

| Cuboid Car (frame, idx) | dist_m (m) | points_in (điểm) |
|---|---|---|
| s01f10, #2 | 5,47 | 2338 |
| s04f31, #2 | 12,71 | 1087 |
| s01f10, #3 | 23,84 | 210 |
| s01f10, #8 | 38,63 | 49 |

Trong các mẫu này, xa hơn khoảng **7,06 lần** (38,63/5,47) thì số điểm còn khoảng **2,10%** (49/2338). Xu hướng giảm phù hợp đường cong bài giảng: trung vị xe con khoảng 1968, 718, 214, 72 điểm ở các dải 0–10, 10–20, 20–30, 30–40 m; từng vật còn chịu ảnh hưởng kích thước, góc nhìn và che khuất.

Hai chỉnh sửa đã ghi nhận sau tự kiểm:

- `s01f10` #1 Car: dài từ khoảng 2,665 lên 3,112 m, rộng từ 1,393 lên 1,411 m; bật occluded. Hộp vẫn nhỏ hơn khoảng sedan tham chiếu, cần xem lại phần bị che.
- `s01f41` #3 Cyclist: cao từ khoảng 1,984 xuống 1,580 m, z tâm từ −0,591 lên −0,450 m; đáy từ khoảng −1,583 lên −1,240 m, bật occluded. Cảnh báo đáy trước đó đã hết khi đối chiếu nền cục bộ.

Ba cảnh báo đang giữ: `s05f02` #5 Car (gap −1,19 m), `s05f46` #2 Car (−0,40 m), `s05f46` #3 Car (−0,36 m). Cả ba có đáy −1,900 m và occluded. Vùng đường quanh chúng bị che hoặc lẫn nền khác cao độ nên ước lượng mặt đường của tự kiểm có thể lệch. Giữ hình học hiện tại để tiếp tục rà soát theo đường cục bộ, không nâng hộp chỉ để hết cờ; chưa khẳng định cả ba là cảnh báo sai.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

Model cần nhãn theo mét để học vị trí, kích thước và khoảng cách vật trong không gian, thay vì phụ thuộc kích thước ảnh theo pixel. Vật thưa điểm cần quy tắc riêng vì phần hình học quan sát được có thể rất nhỏ; hộp đoán thiếu cơ sở đưa sai vị trí, kích thước hoặc hướng vào dữ liệu huấn luyện.