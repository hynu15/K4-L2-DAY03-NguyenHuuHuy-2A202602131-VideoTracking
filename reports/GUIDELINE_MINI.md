# Mini annotation guideline — Ngày 3 (Tracking)

Người thực hiện: Nguyễn Hữu Huy — 2A202602131 (cá nhân)
Clip áp dụng: `clip_02` (warm-up, 60 frame) và `clip_01` (190 frame); cả hai 960×540, 12.5 fps

---

## 1. Phạm vi gán nhãn

Chỉ dùng một lớp `vehicle` cho xe bốn bánh.

| Gán | Không gán |
| --- | --- |
| Xe con, SUV, taxi, xe bán tải | Người đi bộ |
| Van, minivan | Xe đạp, xe máy |
| Xe buýt, minibus | Xe trong ảnh quảng cáo (kể cả hình xe in trên thân xe buýt), gương, bóng phản chiếu |
| Xe tải, xe đầu kéo | Vật tĩnh có hình khối giống xe: sạp báo, rào chắn, thùng rác, biển báo |

Trong `clip_01`, sạp báo bên đường (x ≈ 490–590, y ≈ 210–270, frame 16–116) không được gán. Detector nhận nhầm vật thể này thành xe, nhưng nó không thuộc phạm vi `vehicle`.

## 2. Luật ID

| Tình huống | Luật áp dụng | Lý do |
| --- | --- | --- |
| Xe bị che rồi hiện lại | Giữ nguyên ID nếu thời gian che dưới 25 frame (2 giây) | Khoảng che ngắn vẫn đủ để nối hướng đi và vị trí; ví dụ `clip_01` ID 5 và ID 6 bị xe buýt ID 4 che trong frame 81–140 vẫn giữ một ID |
| Xe bị che từ 25 frame trở lên | Bấm `outside` tại frame xe mất hẳn, mở track mới khi xe hiện lại | Không còn bằng chứng chuyển động liên tục |
| Xe rời khung rồi quay lại | Mở track mới | Không thể xác nhận là cùng một xe |
| Hai xe cắt nhau hoặc chồng lên nhau | Giữ ID theo hướng và vận tốc trước khi chồng; mỗi xe một bbox; keyframe mỗi 1–2 frame trong đoạn chồng | Đây là vị trí dễ xảy ra ID switch nhất |

## 3. Luật bbox

| Tình huống | Luật áp dụng |
| --- | --- |
| Ngưỡng tồn tại của track | Track chỉ tồn tại khi phần nhìn thấy rộng ≥ 20 px và chiếm ≥ 30% thân xe; áp dụng cho cả frame bắt đầu và frame kết thúc |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa, không suy đoán phần ngoài ảnh (`clip_01` ID 4, frame 146–148, x = 0) |
| Xe bị che một phần | Bbox ôm phần nhìn thấy, bật cờ occluded |
| Xe chạy nhanh, bị nhòe | Bbox bao cả phần thân xe bị nhòe theo hướng chuyển động |
| Xe đỗ hoặc dừng | Vẫn gán `vehicle` suốt thời gian trong khung; cảnh báo "bbox đứng im" của validator là not-a-defect |
| Mật độ keyframe | Mỗi 1–2 frame ở 5 frame đầu, 5 frame cuối, đoạn bị che và khi xe đổi kích thước nhanh; mỗi 10–20 frame khi xe đi đều hoặc đứng yên |
| Xe đi sát camera | Keyframe mỗi frame cho đến khi xe rời khung, vì vị trí và kích thước bbox thay đổi rất nhanh |
| Xe nhỏ (bbox rộng < 50 px) | Keyframe mỗi 2–3 frame, vì sai lệch vài pixel đã làm IoU giảm đáng kể |

## 4. Các ca mơ hồ

| Ca | Clip / frame / ID | Tình huống | Quyết định và lý do |
| --- | --- | --- | --- |
| 1 | `clip_02` / 1–3 / ID 4 | Xe bị cắt ở rìa trái phía trên, chỉ lộ vùng 15×31 px | Bản gán tạo track 3 frame; reference không gán. Luật chốt: không gán phần xe dưới ngưỡng 20 px vì không đủ để xác định là xe |
| 2 | `clip_02` / 16–40 / ID 6 | Xe màu đen đi sát camera từ góc dưới trái sang góc dưới phải, kích thước đổi nhanh và bị nhòe | Bản gán bắt đầu từ frame 21 (reference: 16) và thiếu keyframe sau frame 31 nên bbox đứng yên khi xe đã rời khung ở frame 38. Luật chốt: keyframe mỗi frame cho xe sát camera, bấm `outside` ngay khi xe ra khỏi rìa dưới |
| 3 | `clip_01` / 81–100 / ID 6 | Xe đi vào từ rìa phải phía sau thân xe buýt ID 4, chỉ lộ dải hẹp khoảng 10–26 px | Bản gán mở track từ frame 81; gold bắt đầu từ frame 101. Luật chốt: dải hẹp phía sau xe khác chưa đạt ngưỡng, chưa mở track |
| 4 | `clip_01` / 146–151 / ID 4 | Xe buýt khớp nối rời khung ở rìa trái, phần còn lại nhỏ dần | Bản gán kết thúc ở frame 151; gold kết thúc ở frame 148. Luật chốt: kết thúc track tại frame phần nhìn thấy không còn đạt ngưỡng, không kéo đến pixel cuối cùng |
| 5 | `clip_02` / 1–60 / ID 2, 3 và `clip_01` / 1–190 / ID 2 | Xe buýt dừng tại điểm đỗ (`clip_02`) và xe SUV đỗ bên đường (`clip_01`); validator cảnh báo bbox đứng im | Giữ track liên tục, không bấm `outside`. Xe thật đứng yên nên cảnh báo là not-a-defect |
| 6 | `clip_01` / 16–116 / không gán | Sạp báo có khối chữ nhật sáng, kích thước tương tự xe; model tạo track (ReID ID 7, ByteTrack ID 10) | Không gán vì không phải xe bốn bánh; gold cũng không có track tại vị trí này |

## 5. Điều chỉnh sau warm-up và sau khi đối chiếu gold

- Xe rời khung ở rìa ảnh: bấm `outside` tại frame đầu tiên phần nhìn thấy dưới 20 px. Lỗi này xuất hiện ở warm-up (`clip_02` ID 1 frame 9–10, ID 5 frame 57–60) và lặp lại ở `clip_01` (ID 4 frame 149–151, ID 8 frame 169–171), nên được đưa vào checklist bắt buộc.
- Xe vào khung sau vật che: chưa mở track khi chỉ thấy dải nhỏ phía sau xe khác (`clip_01` ID 6, frame 81–100).
- Xe vào từ rìa ảnh không bị che: mở track ngay khi nhận ra là xe, không chờ xe vào hẳn khung (`clip_02` ID 6 từ frame 16, `clip_01` ID 7 từ frame 106).
- Xe sát camera hoặc chạy nhanh: keyframe mỗi frame và bbox bao cả phần nhòe (`clip_02` ID 6 frame 22–38; ID 5 frame 15–17; ID 7 frame 33–35).
- Xe nhỏ ở xa: tăng mật độ keyframe để tránh bbox lỏng (`clip_01` ID 5, frame 83–85, IoU 0.50–0.58).
- Rà soát: bài làm cá nhân, kiểm chéo được thay bằng tự rà soát tại `reports/review_partner.md`.
