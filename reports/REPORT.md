# Báo cáo Ngày 3 — Tracking Annotation

Họ tên / nhóm: `Nguyễn Hữu Huy — 2A202602131`
Ngày: `2026-09-15`

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT (export MOT 1.1) |
| Thời gian gán `clip_02` (warm-up) | `[bạn điền]` phút |
| Thời gian gán `clip_01` | `[bạn điền]` phút |
| Số track đã vẽ trong `clip_01` | `8` (599 row, 190 frame) |
| Số keyframe trung bình mỗi track | `[bạn điền]` |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. **Xe ID 6 đi vào từ rìa phải phía sau xe buýt (frame 81–100).** Xe gần như bị
   thân xe buýt ID 4 che hết, chỉ lộ một dải hẹp (bbox rộng ~10 px ở frame 81).
   Tôi mở track ngay từ frame 81 và ôm phần nhìn thấy. Gold chỉ bắt đầu ở frame
   101, nên 20 frame này thành FP (xem mục 3).
2. **Xe buýt khớp nối ID 4 rất dài, rời khung ở rìa trái (frame 148–151).** Bbox
   bị cắt ở `x = 0`, tôi cho chạm đúng rìa ảnh. Khó ở chỗ chọn frame bấm `outside`:
   tôi bấm ở frame 152, gold kết thúc ở frame 148.
3. **Xe nhỏ ở xa ID 5 chạy sát/qua sau xe buýt (frame 81–140).** Bbox chỉ khoảng
   25–50 px và bị xe buýt che một phần, dễ gây ID switch. Tôi giữ nguyên ID 5 suốt
   đoạn này, không tách track (IDSW = 0 so với gold).

## 2. Tự kiểm và kiểm chéo



## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `[chưa có — thư mục evidence/pre-gold/clip_01/ hiện chỉ có .gitkeep]` |
| Thời điểm khóa | `[chưa có]` |
| Số row / frame / track trước khi mở reference | `[chưa có]` |

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | `[chưa có eval_pre_gold.json]` | | | | | | | | | |
| Sau rework | 0.8015 | 0.7854 | 0.8191 | 0.8724 | 0.9556 | 0.9092 | 0.8592 | 39 | 13 | 0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có**. HOTA 0.80,
IDF1 0.96, MOTA 0.91, LocA 0.87 đều đạt mức **Xuất sắc** theo `RUBRIC.md`.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi | Frame | ID | Đã sửa thế nào |
| --- | --- | --- | --- |
| `[bạn điền]` | | | |
| | | | |
| | | | |

Lỗi **còn lại** trong `outputs/eval_vs_gold.json` (bản cuối):

| Loại lỗi | Frame | ID (của tôi) | Quan sát |
| --- | --- | --- | --- |
| Bắt đầu track quá sớm | 81–100 | 6 | xe còn bị xe buýt che, gold bắt đầu ở 101 → 20 FP |
| Bắt đầu quá sớm / bbox treo | 51–53, 149–151 | 4 | gold là 54–148; lệch 3 frame ở mỗi đầu |
| Bbox treo | 169–171 | 8 | bbox đứng im cùng tọa độ sau khi xe rời khung (gold kết thúc 168) |
| Bắt đầu muộn | 106–110 | 7 | tôi bắt đầu ở 111, gold ở 106 → 5 FN |
| Bbox lỏng | 5, 9, 10 | 1 (gold 2) | IoU 0.52–0.58; track ngắn 11 frame chỉ khớp 8 frame |
| Bbox lỏng | 83–85, 138 | 5 | IoU 0.50–0.58; ở frame 81 bbox của tôi rộng 24 px, gold rộng 49 px |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control) và `configs/trackers/botsort-reid.yaml` (treatment: `with_reid: true`, `gmc_method: none`, `track_buffer: 30`, `appearance_thresh: 0.80`) |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / `[2, 5, 7]` (car, bus, truck) |
| device | `0` (GPU Colab), `persist: true`, 190 frame |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| bạn vs gold | 0.8015 | 0.7854 | 0.8191 | 0.8724 | 0.9556 | 0.9092 | 0.8592 | 39 | 13 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs bạn | 0.7636 | 0.7048 | 0.8284 | 0.8964 | 0.8731 | 0.7396 | 0.8906 | 97 | 58 | 1 |

ByteTrack **không qua** learning gate (MOTA 0.7487 < 0.75, thiếu 0.0013).
BoT-SORT + ReID **qua** cả ba điều kiện.

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của tôi (**0.9092**) **thấp hơn** IDF1 (**0.9556**). Nhãn của tôi không có
lỗi ID: IDSW = 0, không track gold nào bị tách, 8 track của tôi khớp 1–1 với 8
track gold. Vì vậy IDF1 = 2·560 / (2·560 + 39 + 13) ≈ 0.956 chỉ bị kéo xuống bởi
các box thừa/thiếu. MOTA = 1 − (FP + FN + IDSW) / GT = 1 − (39 + 13 + 0) / 573 ≈
0.909, và bị trừ nặng hơn vì mọi FP/FN đều tính trực tiếp trên số box GT. Phần lớn
FP/FN của tôi là **lỗi biên track** (bắt đầu/kết thúc lệch frame: ID 6 sớm 20
frame, ID 4 và ID 8 treo 3 frame, ID 7 muộn 5 frame), không phải lỗi identity.

Nếu gặp trường hợp ngược lại (**MOTA cao mà IDF1 thấp**) thì đó là dấu hiệu lỗi
ID: detection gần như đủ, nhưng xe bị gán sai hoặc bị tách ID. MOTA không phạt nặng
lỗi ID vì mỗi ID switch chỉ bị đếm **một lần** tại frame xảy ra switch. Ví dụ
ByteTrack với gold track 4: ID 14 (frame 56–57) đổi sang ID 15 (frame 59–148),
MOTA chỉ bị trừ 1 IDSW. Nếu switch rơi vào giữa track, IDF1/AssA sẽ phạt cả nửa
quãng đời track vì chỉ một pred ID được khớp với cả track gold.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

| | IDF1 | AssA | IDSW | IDFN | Track bị tách |
| --- | ---: | ---: | ---: | ---: | --- |
| ByteTrack | 0.8746 | 0.7761 | 2 | 57 | gold 4, 5, 7 |
| BoT-SORT + ReID | 0.9001 | 0.8204 | 2 | 28 | gold 5, 6, 7 |

Treatment tốt hơn ở IDF1 (+0.026) và AssA (+0.044), nhưng **IDSW không đổi (2 = 2)**.
Theo evidence từng frame, treatment **tốt hơn một chút**. Phần lớn mức tăng đến từ
việc **giữ track liên tục qua khoảng mất detection**, chứ không phải bớt số lần đổi ID:

- **Gold track 4 (xe buýt vào từ rìa phải, frame 54–60)**: ByteTrack có ID 14 ở
  frame 56–57, không có box ở frame 58, rồi mở ID 15 từ frame 59 → 1 IDSW. ReID giữ
  **một ID 9 liên tục từ frame 55 đến 149**, có cả box ở frame 58 (conf 0.30).
  → **Treatment tốt hơn.**
- **Gold track 5 (xe nhỏ ở xa, frame 85–94)**: cả hai đều switch. ByteTrack có ID 23
  ở frame 85, mất box ở frame **86–93**, rồi mở ID 32 ở frame 94. ReID có ID 17 ở
  frame 85, chỉ mất frame 86, mở ID 18 từ frame 87. ID vẫn đổi, nhưng ReID mất ít
  frame hơn nên IDFN giảm từ 57 xuống 28. → **Không đổi về IDSW, tốt hơn về coverage.**
- **Gold track 6 (xe ra sau xe buýt, frame 104–110)**: ReID tạo ID 24 ở frame 104,
  rồi ID 31 từ frame 110 → switch mới mà ByteTrack không có (ByteTrack chỉ có một
  ID 47 từ frame 110). → **Treatment tệ hơn** ở đây.

Đây là **system comparison, không phải causal ablation của ReID**. BoT-SORT và
ByteTrack là hai implementation khác nhau: ngưỡng `track_high/low/new_track_thresh`,
cách giữ track chưa xác nhận và cách fuse score đều khác. Ví dụ box ở frame 58
(conf 0.30) xuất hiện trong output BoT-SORT nhưng không có trong ByteTrack dù
detector input giống nhau. Khác biệt đó đến từ logic tracker, không nhất thiết từ
appearance embedding. Muốn cô lập tác động của ReID thì phải chạy BoT-SORT với
`with_reid: false` rồi so với `with_reid: true`.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

| | DetA | LocA | FP | FN | PRED boxes |
| --- | ---: | ---: | ---: | ---: | ---: |
| ByteTrack | 0.6487 | 0.8463 | 88 | 54 | 607 |
| BoT-SORT + ReID | 0.7110 | 0.8721 | 91 | 26 | 638 |

Với treatment, DetA tăng 0.062, **FN giảm hơn một nửa (54 → 26)** và FP gần như
không đổi (88 → 91). Vậy treatment xuất ra nhiều box khớp hơn: nó giữ thêm được
box conf thấp trên track đang sống, như frame 58 của gold track 4 và các frame
86–93 của gold track 5.

Lỗi còn lại chủ yếu là **detector (FP)**, association chỉ đóng góp phần nhỏ:

- **~77/91 FP của ReID nằm trong 5 "ghost track"** không khớp track gold nào. Lớn
  nhất là **ID 7 (ByteTrack: ID 10), frame 16–116, 43 box** đứng yên ở
  `(491, 211, 100×57)`. Xem ảnh frame 60 thì đó là **sạp báo/kiosk bên đường**,
  không phải xe. YOLO zero-shot nhận nhầm thành car/truck, và cả hai tracker đều
  bám theo. ID 27 (frame 106–121) và ID 38 (frame 158–178) là vật nhỏ ~20×20 px
  sát rìa trái `x ≈ 0`, cũng không phải xe bốn bánh trong gold.
- Phần association chỉ gồm 2 IDSW và 3 track bị tách, trong đó mảnh ngắn chỉ dài
  **1 frame** (ID 17, 24, 39). Các lỗi này làm AssA giảm nhẹ nhưng không phải nguồn
  FP chính.

→ Muốn tăng tiếp thì phải xử lý **detector** (lọc vùng kiosk cố định, tăng `conf`,
fine-tune trên camera này). Đổi tracker không giúp được, vì cả ByteTrack lẫn
BoT-SORT đều giữ nguyên ghost track sạp báo.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

**Frame 85–87, xe nhỏ ở xa (ID 5 của tôi = gold track 5).** Tôi giữ **một ID 5
liên tục từ frame 81 đến 140**. ReID gán ID 17 ở frame 85 (conf 0.53), không có
box ở frame 86, rồi mở **ID 18** từ frame 87 (conf 0.40) → 1 ID switch và gold
track 5 bị tách thành 2 track (`fragmented_gt_tracks` trong `eval_reid_vs_me.json`).
Xe chỉ khoảng 40×30 px và đang chạy sát xe buýt dài ID 4, nên detector mất 1 frame.
Với box nhỏ như vậy, appearance embedding không đủ phân biệt để nối lại vào ID 17.
Người gán nhìn được chuyển động liên tục qua nhiều frame nên biết đó vẫn là cùng
một xe.

Chỗ phụ: **frame 16–116, ghost ID 7 của ReID** là sạp báo bên đường. Tôi không gán
vì nó không phải xe bốn bánh, đúng với gold.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

**Frame 108–110, xe ID 7 của tôi (gold track 7) đi vào từ rìa phải.** Trong
`eval_reid_vs_me.json`, ReID ID 29 bị đánh dấu *"đã có bbox trước khi track tham
chiếu 7 xuất hiện"* ở frame 108–110. Xem lại ảnh frame 110 thì chiếc xe tải/van
đã lộ rõ ở rìa phải `x ≈ 922`. Gold bắt đầu track 7 từ **frame 106**, còn tôi từ
**frame 111**. Lần này **model đúng, tôi bắt đầu track muộn 5 frame**, đúng bằng
5 FN của ID 7 trong `eval_vs_gold.json`. Cần sửa: tua lùi từng frame ở rìa khung để
bắt đầu track từ frame đầu tiên xe xác định được.

Ngược lại, có một chỗ evidence cho thấy **model đúng khi không có box**: ID 6 của
tôi ở frame 81–100 (xe sau xe buýt). ReID không có box nào cho xe này trước frame
104, và gold bắt đầu ở 101. Điều này xác nhận 20 frame đầu track ID 6 của tôi là
FP: xe bị che gần hết nên chưa "xác định được là xe bốn bánh".

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

**Sửa trong `GUIDELINE_MINI.md`:**

- **Luật bắt đầu track khi xe vào khung sau vật che:** chỉ mở track khi phần nhìn
  thấy đủ để xác định là xe bốn bánh, ví dụ **≥ 30% thân xe hoặc bbox rộng ≥ 20 px**.
  Không mở track cho dải vài pixel sau xe buýt (lỗi ID 6, frame 81–100).
- **Luật bắt đầu track ở rìa khung:** xe đi vào từ rìa ảnh và đã nhận ra là xe thì
  mở track ngay, không chờ xe vào hẳn (lỗi ID 7 muộn 5 frame).
- **Luật `outside`:** bấm `outside` ở frame **đầu tiên** xe không còn phần nào nhìn
  thấy/xác định được, không phải frame sau đó. Bbox mà tọa độ không đổi qua ≥ 2
  frame ở rìa ảnh coi là bbox treo (lỗi ID 8 frame 169–171, ID 4 frame 149–151).
- **Ghi rõ vật tĩnh dễ nhầm:** sạp báo/kiosk, rào chắn, biển quảng cáo không gán.
- **Keyframe:** bắt buộc keyframe dày (mỗi 1–2 frame) ở 5 frame đầu và 5 frame cuối
  của mỗi track, và khi xe nhỏ đi sát xe lớn.

**Đổi trong quy trình:**

- Thêm một lượt tự kiểm riêng cho **biên track**: với mỗi track, tua từng frame
  quanh frame đầu/cuối, vì toàn bộ FP/FN còn lại của tôi đều nằm ở đây.
- Khóa pre-gold bằng `tools/lock_pre_gold.py` **ngay** khi xong vòng tự gán, và lưu
  `eval_pre_gold.json`, để có bằng chứng trước/sau rework.
- Sau khi khóa, dùng output ReID làm **danh sách chỗ đáng ngờ** (ghost track, box
  trước/sau track của tôi) thay vì tin model: model chỉ ra đúng ID 7 bắt đầu muộn,
  nhưng cũng tạo 43 FP ở sạp báo.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [ ] `annotations/clip_02/gt.txt`
- [ ] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json`
- [ ] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [ ] `reports/review_partner.md`
- [x] `reports/REPORT.md` (file này)
