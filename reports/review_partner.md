# Peer review — Day 3

Bài làm cá nhân, không có partner. Tài liệu này là bản tự rà soát của tác giả trên cả hai clip, dựa trên kết quả `check_mot_labels.py`, ba lượt tua, `outputs/eval_clip02_vs_warmup.json` (so với reference warm-up) và `outputs/eval_vs_gold.json` (so với gold).

| Trường | Giá trị |
| --- | --- |
| Author | Nguyễn Hữu Huy — 2A202602131 |
| Reviewer | Nguyễn Hữu Huy (tự rà soát) |
| Pair ID | N/A |
| Công cụ | CVAT, export MOT 1.1 |
| Thời điểm review | 2026-09-15, 17:10–17:30 |

## Danh sách finding

CVAT frame = MOT frame − 1. ID là ID trong bản gán của tác giả.

| # | Clip | CVAT frame | MOT frame | ID | Loại lỗi | Quan sát và rule áp dụng | Cách sửa đề xuất | Closure |
| ---: | --- | ---: | ---: | ---: | --- | --- | --- | --- |
| 1 | `clip_02` | 0–14 | 1–15 | 2, 3 | Bbox đứng im | Validator cảnh báo; ảnh cho thấy hai xe buýt dừng tại điểm đỗ cả clip. Rule: xe dừng vẫn là `vehicle` | Không sửa | not-a-defect |
| 2 | `clip_02` | 0–2 | 1–3 | 4 | Track dưới ngưỡng | Xe bị cắt ở rìa trái, chỉ lộ 15×31 px; reference không có track (3 FP). Rule: ngưỡng 20 px | Xóa track | needs-review |
| 3 | `clip_02` | 8–9 | 9–10 | 1 | Bbox treo | Phần xe ở rìa trái chỉ còn 29 và 13 px; reference kết thúc ở frame 8 (2 FP) | Bấm `outside` tại frame 9 | needs-review |
| 4 | `clip_02` | 56–59 | 57–60 | 5 | Bbox treo | Phần xe ở rìa trái giảm từ 35 xuống 8 px; reference kết thúc ở frame 56 (4 FP) | Bấm `outside` tại frame 57 | needs-review |
| 5 | `clip_02` | 15–19 | 16–20 | 6 | Bắt đầu track muộn | Xe đã lộ rõ ở góc dưới trái từ frame 16 (rộng 49–138 px); bản gán bắt đầu ở 21 (5 FN) | Kéo track lùi về frame 16 | needs-review |
| 6 | `clip_02` | 31–39 | 32–40 | 6 | Interpolation drift, bbox treo | Không có keyframe sau frame 31, bbox đứng yên tại (252, 313, 369×227) trong khi xe chạy sang góc dưới phải và rời khung ở frame 38; IoU giảm từ 0.66 xuống 0 (6 FP + 6 FN, thêm 2 FP ở frame 39–40) | Keyframe mỗi frame 31–38, bấm `outside` tại frame 39 | needs-review |
| 7 | `clip_02` | 14–16, 22–27, 32–34 | 15–17, 23–28, 33–35 | 5, 6, 7 | Bbox lỏng | Bbox hẹp hơn reference do không bao phần nhòe (ID 5 rộng 151–179 px so với 236–257 px) hoặc lệch trước xe khi nội suy (ID 6); IoU 0.55–0.64 | Thêm keyframe, bao cả phần nhòe | needs-review |
| 8 | `clip_01` | 0–14, 136–150 | 1–15, 137–151 | 2 | Bbox đứng im | Validator cảnh báo; ảnh cho thấy xe SUV đỗ cả clip. Rule: xe đỗ vẫn là `vehicle` | Không sửa | not-a-defect |
| 9 | `clip_01` | 80–99 | 81–100 | 6 | Bắt đầu track sớm | Xe chỉ lộ dải 10–26 px sau xe buýt ID 4; gold bắt đầu ở 101 (20 FP). Rule: ngưỡng 20 px / 30% thân xe | Đặt `outside` đến frame 100 | needs-review |
| 10 | `clip_01` | 105–109 | 106–110 | 7 | Bắt đầu track muộn | Xe đã lộ rõ ở rìa phải từ frame 106 (5 FN). Rule: xe vào từ rìa không bị che thì mở track ngay | Kéo track lùi về frame 106 | needs-review |
| 11 | `clip_01` | 50–52 | 51–53 | 4 | Bắt đầu track sớm | Gold bắt đầu ở 54 (3 FP). Rule: ngưỡng xác định xe | Bắt đầu track từ frame 54 | needs-review |
| 12 | `clip_01` | 148–150 | 149–151 | 4 | Bbox treo | Phần xe ở rìa trái không còn đạt ngưỡng; gold kết thúc ở 148 (3 FP) | Bấm `outside` tại frame 149 | needs-review |
| 13 | `clip_01` | 168–170 | 169–171 | 8 | Bbox treo | Bbox giữ nguyên tọa độ sau khi xe rời khung; gold kết thúc ở 168 (3 FP) | Bấm `outside` tại frame 169 | needs-review |
| 14 | `clip_01` | 4–9, 82–84 | 5–10, 83–85 | 1, 5 | Bbox lỏng | Xe nhỏ, IoU với gold 0.50–0.58. Rule: bbox < 50 px cần keyframe mỗi 2–3 frame | Thêm keyframe, chỉnh bbox ôm sát | needs-review |

## Reviewer checklist

| Hạng mục | PASS / FINDING / N/A | Frame–ID–evidence |
| --- | --- | --- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh | PASS | `clip_02`: 7 track, 233 bbox; `clip_01`: 8 track, 599 bbox; không gán người, xe máy, sạp báo |
| Một xe giữ một ID; không reuse ID cho xe khác | PASS | IDSW = 0 ở cả hai clip; không track tham chiếu nào bị tách |
| Occlusion ngắn giữ ID; crossing không đổi ID | PASS | `clip_01` ID 5, ID 6 bị xe buýt ID 4 che trong frame 81–140 vẫn giữ ID |
| Entry/exit đúng; không box treo sau khi xe rời khung | FINDING | Finding 2–6, 9–13 |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS | `clip_02` ID 1 và `clip_01` ID 4 chạm đúng x = 0 khi rời khung |
| Frame giữa hai keyframe không bị interpolation drift | FINDING | Finding 6, 7, 14 |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS | Validator 0 lỗi ở cả hai clip; `clip_02` frame 1..60, ID 1–7; `clip_01` frame 1..190, ID 1–8 |
| Mọi finding có cách sửa và closure do tác giả điền | PASS | 14/14 finding có closure |

## Self-QC attestation của reviewer

| Lượt | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence |
| --- | --- | --- |
| 1 — identity/timeline | PASS | ID liên tục, không trùng ID trong cùng frame ở cả hai clip |
| 2 — endpoint/scope | NEEDS-REVIEW | Lệch biên track: `clip_02` ID 1, 4, 5, 6; `clip_01` ID 4, 6, 7, 8 |
| 3 — geometry/interpolation | NEEDS-REVIEW | `clip_02` ID 6 bbox đứng yên frame 32–40; bbox lỏng ở `clip_02` ID 5, 7 và `clip_01` ID 1, 5 |

## Exit ticket

1. Finding quan trọng nhất: finding 9 (`clip_01` ID 6, frame 81–100), gây 20/39 FP của clip chính. Rule áp dụng: chỉ mở track khi phần nhìn thấy đạt ≥ 20 px và ≥ 30% thân xe.
2. Finding đóng not-a-defect: finding 1 và 8, vì đây là xe dừng hoặc đỗ thật, không phải lỗi quên bấm `outside`.
3. Rule cần Lab Coach làm rõ: ngưỡng định lượng cho thời điểm xe được coi là "xác định được" khi vào khung sau vật che hoặc rời khung ở rìa ảnh.
