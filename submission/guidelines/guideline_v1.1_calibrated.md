# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603192 - Nguyễn Bá Duy

> Bản bổ sung quy chuẩn gán nhãn v1.1 dựa trên kết quả thực nghiệm pre-label và các ca biên thu được qua quá trình review 24 ảnh batch và 4 ảnh gold. Mọi quyết định đều đối chiếu với `submission/qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Xe có bàn đạp trợ lực thể lực gán `bicycle`; xe không bàn đạp, sàn để chân dáng scooter gán `motorbike` | `img_004.jpg` | `CAL-02` |
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Xe phân biệt rõ biên dạng và cao $\ge 20\text{ px}$ gán từng xe; khối xe dính sát ở cự ly xa hoặc bị che $> 80\%$ gom vào `ignore_region` | `img_018.jpg` | `CAL-03` |
| 3 | Hàng nhô ra khỏi xe: box xe có gồm hàng không? | Hàng gắn cố định trong giỏ/thùng tính vào box xe; hàng cồng kềnh nhô quá 20% chiều dài xe thì cắt mép box tại khung xe | `img_017.jpg` | `CAL-01` |
| 4 | Người ngồi trong xe ba bánh có mui che nhìn thấy thấp thoáng | Nếu diện tích cơ thể nhìn thấy $\ge 20\%$ và phân biệt được đầu/vai thì vẽ box `person`; dưới 20% chỉ tính vào thân xe `three_wheeler` | `img_013.jpg` | `CAL-01` |

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_013.jpg` | Xe tuk-tuk chở khách có mui bạt hở sườn | YOLO26m gán nhầm thành `car` (conf 0.42); YOLOE vẽ 2 box chồng đè | Chuyển toàn bộ về `three_wheeler`, xoá box trùng lặp, tách riêng người lái bằng box `person` | Mục 2 & 5: Xe ba bánh mọi biến thể gắn động cơ/đạp chân đều thuộc `three_wheeler` |
| `img_004.jpg` | Xe ba gác chở đồ gia dụng dừng bên đường | YOLO26m bỏ sót hoặc nhầm `truck`; YOLOE bắt đúng nhưng box hơi lỏng | Kéo khít viền box theo mép bánh và thành xe, gán `three_wheeler` | Mục 2 & 4.1: Box ôm sát phần cơ khí, không bao gồm khoảng trống mặt đường |
| `img_018.jpg` | Cụm xe máy và xe đạp nhỏ ở xa góc ngã tư | Detector sinh ra các box rác cao 15-18 px gây cảnh báo validate | Xóa các box nhỏ $< 20\text{ px}$ và vẽ một box `ignore_region` bao quanh toàn cụm xe ở xa | Mục 4.1.6: Bắt buộc dùng `ignore_region` cho đám đông vật thể $< 20\text{ px}$ |
| `img_021.jpg` | Xe máy bị khuất nửa thân sau biển hiệu ở hậu cảnh | Bẫy missing nhân tạo bị lọt do mắt người bị hút vào người đi bộ trung tâm | Giữ nguyên nguyên tắc kiểm soát mắt trước gợi ý máy, bổ sung box `motorbike` khi thấy $\ge 20\%$ | Mục 4.1.2: Không bỏ qua vật thể nhận diện được cấu trúc dù bị che khuất |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản hợp đồng nhãn gốc giảng viên soạn thảo | Khởi tạo dự án |
| v1.1 | - Chốt phân loại xe đạp điện có bàn đạp (`bicycle`) vs xe máy điện (`motorbike`).<br>- Chuẩn hóa quy tắc bao phủ cho `three_wheeler` (ưu tiên YOLOE, sửa nhãn sai của YOLO26).<br>- Quy định bắt buộc vùng `ignore_region` cho cụm vật thể $< 20\text{ px}$ ở hậu cảnh.<br>- Cụ thể hóa định lượng che khuất $\ge 20\%$ diện tích thân xe và người lái. | Giải quyết các câu hỏi mở mục 6, sửa lỗi nhầm lớp của detector đóng và khắc phục bẫy missing (`CAL-01`, `CAL-02`, `CAL-03`, `CAL-04`) |
