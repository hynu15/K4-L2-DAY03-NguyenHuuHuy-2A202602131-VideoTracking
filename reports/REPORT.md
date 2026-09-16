# Báo cáo Ngày 3 — Tracking Annotation

Họ tên: Nguyễn Hữu Huy — 2A202602131 (cá nhân)
Ngày: 2026-09-15

---

## 1. Quá trình gán nhãn

| Mục | Giá trị |
| --- | --- |
| Công cụ | CVAT, export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | 30 phút (14:15–14:45) |
| Thời gian gán `clip_01` | Khoảng 60 phút (14:45–15:46) |
| Số track đã vẽ trong `clip_01` | 8 track, 599 bbox, 190 frame |
| Số keyframe trung bình mỗi track | Khoảng 8.4 (2–13 keyframe/track, nhiều nhất ở xe buýt ID 4) |

Tiến trình: warm-up `clip_02` (14:15–14:45), gán nhãn `clip_01` (14:45–15:46), kiểm tra định dạng và đối chiếu gold (15:46–16:40), chạy notebook ByteTrack và BoT-SORT + ReID trên Colab (16:40–17:10), hoàn thiện guideline, tự rà soát và báo cáo (17:10–17:30).

Kết quả warm-up `clip_02`: 7 track, 233 bbox, validator 0 lỗi và 3 cảnh báo. Chấm với reference warm-up (`outputs/eval_clip02_vs_warmup.json`):

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| `clip_02` vs reference | 0.7834 | 0.7325 | 0.8519 | 0.8556 | 0.9391 | 0.8767 | 0.8363 | 17 | 11 | 0 |

Warm-up qua cổng, không có lỗi identity. Toàn bộ FP/FN nằm ở biên track và hình học bbox: track dưới ngưỡng (ID 4, 3 FP), bbox treo khi xe rời rìa trái (ID 1 và ID 5, 6 FP), bắt đầu muộn và thiếu keyframe cho xe đi sát camera (ID 6, 8 FP và 11 FN). Chi tiết ở `reports/review_partner.md`, finding 1–7.

Ba tình huống khó nhất:

1. ID 6, frame 81–100: xe đi vào từ rìa phải phía sau xe buýt ID 4, chỉ lộ dải hẹp khoảng 10 px. Track được mở từ frame 81 và ôm phần nhìn thấy; gold bắt đầu từ frame 101.
2. ID 4, frame 146–151: xe buýt khớp nối rời khung ở rìa trái. Bbox được cắt tại x = 0; điểm khó là chọn frame bấm `outside` (bản gán kết thúc ở 151, gold ở 148).
3. ID 5, frame 81–140: xe nhỏ (25–50 px) chạy sát và bị xe buýt che một phần. ID 5 được giữ liên tục, không phát sinh ID switch so với gold.

## 2. Tự kiểm và kiểm chéo

- Lượt 1 (ID): `clip_01` có 8 ID liên tục, không trùng ID trong cùng frame. ID 5 và ID 6 đi cạnh xe buýt ID 4 (frame 81–156) vẫn giữ nguyên ID. Kết quả đối chiếu: IDSW = 0 ở cả hai clip, không track nào bị tách.
- Lượt 2 (frame đầu/cuối): `check_mot_labels.py` đạt 0 lỗi ở cả hai clip. Cảnh báo bbox đứng im tại `clip_01` ID 2 và `clip_02` ID 2, 3 là xe đỗ/xe buýt dừng, kết luận not-a-defect. Cảnh báo track 3 frame tại `clip_02` ID 4 là lỗi thật (xe dưới ngưỡng 20 px). Lỗi biên track của `clip_01` (ID 4, 6, 7, 8) được xác định sau khi đối chiếu gold.
- Lượt 3 (frame giữa): `clip_02` ID 6 bbox đứng yên từ frame 32 do thiếu keyframe khi xe đi sát camera. Ở `clip_01`, bbox ID 4 tại rìa trái chạm đúng x = 0; bbox xe nhỏ ID 5 (frame 83–85) và ID 1 (frame 5–10) hơi lỏng, IoU với gold 0.50–0.58.

Kiểm chéo: bài làm cá nhân, không có partner. Thay bằng tự rà soát tại `reports/review_partner.md` gồm 14 finding trên hai clip (2 not-a-defect, 12 needs-review).
Số lỗi tìm được trong bản của partner: N/A. Số lỗi partner tìm được trong bản của tôi: N/A.

Ca có quyết định khác biệt rõ nhất là giữa bản gán và gold tại ID 6, frame 81–100: bản gán coi xe đã xuất hiện, gold thì chưa. Luật còn thiếu là ngưỡng định lượng cho việc "xác định được là xe" khi xe vào, ra khung hoặc bị che. Ngưỡng ≥ 20 px và ≥ 30% thân xe đã được bổ sung vào `GUIDELINE_MINI.md` (mục 3 và 5).

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence | Giá trị |
| --- | --- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | Không có manifest |
| Thời điểm khóa | Không chạy `lock_pre_gold.py`; bằng chứng thay thế là commit `00f43a5` (15:46), trước thời điểm nhận gold (16:42) |
| Số row / frame / track trước khi mở reference | 599 / 190 / 8 |

Snapshot không được tạo bù sau khi mở gold, vì khi đó nó không còn giá trị là bằng chứng độc lập. Theo lịch sử git, `annotations/clip_01/gt.txt` không thay đổi từ commit `00f43a5`, do đó bản chấm dưới đây chính là bản pre-gold.

| | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Bản pre-gold | 0.8015 | 0.7854 | 0.8191 | 0.8724 | 0.9556 | 0.9092 | 0.8592 | 39 | 13 | 0 |
| Sau rework | Không rework, giữ nguyên bản pre-gold | | | | | | | | | |

Qua cổng (IDF1 ≥ 0.80, MOTA ≥ 0.75, MOTP ≥ 0.70): có. HOTA 0.80, IDF1 0.96, MOTA 0.91 và LocA 0.87 đều đạt mức Xuất sắc theo `RUBRIC.md`.

Do bản pre-gold đã đạt mức Xuất sắc, bản nộp giữ nguyên nhãn gốc. Các lỗi còn lại trong `outputs/eval_vs_gold.json` được ghi nhận và chuyển thành luật trong `GUIDELINE_MINI.md`:

| Loại lỗi | Frame | ID | Quan sát và hướng xử lý |
| --- | --- | --- | --- |
| Bắt đầu track sớm | 81–100 | 6 | Xe còn bị xe buýt che, gold bắt đầu ở 101 (20 FP); đặt `outside` đến frame 100 |
| Biên track lệch | 51–53, 149–151 | 4 | Gold là 54–148 (6 FP); rút ngắn mỗi đầu 3 frame |
| Bbox treo | 169–171 | 8 | Xe đã rời khung, gold kết thúc ở 168 (3 FP); bấm `outside` tại frame 169 |
| Bắt đầu track muộn | 106–110 | 7 | Gold bắt đầu ở 106 (5 FN); kéo track lùi về frame 106 |
| Bbox lỏng | 5, 9, 10 | 1 (gold 2) | IoU 0.52–0.58; thêm keyframe |
| Bbox lỏng | 83–85, 138 | 5 | IoU 0.50–0.58; tại frame 81 bbox rộng 24 px so với 49 px của gold |

## 4. Kết quả model: ByteTrack control vs ReID treatment

| Mục | Giá trị |
| --- | --- |
| Python / ultralytics / torch / lap | 3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13 |
| weights / hai tracker | `yolo26n.pt` / `bytetrack.yaml` (control) và `configs/trackers/botsort-reid.yaml` (treatment: `with_reid: true`, `gmc_method: none`, `track_buffer: 30`, `appearance_thresh: 0.80`) |
| conf / IoU / imgsz / classes | 0.25 / 0.7 / 960 / [2, 5, 7] (car, bus, truck) |
| device | GPU Colab (`0`), `persist: true`, 190 frame |

| So sánh | HOTA | DetA | AssA | LocA | IDF1 | MOTA | MOTP | FP | FN | IDSW |
| --- | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: | ---: |
| Nhãn của tôi vs gold | 0.8015 | 0.7854 | 0.8191 | 0.8724 | 0.9556 | 0.9092 | 0.8592 | 39 | 13 | 0 |
| ByteTrack control vs gold | 0.7085 | 0.6487 | 0.7761 | 0.8463 | 0.8746 | 0.7487 | 0.8226 | 88 | 54 | 2 |
| BoT-SORT + ReID vs gold | 0.7635 | 0.7110 | 0.8204 | 0.8721 | 0.9001 | 0.7923 | 0.8595 | 91 | 26 | 2 |
| ReID vs nhãn của tôi | 0.7636 | 0.7048 | 0.8284 | 0.8964 | 0.8731 | 0.7396 | 0.8906 | 97 | 58 | 1 |

ByteTrack không qua learning gate (MOTA 0.7487 < 0.75). BoT-SORT + ReID qua cả ba điều kiện.

## 5. Phân tích — năm câu hỏi

### 5.1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?

MOTA (0.9092) thấp hơn IDF1 (0.9556). Nhãn không có lỗi ID: IDSW = 0 và 8 track khớp 1–1 với gold, nên IDF1 = 2·560 / (2·560 + 39 + 13) ≈ 0.956 chỉ bị giảm bởi box thừa/thiếu. MOTA = 1 − (39 + 13 + 0) / 573 ≈ 0.909 giảm nhiều hơn vì mọi FP/FN được tính trực tiếp trên tổng số box GT. Phần lớn FP/FN là lỗi biên track (ID 6, 4, 8, 7), không phải lỗi identity.

Trường hợp MOTA cao nhưng IDF1 thấp cho thấy detection gần đủ nhưng association sai: xe bị gán nhầm hoặc bị tách ID. MOTA không phạt nặng lỗi ID vì mỗi ID switch chỉ bị trừ một lần tại frame xảy ra. Ví dụ ByteTrack với gold track 4: ID 14 (frame 56–57) chuyển sang ID 15 (frame 59–148), MOTA chỉ bị trừ 1 IDSW. Ngược lại, IDF1 và AssA phạt toàn bộ phần track không thuộc pred ID được khớp.

### 5.2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW?

| | IDF1 | AssA | IDSW | IDFN | Track gold bị tách |
| --- | ---: | ---: | ---: | ---: | --- |
| ByteTrack | 0.8746 | 0.7761 | 2 | 57 | 4, 5, 7 |
| BoT-SORT + ReID | 0.9001 | 0.8204 | 2 | 28 | 5, 6, 7 |

Treatment tăng IDF1 (+0.026) và AssA (+0.044), IDSW không đổi. Mức cải thiện đến chủ yếu từ việc duy trì track qua các frame mất detection, không phải từ việc giảm số lần đổi ID:

- Gold track 4, frame 54–60: ByteTrack có ID 14 ở frame 56–57, mất box ở frame 58, mở ID 15 từ frame 59. ReID giữ một ID 9 liên tục từ frame 55 đến 149, có box ở frame 58 (conf 0.30). Treatment tốt hơn.
- Gold track 5, frame 85–94: cả hai đều switch. ByteTrack mất box ở frame 86–93 rồi mở ID 32 ở frame 94; ReID chỉ mất frame 86 và mở ID 18 từ frame 87. IDSW không đổi, nhưng coverage tốt hơn (IDFN giảm từ 57 xuống 28).
- Gold track 6, frame 104–110: ReID tạo ID 24 ở frame 104, sau đó ID 31 từ frame 110, phát sinh một switch mà ByteTrack không có (ByteTrack chỉ có ID 47 từ frame 110). Treatment kém hơn.

Đây là so sánh hệ thống, không phải causal ablation của ReID. ByteTrack và BoT-SORT khác nhau ở ngưỡng `track_high/low/new_track_thresh`, cách giữ track chưa xác nhận và cách kết hợp score. Ví dụ box conf 0.30 ở frame 58 chỉ có trong output BoT-SORT dù detector input giống nhau. Để cô lập tác động của ReID cần so sánh BoT-SORT với `with_reid: false` và `with_reid: true`.

### 5.3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?

| | DetA | LocA | FP | FN | PRED boxes |
| --- | ---: | ---: | ---: | ---: | ---: |
| ByteTrack | 0.6487 | 0.8463 | 88 | 54 | 607 |
| BoT-SORT + ReID | 0.7110 | 0.8721 | 91 | 26 | 638 |

Với treatment, DetA tăng 0.062, FN giảm hơn một nửa (54 → 26), FP gần như không đổi (88 → 91). Treatment giữ được thêm box conf thấp trên track đang tồn tại, như frame 58 của gold track 4 và frame 87–93 của gold track 5.

Lỗi còn lại chủ yếu thuộc detector:

- Khoảng 77/91 FP của ReID nằm trong 5 ghost track. Lớn nhất là ID 7 (ByteTrack ID 10), frame 16–116, 43 box cố định tại (491, 211, 100×57); đối chiếu ảnh frame 60 cho thấy đây là sạp báo bên đường. Cả hai tracker đều bám theo phát hiện sai này. ID 27 (frame 106–121) và ID 38 (frame 158–178) là vật nhỏ khoảng 20×20 px sát rìa trái, không thuộc gold.
- Phần association chỉ gồm 2 IDSW và 3 track bị tách, trong đó mảnh ngắn chỉ dài 1 frame (ID 17, 24, 39).

Để cải thiện tiếp cần xử lý ở detector (lọc vùng vật tĩnh, tăng `conf`, fine-tune trên camera này); thay đổi tracker không loại bỏ được ghost track sạp báo.

### 5.4. Một chỗ bạn đúng và ReID sai

Frame 85–87, ID 5 (gold track 5). Nhãn giữ một ID 5 liên tục từ frame 81 đến 140. ReID gán ID 17 ở frame 85 (conf 0.53), mất box ở frame 86, rồi mở ID 18 từ frame 87 (conf 0.40), khiến track bị tách thành hai (`fragmented_gt_tracks` trong `eval_reid_vs_me.json`). Xe chỉ khoảng 40×30 px và chạy sát xe buýt ID 4 nên detector mất một frame; embedding của box nhỏ không đủ phân biệt để nối lại ID 17. Người gán dựa vào chuyển động liên tục qua nhiều frame nên xác định được đây vẫn là cùng một xe.

Ngoài ra, ghost track ID 7 của ReID (frame 16–116) là sạp báo; nhãn không gán vật thể này, phù hợp với gold.

### 5.5. Một chỗ ReID làm bạn xem lại annotation

Frame 106–110, ID 7 (gold track 7). Trong `eval_reid_vs_me.json`, ReID ID 29 bị đánh dấu "đã có bbox trước khi track tham chiếu 7 xuất hiện" tại frame 108–110. Xem lại ảnh frame 110, xe đã lộ rõ ở rìa phải (x ≈ 922). Gold bắt đầu track từ frame 106, nhãn bắt đầu từ frame 111. Model đúng trong trường hợp này: track bắt đầu muộn 5 frame, tương ứng 5 FN của ID 7 trong `eval_vs_gold.json`.

Ngược lại, tại ID 6 frame 81–100, ReID không có box nào trước frame 104 và gold bắt đầu ở 101. Điều này xác nhận 20 frame đầu của ID 6 là FP do xe bị che gần hết.

## 6. Nếu phải gán thêm 10 clip nữa

Điều chỉnh trong `GUIDELINE_MINI.md`:

- Mở track khi phần nhìn thấy đạt ≥ 20 px và ≥ 30% thân xe; không mở track cho dải hẹp phía sau xe khác (ID 6).
- Xe vào từ rìa ảnh không bị che: mở track ngay khi nhận ra là xe (`clip_02` ID 6, `clip_01` ID 7).
- Bấm `outside` tại frame đầu tiên phần nhìn thấy không còn đạt ngưỡng; bbox giữ nguyên tọa độ từ 2 frame trở lên ở rìa ảnh là bbox treo (`clip_02` ID 1, 5; `clip_01` ID 4, 8).
- Xe đi sát camera hoặc chạy nhanh: keyframe mỗi frame và bbox bao cả phần nhòe (`clip_02` ID 5, 6, 7).
- Liệt kê vật tĩnh dễ nhầm: sạp báo, rào chắn, biển quảng cáo, hình xe in trên thân xe buýt.
- Keyframe mỗi 1–2 frame ở 5 frame đầu, 5 frame cuối mỗi track và khi xe nhỏ đi sát xe lớn.

Điều chỉnh trong quy trình:

- Chuyển lỗi của warm-up thành checklist trước khi gán clip chính. Lỗi bbox treo ở rìa ảnh đã xuất hiện ở `clip_02` (ID 1, 5) và lặp lại ở `clip_01` (ID 4, 8), cho thấy chấm warm-up thôi chưa đủ nếu không cập nhật luật ngay.
- Bổ sung một lượt tự kiểm riêng cho biên track, tua từng frame quanh frame đầu và cuối, vì phần lớn FP/FN của cả hai clip nằm ở đây.
- Chạy `tools/lock_pre_gold.py` ngay sau vòng tự gán để có bằng chứng độc lập và so sánh trước/sau rework.
- Sau khi khóa pre-gold, dùng output ReID làm danh sách vị trí cần kiểm tra (ghost track, box trước/sau track), không coi model là đáp án.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt` (validator: 0 lỗi, 2 cảnh báo not-a-defect)
- [x] `annotations/clip_02/gt.txt` (validator: 0 lỗi, 3 cảnh báo) và `outputs/eval_clip02_vs_warmup.json`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json` — không khóa pre-gold (xem mục 3)
- [x] `GUIDELINE_MINI.md` (bản sao tại `reports/GUIDELINE_MINI.md`)
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `reports/review_partner.md` (tự rà soát)
- [x] `reports/REPORT.md`
