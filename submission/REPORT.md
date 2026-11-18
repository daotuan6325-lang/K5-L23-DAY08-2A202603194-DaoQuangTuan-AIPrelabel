# Báo cáo Day 08 — 2A202603194 - Đào Quang Tuấn

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person = 0.25 (sweep: recall 20/22 = 91%, precision 20/29 = 69%); các lớp khác mặc định 0.3 (motorbike, car: không ngưỡng nào ≤ 0.5 đạt recall 90%; bicycle, truck, bus: < 5 vật trong gold) | Chọn 0.25 thay vì 0.5: ở r03 (0.5) img_002 mất 4 người nhỏ / bị che mà không lọc thêm được box sai nào (r02 → r03); sweep cho thấy person ở 0.25 đạt recall 91%, ở 0.5 chỉ 14/22 = 64%. |
| three_wheeler | YOLOE-26s, prompt: "tuk-tuk", "cyclo", "cargo tricycle" | 0.15 (sweep: recall 7/7 = 100%, precision 7/9 = 78%; ở 0.5 recall chỉ 3/7 = 43%) | Ở r04 tôi định dùng 0.5–0.6 để bỏ box rác, nhưng sweep cho thấy ở 0.5 máy chỉ tìm được 3/7 xe ba bánh trong gold, và ở img_032 xích lô thật chỉ được 0.48 và 0.27, nên đổi sang 0.15. |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.25,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json` → 633 box; 11 box `car` và 6 box `truck` của YOLO26 bị loại vì chồng lên box `three_wheeler` của YOLOE.
- **Phiên bản SAM bạn chọn**: SAM 2.1 base (`sam2.1_b.pt`). Lý do: máy có GPU (GTX 1650) nên chạy được bản base, 3.37 s/ảnh ở r05, mask đẹp hơn SAM 1 (mobile_sam).
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Tuk-tuk bị YOLO26 gọi `car` (img_013 `car` 0.42) vì COCO không có lớp xe ba bánh: tăng hay giảm ngưỡng chỉ làm box `car` đó còn hoặc mất, không bao giờ thành `three_wheeler` (sweep: `car` vẫn 7 box sai ở mọi ngưỡng ≤ 0.25). Sửa bằng đổi model: dùng YOLOE cho `three_wheeler`, lúc merge 11 box `car` và 6 box `truck` của YOLO26 bị thay.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Điểm cao mà sai: img_002 bán tải bị gán `truck` 0.73. Điểm thấp mà đúng: img_032 hai xích lô thật chỉ được 0.48 và 0.27 (YOLOE).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: tôi tin ngưỡng này ít, vì chỉ có 4 ảnh, `three_wheeler` chỉ 7 vật nằm trong 2 ảnh, và ngưỡng được chọn rồi chấm trên cùng dữ liệu nên precision / recall trên gold là lạc quan.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 497 | 497/544 = 91.4% | 544 box máy |
| edited | 11 | 11/544 = 2.0% | 544 box máy |
| deleted | 24 | 24/544 = 4.4% | 544 box máy |
| added | 18 | 18/526 = 3.4% | 526 đối tượng sau duyệt |

(Đã loại 23 đối tượng quanh vùng bẫy khỏi bảng. Trong deleted + added có 5 ca đổi lớp; máy sót thật ≈ 18 − 5 = 13.)

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? three_wheeler: accepted 13, edited 7, deleted 17, added 2. Lỗi nhiều nhất là box thừa: ngưỡng YOLOE 0.15 rất thấp nên bắt nhầm vật không phải xe ba bánh (như ở r04: xe đẩy hàng 0.40 và bán tải 0.37 ở img_002, box rác dọc mép img_011).
- So với gold: precision / recall của **máy** 44/64 = 68.8% (KTC 95%: 57–79%) / 44/54 = 81.5% (KTC 95%: 69–90%) (mẫu số: 54 đối tượng gold)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: 5/12 = 41.7% (KTC 95%: 19–68%); theo loại: wrong_class 3/3, extra 1/3, missing 1/3, loose 0/3. Lọt: img_026 (missing), img_024 (missing), img_029 (extra), img_021 (extra), img_010 (loose), img_028 (loose), img_004 (loose). Vì sao: tôi chỉ làm bước 0 (ẩn nhãn máy, quét ảnh trước) ở vài ảnh đầu rồi bỏ, nên lọt 2/3 bẫy thiếu box; tôi chỉ phóng to kiểm biên khi box trông lệch rõ, nên lọt cả 3/3 bẫy box lỏng. Bẫy sai lớp thì bắt đủ 3/3 vì tôi có đọc nhãn lớp.
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? Accepted 91.4% nhưng bắt bẫy chỉ 41.7%: tôi thường giữ nguyên box máy trông hợp lý, nhất là box score cao, thay vì tự kiểm. Đây là thiên kiến tự động hóa: accepted cao phản ánh việc tôi bấm giữ, không chứng minh máy tốt (recall máy trên gold chỉ 81.5%).

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 23 phút / 4 ảnh = 5.75 phút/ảnh (33 phút = 8.25 phút/ảnh nếu tính 10 phút sửa gold) vs duyệt 30 phút / 24 ảnh = 1.25 phút/ảnh (không tính 6 phút nghỉ). Thiên lệch: gán gold trước giúp tôi quen guideline và cảnh phố nên review nhanh hơn; ngoài ra review nhanh một phần vì tôi bỏ bước 0 và ít zoom, nên tốc độ 1.25 phút/ảnh không hoàn toàn là nhờ máy.
- Kết luận: có, máy đáng dùng (nhanh hơn ~4.6 lần, accepted 91.4%), nhưng phải review chậm và kỹ hơn, dựa trên bắt bẫy mới 5/12 và QC còn 2/7 ảnh lỗi.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Làm bước 0 ở 100% ảnh: tự đếm vật rồi ghi số đếm vào note trước khi hiện nhãn máy (kiểm được: đủ 24 dòng ghi đếm).
  2. Zoom ≥ 2x kiểm biên ở mọi box `three_wheeler` và mọi box score ≥ 0.8 (đo bằng tỉ lệ bắt bẫy loose, mục tiêu 3/3).
  3. Ảnh đông quét theo 4 góc phần tư, đối chiếu số vật đếm được với số box (kiểm được: chênh lệch ghi vào note, mục tiêu không còn `bo_sot` trong QC).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có người lái + khách) | SAM 2.1 base | 3 (2 dương, 1 âm lên người lái) | Một phần: tô phần lớn người lái dù đã click âm, và cả khách / người nhìn qua khung hở | Mask phủ đủ cả xe, nhưng 1 click âm không đủ tách người ra khỏi xe | `tuktuk_img013.jpg` (mask 45.4% ảnh) |
| img_004 (tuk-tuk xanh dương YOLO26 bỏ sót) | SAM 2.1 base | 4 (3 dương, 1 âm lên người áo đen) | Không: người áo đen không bị tô | Ôm trọn chiếc tuk-tuk xanh; không lỗi rõ | `tuktuk_blue_img004.jpg` (mask 13.3% ảnh) |
| img_032 (xích lô) | SAM 2.1 base | 4 (3 dương, 1 âm lên người đội mũ trắng) | Không: người đội mũ trắng không bị tô | Phần trong vành bánh (giữa nan hoa) bị tô; yên người đạp và bánh sau không thấy rõ trong ảnh nên không đánh giá được | `xichlo_img032.jpg` (mask 8.6% ảnh) |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? Quy tắc lỗ thủng (4.2.2) và bóng đổ (4.2.3) giúp đánh giá mask, nhưng 4.2 chưa nói rõ phần nhìn xuyên qua kính / khung hở của tuk-tuk (img_013 mask tô cả người và khách phía sau) nên tôi thêm CAL-04.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: không làm được vì CVAT Docker (v2.82.1) chưa cài serverless (Nuclio), menu Models không có model nào; đã thử chuẩn bị deploy hàm SAM (`serverless/pytorch/facebookresearch/sam`) nhưng dừng ở bước cài `nuctl`. Thay bằng `scripts/sam2_points.py` (SAM 2.1 base, click dương/âm) để mô phỏng Interactor.

## 6. Calibration (Part 6)

- Số quyết định: 4 (`calibration_decisions.csv`: CAL-01, CAL-02 `loi_may`; CAL-03, CAL-04 `cau_hoi_mo`); quan trọng nhất (dẫn `CAL-xx`): CAL-01 (tuk-tuk), vì là lỗi máy lặp nhiều nhất: khi review phải xoá 17 và sửa 7 box `three_wheeler`.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: chốt câu hỏi mở 4 (gán `person` khi thấy rõ đầu hoặc thân trên) và 5 (kính thuộc mask xe, khoảng hở lớn của khung bỏ khỏi mask); thêm ví dụ `three_wheeler` (img_013, img_004) và bán tải = `car` (img_002).
- Nguyên nhân gốc: CAL-01, CAL-02 do máy sai (guideline v1.0 đã đúng, chỉ thêm ví dụ); CAL-03, CAL-04 do guideline mơ hồ (câu hỏi mở 4, 5); bẫy lọt (loose 0/3, missing 1/3) là do tôi bỏ bước của checklist, không cần đổi guideline.

## 7. QC và quyết định (Part 6)

- Mẫu: 7 ảnh (2 ảnh `can_review`: img_003, img_013 + 5 ngẫu nhiên: img_004, img_008, img_017, img_018, img_034; seed 42), 149 đối tượng; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: 3/149 = 2.0% · theo **ảnh**: 2/7 = 28.6% (ngưỡng Release ≤ 2%)
- Top lỗi: bỏ sót 3/3 lỗi (img_004: tuk-tuk xanh dương ở xa bên trái; img_008: 2 người ở mép phải / sau sạp hàng). Không có lỗi thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính trong mẫu.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 2/7 ảnh lỗi, cận trên 95% (Wilson) là 64%: tỉ lệ ảnh lỗi thật có thể cao tới 64%, nên mẫu 7 ảnh không thể chứng minh lô đạt ngưỡng 2%, kể cả khi mẫu sạch hoàn toàn.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script đếm được số lỗi, tỉ lệ, accepted / edited / deleted / added, bẫy bắt được; không kiểm được box có ôm sát vật thật không, vật mà cả tôi lẫn máy cùng bỏ sót ngoài gold, và việc tôi có theo checklist thật không: những việc này chỉ người nhìn ảnh mới kiểm được (ví dụ box #486 img_034 tôi phải tự xem lại).

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): không Hold vì guideline đã chốt các ca gặp trong lô và dữ liệu không hỏng; Rework vì ảnh lỗi 2/7 = 28.6% > 2%, bắt bẫy 5/12 = 41.7% < 80%, và lỗi lặp là bỏ sót (3/3 lỗi QC).
Nếu Rework / Hold, việc cần làm tiếp: review lại 24 ảnh, bắt buộc làm bước 0 ở mọi ảnh, tìm vật bỏ sót (img_004, img_008…), export `qc_fixed.json`, rồi chạy lại `build_release.py` và `qc_sample.py --force`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude Code (Claude Opus): cài môi trường, chạy các script (prelabel, threshold_sweep, merge, inject_traps, review_stats, build_release, qc_sample, qc_report, check_submission), đối chiếu số liệu, điền số liệu từ file đo và ghép câu trả lời của tôi thành câu văn trong REPORT; tôi chọn các quyết định và kết luận. Cách kiểm: tự mở ảnh `viz/`, ảnh mask, ảnh QC để xác nhận (ví dụ kiểm lại box #486 img_034 có người áo xanh sau cột điện). Không dùng chatbot nào khác.
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? Không nhìn pre-label của gold trước khi gán. Trong lúc gán có gửi 2–3 ảnh cắt từ img_001 cho Claude Code hỏi cách xử lý; Claude nhận ra đó là ảnh gold, chỉ dẫn quy tắc guideline, không gợi ý nhãn; quyết định do tôi.
- Ca `can_review` khi gán gold (ghi lúc gán):
  - img_001: person nghi là bóng người (thấy đen, không chắc là người hay bóng).
  - img_036: motorbike bị gương chiếu hậu / xe khác che một phần, chỉ chắc ~70–80% là xe máy.
- Sửa gold (`gold_fix`, 10 phút): lần đầu gán cả vật cao < 20 px (88 đối tượng, 25 `can_review`); sửa theo mục 4.1.6 còn 62 đối tượng. Còn giữ 3 box 16.9–19.8 px vì gần 20 px và vật nhìn rõ, tôi tự quyết giữ.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: Windows 11, GTX 1650 4 GB, Python 3.13.5, CVAT Docker v2.82.1. Lần cài đầu pip lấy torch bản CPU từ PyPI, phải cài lại torch 2.14.1+cu126 để dùng GPU.
- Dùng cache `data/cache/` thay vì tự chạy? Không: tự chạy mọi lệnh trên GPU.
- Lỗi / bug script đã gặp (nếu có): không gặp bug script.
- Một điều bạn làm khác lần sau: review chậm hơn, theo đủ checklist ở mọi ảnh.
