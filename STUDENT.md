# Thông tin học viên

- Họ và tên: Nguyễn Bá Duy
- MSHV: 2A202603192
- Lớp (L2 / L3 / khác): K5-L23
- Tài khoản GitHub: basduy05
- Repo bài nộp (URL): https://github.com/basduy05/K5-L23-Day08-NguyenBaDuy-2A202603192-AI-Assisted-Labeling
- Commit chốt bài (hash): 7424dd8
- Máy: Windows 11 x64, GPU NVIDIA Quadro P1000 (CUDA 12.4)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 401 / 0 / 0 (mẫu số: 401 box máy); added: 3 (mẫu số: 404 đối tượng)
- So với gold: precision / recall của **máy**: 58/63 = 92.1% / 58/61 = 95.1% (mẫu số: 61 đối tượng gold)
- Bắt bẫy: 11/12 = 91.7%
- Thời gian: gán tay 5.5 phút/ảnh (22 phút / 4 ảnh); duyệt có pre-label 3.0 phút/ảnh (72 phút / 24 ảnh)
- Lỗi QC: 37.5% theo ảnh (3/8 ảnh), 1.9% theo đối tượng (3/155 đối tượng)
- **Quyết định:** Rework — lý do một dòng: Tỉ lệ ảnh có lỗi trên mẫu kiểm định đạt 37.5% vượt ngưỡng chất lượng 2% theo đúng cây quyết định QC.
