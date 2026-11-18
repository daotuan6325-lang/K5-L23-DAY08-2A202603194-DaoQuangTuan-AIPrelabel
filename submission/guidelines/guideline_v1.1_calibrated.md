# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603194 - Đào Quang Tuấn

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 4 | Người ngồi trong xe ba bánh có mui che (chỉ thấy thấp thoáng): ngưỡng "còn thấy ≥ 20%" có hợp lý không? | Cụ thể hoá mục 4.1.2: gán `person` khi thấy rõ **đầu hoặc thân trên**; chỉ thấy tay / chân / vạt áo thì **không gán** | img_013, img_002 | CAL-03 |
| 5 | Kính chắn gió / cửa sổ xe thuộc mask xe không? | **Kính chắn gió thuộc mask xe** (giữ v1.0 mục 4.2.3). **Khoảng hở lớn của khung** tuk-tuk / xích lô (nhìn xuyên thấy người, ghế phía sau) **bỏ ra khỏi mask** theo quy tắc lỗ thủng 4.2.2. Người nhìn thấy qua kính / khung là box / mask `person` riêng | img_001, img_013, img_004 | CAL-04 |

Chưa chốt: câu 1 (xe đạp điện), 2 (khối xe máy đỗ dày gần camera), 3 (hàng nhô ra), 6 (box máy chồng nhau ảnh đêm), 7 (conf tối thiểu): chưa gặp đủ ví dụ trong lô này để quyết định.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_004 | Tuk-tuk / xích lô | YOLO26 gọi `car` (img_013 `car` 0.42, box cắt mất nửa sau xe) hoặc bỏ sót (img_004 tuk-tuk xanh dương không có box) | Luôn gán `three_wheeler`, box gồm cả mái và thùng; máy gọi `car`/`truck` thì đổi lớp | Mục 2, lớp `three_wheeler`: thêm ví dụ (CAL-01) |
| img_002 | Xe bán tải trắng | YOLO26 gán `truck` 0.73 (điểm cao mà sai lớp) | Giữ v1.0: bán tải là `car`; box `truck` điểm cao vẫn phải kiểm lớp | Mục 2, lớp `car`: thêm ví dụ (CAL-02) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 4 (tiêu chí đầu / thân trên thay cho ước lượng 20%) và 5 (kính thuộc xe, khoảng hở khung bỏ khỏi mask); thêm ví dụ cho `three_wheeler` và bán tải = `car` | CAL-01, CAL-02, CAL-03, CAL-04 |
