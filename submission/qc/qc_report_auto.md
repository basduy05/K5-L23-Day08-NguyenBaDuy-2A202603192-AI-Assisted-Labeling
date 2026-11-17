# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 8 ảnh (3 ca khó + 5 ngẫu nhiên), 155 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 3/155 = 1.9% | 155 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 3/8 = 37.5% | 8 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 401/401 = 100.0% / 0/401 = 0.0% / 0/401 = 0.0% | 401 box máy |
| Máy: added | 3/404 = 0.7% | 404 đối tượng sau duyệt |
| Bắt lỗi bẫy | 11/12 = 91.7% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 1 | img_013.jpg |
| sai_vi_tri_kich_thuoc | 1 | img_018.jpg |
| bo_sot | 1 | img_021.jpg |

## Giới hạn thống kê
- 3 ảnh lỗi trong 8 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **69%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 37.5% > 2%

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
