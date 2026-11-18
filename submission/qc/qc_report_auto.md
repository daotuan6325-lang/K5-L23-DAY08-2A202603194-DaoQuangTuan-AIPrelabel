# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 7 ảnh (2 ca khó + 5 ngẫu nhiên), 149 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 3/149 = 2.0% | 149 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 2/7 = 28.6% | 7 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 497/544 = 91.4% / 11/544 = 2.0% / 24/544 = 4.4% | 544 box máy |
| Máy: added | 18/526 = 3.4% | 526 đối tượng sau duyệt |
| Bắt lỗi bẫy | 5/12 = 41.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 3 | img_004.jpg, img_008.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 2 ảnh lỗi trong 7 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **64%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 28.6% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 3/3 lỗi
- bắt bẫy 42% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
