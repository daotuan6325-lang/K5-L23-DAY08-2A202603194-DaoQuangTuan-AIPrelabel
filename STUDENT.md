# Thông tin học viên

- Họ và tên: Đào Quang Tuấn
- MSHV: 2A202603194
- Lớp (L2 / L3 / khác): L2A
- Tài khoản GitHub: daotuan6325-lang
- Repo bài nộp (URL): https://github.com/daotuan6325-lang/K5-L23-DAY08-2A202603194-DaoQuangTuan-AIPrelabel
- Commit chốt bài (hash): b42dc62
- Máy: Windows 11 Home, GPU NVIDIA GeForce GTX 1650 4 GB (CUDA), Python 3.13.5
- CVAT (Docker): v2.82.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 497/544 = 91.4% / 11/544 = 2.0% / 24/544 = 4.4% (mẫu số: 544 box máy); added: 18/526 = 3.4% (mẫu số: 526 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 44/64 = 68.8% / 44/54 = 81.5%
- Bắt bẫy: 5/12 = 41.7%
- Thời gian: gán tay 5.75 phút/ảnh (23 phút / 4 ảnh); duyệt có pre-label 1.25 phút/ảnh (30 phút / 24 ảnh)
- Lỗi QC: 28.6% theo ảnh (2/7), 2.0% theo đối tượng (3/149)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 2/7 = 28.6% > 2%, bắt bẫy 5/12 = 41.7% < 80%, lỗi lặp là bỏ sót (3/3 lỗi QC).
