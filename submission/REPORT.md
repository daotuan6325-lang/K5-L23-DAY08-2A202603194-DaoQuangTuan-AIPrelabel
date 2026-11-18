# Báo cáo Day 08 — 2A202603194 - Đào Quang Tuấn

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person = 0.25 (sweep: recall 20/22 = 91%, precision 20/29 = 69%); các lớp khác mặc định 0.3 (motorbike, car: không ngưỡng nào ≤ 0.5 đạt recall 90%; bicycle, truck, bus: < 5 vật trong gold) | <<BẠN VIẾT: dẫn r02/r03>> |
| three_wheeler | YOLOE-26s, prompt: "tuk-tuk", "cyclo", "cargo tricycle" | 0.15 (sweep: recall 7/7 = 100%, precision 7/9 = 78%; ở 0.5 recall chỉ 3/7 = 43%) | <<BẠN VIẾT: dẫn r04/r07; vì sao chọn 0.15 thay vì 0.5–0.6 như dự định ở r04>> |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 633 box; 11 box `car` và 6 box `truck` của YOLO26 bị loại vì chồng lên box `three_wheeler` của YOLOE.
- **Phiên bản SAM bạn chọn**: SAM 2.1 base (`sam2.1_b.pt`). Lý do: <<BẠN VIẾT: chất lượng mask, tốc độ (3.37 s/ảnh trên GTX 1650 ở r05), RAM>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? <<BẠN VIẾT: dẫn img_013 car 0.42, img_004; car có 7 FP ở mọi ngưỡng ≤ 0.25 trong threshold_sweep; sửa bằng đổi model (YOLOE) khi merge>>
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): <<BẠN VIẾT: ví dụ img_002 truck 0.73 (bán tải); img_032 xích lô thật chỉ 0.48 và 0.27>>
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: <<BẠN VIẾT: 4 ảnh, three_wheeler 7 vật chỉ trong 2 ảnh, chọn và chấm cùng dữ liệu>>

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 497 | 497/544 = 91.4% | 544 box máy |
| edited | 11 | 11/544 = 2.0% | 544 box máy |
| deleted | 24 | 24/544 = 4.4% | 544 box máy |
| added | 18 | 18/526 = 3.4% | 526 đối tượng sau duyệt |

(Đã loại 23 đối tượng quanh vùng bẫy khỏi bảng. Trong deleted + added có 5 ca đổi lớp; máy sót thật ≈ 18 − 5 = 13.)

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? three_wheeler: accepted 13, edited 7, deleted 17, added 2. <<BẠN VIẾT: vì sao, dẫn ảnh>>
- So với gold: precision / recall của **máy** 44/64 = 68.8% (KTC 95%: 57–79%) / 44/54 = 81.5% (KTC 95%: 69–90%) (mẫu số: 54 đối tượng gold)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 5/12 = 41.7% (KTC 95%: 19–68%); theo loại: wrong_class 3/3, extra 1/3, missing 1/3, loose 0/3. Lọt: img_026 (missing), img_024 (missing), img_029 (extra), img_021 (extra), img_010 (loose), img_028 (loose), img_004 (loose). Vì sao (bạn đã bỏ bước nào của checklist)? <<BẠN VIẾT>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<BẠN VIẾT: accepted 91.4% vs bắt bẫy 41.7%>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 23 phút / 4 ảnh = 5.75 phút/ảnh (33 phút = 8.25 phút/ảnh nếu tính 10 phút sửa gold) vs duyệt 30 phút / 24 ảnh = 1.25 phút/ảnh (không tính 6 phút nghỉ). Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<BẠN VIẾT>>
- Kết luận: <<BẠN VIẾT: có / không / chỉ cho lớp nào>>, dựa trên <<BẠN VIẾT: số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<BẠN VIẾT>>
  2. <<BẠN VIẾT>>
  3. <<BẠN VIẾT>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có người lái + khách) | SAM 2.1 base | 3 (2 dương, 1 âm lên người lái) | <<BẠN VIẾT>> | <<BẠN VIẾT>> | `tuktuk_img013.jpg` (mask 45.4% ảnh) |
| img_004 (tuk-tuk xanh dương YOLO26 bỏ sót) | SAM 2.1 base | 4 (3 dương, 1 âm lên người áo đen) | <<BẠN VIẾT>> | <<BẠN VIẾT>> | `tuktuk_blue_img004.jpg` (mask 13.3% ảnh) |
| img_032 (xích lô) | SAM 2.1 base | 4 (3 dương, 1 âm lên người đội mũ trắng) | <<BẠN VIẾT>> | <<BẠN VIẾT>> | `xichlo_img032.jpg` (mask 8.6% ảnh) |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<BẠN VIẾT>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không làm được vì CVAT Docker (v2.82.1) chưa cài serverless (Nuclio), menu Models không có model nào; đã thử chuẩn bị deploy hàm SAM (`serverless/pytorch/facebookresearch/sam`) nhưng dừng ở bước cài `nuctl`. Thay bằng `scripts/sam2_points.py` (SAM 2.1 base, click dương/âm) để mô phỏng Interactor.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`: CAL-01, CAL-02 `loi_may`; CAL-03, CAL-04 `cau_hoi_mo`); quan trọng nhất (dẫn `CAL-xx`): <<BẠN VIẾT>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu hỏi mở 4 (gán `person` khi thấy rõ đầu hoặc thân trên) và 5 (kính thuộc mask xe, khoảng hở lớn của khung bỏ khỏi mask); thêm ví dụ `three_wheeler` (img_013, img_004) và bán tải = `car` (img_002).
- Nguyên nhân gốc: <<BẠN VIẾT: ca nào do guideline mơ hồ (CAL-03, 04?), ca nào do máy (CAL-01, 02?), ca nào do bạn (bẫy loose lọt?)>>

## 7. QC và quyết định (Part 6)

- Mẫu: 7 ảnh (2 ảnh `can_review`: img_003, img_013 + 5 ngẫu nhiên: img_004, img_008, img_017, img_018, img_034; seed 42), 149 đối tượng; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 3/149 = 2.0% · theo **ảnh**: 2/7 = 28.6% (ngưỡng Release ≤ 2%)
- Top lỗi: bỏ sót 3/3 lỗi (img_004: tuk-tuk xanh dương ở xa bên trái; img_008: 2 người ở mép phải / sau sạp hàng). Không có lỗi thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính trong mẫu.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 2/7 ảnh lỗi, cận trên 95% (Wilson) là 64%. <<BẠN VIẾT: ý nghĩa với ngưỡng 2%>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<BẠN VIẾT>>

**Quyết định: <<BẠN CHỌN: Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<BẠN VIẾT>>
Nếu Rework / Hold, việc cần làm tiếp: <<BẠN VIẾT>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<BẠN VIẾT: Claude Code (cài đặt, chạy script, đối chiếu số liệu); chatbot khác (viết nháp nhận xét model_log / QC, đã gửi ảnh pilot/QC lên); cách bạn kiểm lại, ví dụ box #486 img_034 chatbot nói sai>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<BẠN VIẾT: có gửi ảnh cắt img_001 cho Claude hỏi; Claude chỉ dẫn quy tắc, không gợi ý nhãn; chưa nhìn pre-label của gold>>
- Ca `can_review` khi gán gold (ghi lúc gán, <<viết lại bằng lời của bạn>>):
  - img_001: person nghi là bóng người (thấy đen, không chắc là người hay bóng).
  - img_036: motorbike bị gương chiếu hậu / xe khác che một phần, chỉ chắc ~70–80% là xe máy.
- Sửa gold (`gold_fix`, 10 phút): lần đầu gán cả vật cao < 20 px (88 đối tượng, 25 `can_review`); sửa theo mục 4.1.6 còn 62 đối tượng. Còn giữ 3 box 16.9–19.8 px vì <<lý do của bạn>>.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GTX 1650 4 GB, Python 3.13.5, CVAT Docker v2.82.1. Lần cài đầu pip lấy torch bản CPU từ PyPI, phải cài lại torch 2.14.1+cu126 để dùng GPU.
- Dùng cache `data/cache/` thay vì tự chạy? Không: tự chạy mọi lệnh trên GPU.
- Lỗi / bug script đã gặp (nếu có): không gặp bug script.
- Một điều bạn làm khác lần sau: <<BẠN VIẾT>>
