# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 544; đối tượng sau duyệt: 526; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 497 | 497/544 = 91.4% | 544 box máy |
| edited (sửa) | 11 | 11/544 = 2.0% | 544 box máy |
| deleted (xoá) | 24 | 24/544 = 4.4% | 544 box máy |
| added (vẽ thêm) | 18 | 18/526 = 3.4% | 526 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 13.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 286 | 2 | 4 | 5 |
| bicycle | 5 | 1 | 1 | 2 |
| motorbike | 132 | 1 | 0 | 5 |
| car | 53 | 0 | 1 | 0 |
| bus | 3 | 0 | 0 | 3 |
| truck | 5 | 0 | 1 | 1 |
| three_wheeler | 13 | 7 | 17 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 54 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 44 | 20 | 10 | 68.8% (KTC 95%: 57–79%) | 81.5% (KTC 95%: 69–90%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 5/12 = 41.7% (KTC 95%: 19–68%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - extra: 1/3
  - missing: 1/3
  - loose: 0/3
  - lọt: img_026.jpg (missing), img_010.jpg (loose), img_028.jpg (loose), img_029.jpg (extra), img_024.jpg (missing), img_021.jpg (extra), img_004.jpg (loose)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
