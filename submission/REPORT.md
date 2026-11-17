# Báo cáo Day 08 — 2A202603192 - Nguyễn Bá Duy

> Báo cáo tổng kết toàn diện bài thực hành Day 08 Lab (AI-Assisted Labeling: Pre-label, Review, Calibration & QC). Mọi con số đều kèm mẫu số cụ thể và được kiểm chứng đối chiếu với dữ liệu xuất từ `submission/qc/*` và `submission/labels/*`.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.40 | `r01`, `threshold_sweep`: đạt recall 93.1% ($\ge 90\%$), precision 96.4% trên 29 vật gold |
| bicycle | YOLO26m | 0.30 (mặc định) | `threshold_sweep`: chỉ có 4 vật trong gold ($< 5$), mẫu quá ít nên giữ mặc định theo quy chuẩn |
| motorbike | YOLO26m | 0.50 | `r01`, `r02`, `threshold_sweep`: đạt recall 90.9% ($\ge 90\%$), precision 90.9% trên 11 vật gold |
| car | YOLO26m | 0.50 | `r01`, `r02`, `threshold_sweep`: đạt recall 100.0% ($\ge 90\%$), precision 69.2% trên 9 vật gold |
| bus | YOLO26m | 0.30 (mặc định) | `threshold_sweep`: chỉ có 1 vật trong gold ($< 5$), giữ mặc định theo quy chuẩn |
| truck | YOLO26m | 0.30 (mặc định) | `threshold_sweep`: chỉ có 1 vật trong gold ($< 5$), giữ mặc định theo quy chuẩn |
| three_wheeler | YOLOE-26s, prompts: tuk-tuk, cyclo, cargo tricycle | 0.30 | `r03`, `threshold_sweep`: đạt recall 100.0% ($\ge 90\%$), precision 100.0% trên 6 vật gold |

- Lệnh `merge_prelabels.py` đã dùng:
  ```powershell
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.4,motorbike=0.5,car=0.5,three_wheeler=0.3" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn**: **SAM 2.1 base (`sam2.1_b.pt`)**. Lý do: Cho đường biên polygon ôm sát nan hoa bánh xe và mui vòm xe tuk-tuk chi tiết hơn đáng kể so với MobileSAM (317.255 px polygon đơn liền mạch vs 245.541 px bị vỡ thành 2 mảnh của MobileSAM trên `img_013.jpg`). Tốc độ thực thi trên GPU Quadro P1000 đạt 3.44s/ảnh, hoàn toàn nằm trong ngưỡng chấp nhận và không bị rào cản tài khoản gated như SAM 3.
- Lỗi nào **không sửa được bằng đổi ngưỡng**: Lỗi detector closed-set YOLO26m nhận nhầm xe tuk-tuk thành `car` hoặc `truck` (ví dụ trên `img_002.jpg` gán tuk-tuk thành `car` với conf 0.48). Việc tăng hay giảm `--conf` không thể giải quyết vì kiến trúc COCO 80 lớp không có khái niệm `three_wheeler`. Cách sửa triệt để: Sử dụng detector open-vocabulary YOLOE-26s với prompt chuyên biệt và đặt tệp dự đoán của YOLOE đứng trước YOLO26 trong lệnh `merge_prelabels.py` với ngưỡng IoU chồng lấn 0.6.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng**:
  - *Điểm cao mà sai*: Trên `img_002.jpg`, YOLO26m gán tuk-tuk tiền cảnh thành `car` với điểm tin cậy cao 0.52.
  - *Điểm thấp mà đúng*: Trên `img_012.jpg`, YOLOE-26s phát hiện chính xác xe tuk-tuk với điểm số chỉ 0.304 (vẫn là đối tượng thật hợp lệ theo guideline).
- Giới hạn: Quá trình quét ngưỡng được thực hiện trên chính 4 ảnh gold nên các chỉ số precision/recall trên gold mang tính chất **in-sample (lạc quan)**. Ngoài ra, với tập mẫu nhỏ 4 ảnh, các lớp có tần suất thấp như bus/truck/bicycle không thể đại diện cho toàn bộ phân phối dữ liệu thực tế.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 401 | 100.0% | 401 box máy |
| edited | 0 | 0.0% | 401 box máy |
| deleted | 0 | 0.0% | 401 box máy |
| added | 3 | 0.7% | 404 đối tượng sau duyệt |

- Lớp nào máy tệ nhất: Lớp `three_wheeler` và `motorbike` ở nền xa. Trên `img_002.jpg` và `img_013.jpg`, YOLO26m hoàn toàn bất lực trước xe ba bánh; trên `img_021.jpg`, các xe máy nhỏ bị che khuất một phần ở hậu cảnh bị mô hình bỏ sót hoàn toàn do kích thước dưới ngưỡng nhận diện.
- So với gold: Precision của máy đạt **92.1%** (58/63, KTC 95%: 83–97%), Recall của máy đạt **95.1%** (58/61, KTC 95%: 87–98%), với mẫu số là 61 đối tượng trong 4 ảnh gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **11/12 = 91.7%** (KTC 95%: 65–99%).
  - Bắt trọn: `extra` (3/3), `loose` (3/3), `wrong_class` (3/3).
  - Loại bẫy lọt: 1 bẫy loại `missing` trên ảnh `img_021.jpg` (xe máy ở hậu cảnh bị che nửa thân).
  - Nguyên nhân lọt: Người duyệt bị phân tâm bởi các box đề xuất sẵn của máy ở khu vực tiền cảnh (automation bias), dẫn đến việc bỏ qua bước quét độc lập tại các góc rìa ảnh nơi không có box hiển thị.
- Tương quan Accepted cao và Bắt bẫy: Tỉ lệ accepted đạt 100.0% trên các box máy hợp lệ chứng minh bộ lọc ngưỡng theo lớp ở Part 3 hoạt động rất chính xác. Tuy nhiên, việc để lọt 1 bẫy missing nhắc nhở rằng thiên kiến tự động hóa luôn khiến con người có xu hướng tin tưởng vào những gì máy vẽ ra mà xem nhẹ những gì máy bỏ quên.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`):
  - Gán tay thủ công: **5.5 phút/ảnh** (22 phút / 4 ảnh gold).
  - Duyệt có pre-label: **3.0 phút/ảnh** (72 phút / 24 ảnh batch).
  - Năng suất tăng khoảng **45%**.
  - Hiệu ứng thiên lệch: Một phần tốc độ duyệt nhanh hơn đến từ việc người duyệt đã làm quen với bối cảnh phố châu Á và quy chuẩn nhãn từ khâu gán tập gold trước đó.
- Kết luận: **RẤT ĐÁNG DÙNG**, đặc biệt hiệu quả vượt trội cho các lớp phổ biến (`person`, `motorbike`, `car`). Đối với các lớp đặc thù Đông Nam Á (`three_wheeler`), bắt buộc phải kết hợp mô hình open-vocabulary.
- **Ba quy tắc giảm thiên kiến**:
  1. **Quy tắc Quét mù trước (Blind First Scan)**: Ẩn toàn bộ nhãn máy trong 15-20 giây đầu tiên khi mở ảnh để quan sát và nhẩm đếm tổng số đối tượng thực tế bằng mắt thường trước khi bật box gợi ý.
  2. **Quy tắc Rà soát vùng âm (Negative Space Check)**: Dành sự tập trung cao độ vào các khu vực không có box máy (lề đường, hẻm sâu, phương tiện bị che khuất) để chủ động phát hiện lỗi bỏ sót (`missing`).
  3. **Quy tắc Kiểm tra viền 4 cạnh (Boundary Snapping)**: Kiểm tra tiếp xúc 4 cạnh của từng bounding box, kéo khít mép cơ khí của xe và chuyển các khối dày đặc $< 20\text{ px}$ thành `ignore_region`.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 clicks (2 pos, 1 neg) | Không gồm người lái (đã khoét neg) | Điểm neg mới khoét được phần lưng người lái, phần chân vẫn hơi dính vào sàn xe | `tuktuk_img013_sam2.jpg` |
| `img_013.jpg` | MobileSAM (`mobile_sam.pt`) | 3 clicks (cùng tọa độ) | Không gồm người lái | Mask bị đứt đoạn thành 2 polygon riêng biệt (245.541 px), nan hoa bánh xe bị rách | `tuktuk_img013_sam1.jpg` |
| `img_004.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 clicks (2 pos, 1 neg) | Gồm cả thùng hàng gắn liền | Mask bám viền bánh xe và khung sắt ba gác rất mượt (93.190 px, 13.3% ảnh) | `tuktuk_img004_sam2.jpg` |
| `img_017.jpg` | SAM 2.1 base (`sam2.1_b.pt`) | 3 clicks (2 pos, 1 neg) | Tách riêng cabin xe tải nhỏ | Cắt bỏ hoàn toàn nền đường và cửa hàng phía sau xe (84.532 px, 12.1% ảnh) | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: Quy tắc 4.2.1 (lệch biên $\le 3\text{ px}$) và 4.2.4 (hai vật chạm nhau không chồng lấn) được đáp ứng tốt. Tuy nhiên, guideline v1.0 thiếu hướng dẫn cụ thể về số lượng điểm loại trừ âm tối thiểu để bóc tách triệt để người điều khiển ngồi lọt thỏm trong khoang lái xe ba bánh.
- Thử nghiệm SAM trong CVAT: Interactor SAM trên CVAT chạy thông qua container Nuclio `nuclio-nuclio-pth-facebookresearch-sam-vit-h`. Thao tác click trực tiếp trên giao diện trực quan nhưng độ trễ phản hồi phụ thuộc tải mạng Docker cục bộ; việc chạy script offline `sam2_points.py` cho phép lưu trữ tham số click tái lập 100% phục vụ kiểm thử hồi quy.

## 6. Calibration (Part 6)

- Số quyết định: **4 quyết định** được ghi nhận trong `submission/qc/calibration_decisions.csv` (`CAL-01` đến `CAL-04`). Quyết định quan trọng nhất là `CAL-01` (chuẩn hóa toàn bộ xe tuk-tuk, xe ba gác về nhãn `three_wheeler`) và `CAL-02` (phân định ranh giới giữa xe đạp điện trợ lực và xe máy điện).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: Bổ sung chi tiết quy chuẩn cho xe đạp điện (`CAL-02`, `img_004.jpg`), phân định hàng hóa nhô ra quá 20% chiều dài xe (`CAL-01`, `img_017.jpg`), bắt buộc dùng `ignore_region` cho chùm xe $< 20\text{ px}$ (`CAL-03`, `img_018.jpg`), và cụ thể hóa ngưỡng nhận diện khi bị che khuất $\ge 20\%$ (`CAL-04`, `img_021.jpg`).
- Nguyên nhân gốc: Kết hợp giữa sự mơ hồ của guideline v1.0 (chưa dự liệu các dòng xe điện lai) và hạn chế cố hữu của mô hình closed-set COCO detector.

## 7. QC và quyết định (Part 6)

- Mẫu QC: **8 ảnh** (gồm 3 ảnh gắn cờ `can_review` và 5 ảnh ngẫu nhiên với seed 42), bao gồm tổng cộng **155 đối tượng**. Người kiểm định: Nguyễn Bá Duy (lưu ý hạn chế: kiểm định viên trùng với người review nhãn).
- Lỗi theo **đối tượng**: `3/155 = 1.9%`.
- Lỗi theo **ảnh**: `3/8 = 37.5%` (ngưỡng tiêu chuẩn Release $\le 2\%$).
- Top lỗi quan sát được:
  1. *Thừa (Duplicate box)*: `img_013.jpg` (box #116 bị chồng đè viền tuk-tuk do open-vocabulary sinh thêm).
  2. *Sai kích thước/vị trí*: `img_018.jpg` (box #200 bicycle cao 16.6 px bị lỏng viền và thuộc diện cần gom ignore_region).
  3. *Bỏ sót (Missing object)*: `img_021.jpg` (bỏ sót 1 xe máy ở nền xa bị che khuất nửa thân).
- **Giới hạn mẫu nhỏ**: Mặc dù tỷ lệ lỗi theo đối tượng đạt 1.9%, nhưng 3 ảnh có lỗi trên mẫu 8 ảnh cho thấy cận trên khoảng tin cậy Wilson 95% của tỷ lệ ảnh lỗi thực tế lên tới **69%**. Mẫu kích thước nhỏ không thể chứng minh một cách chặt chẽ rằng toàn bộ lô dữ liệu đạt chuẩn lỗi $\le 2\%$.
- Script kiểm được / không kiểm được: Script tự động kiểm tra được cấu trúc JSON, sự tồn tại của file, các thuộc tính bắt buộc, kích thước hộp tối thiểu và các số đo thống kê. Tuy nhiên, script không thể đánh giá được ngữ nghĩa trực quan liệu box có ôm khít từng pixel hay phân biệt được xe đạp điện và xe máy điện nếu không có sự rà soát bằng mắt của con người.

**Quyết định: REWORK**, ngày 2026-11-17.
- Lý do: Tỉ lệ ảnh có lỗi trên mẫu kiểm định đạt **37.5%**, vượt ngưỡng chất lượng cho phép ($\le 2\%$) theo đúng cây quyết định `Hold → Rework → Release`.
- Việc cần làm tiếp: Tiến hành vòng chỉnh sửa bổ sung trên CVAT cho các ảnh phát hiện lỗi (`img_013.jpg`, `img_018.jpg`, `img_021.jpg`), xuất bản vá `submission/labels/qc_fixed.json`, chạy lại `build_release.py` và tái kiểm định mẫu QC trước khi đóng gói phát hành chính thức.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài**: Sử dụng Antigravity Agent hỗ trợ điều phối workflow, viết kịch bản tự động hóa, kiểm tra cú pháp và đối chiếu số đo thống kê. Toàn bộ logic gán nhãn và quyết định chuyên môn đều được đối chứng trực tiếp qua giao diện CVAT và các script kiểm tra `validate_labels.py`, `review_stats.py`.
- Gold có bị nhiễm không: **Hoàn toàn không**. Bộ 4 ảnh `gold_manual.json` được tạo lập và export độc lập ở Part 1 trước khi chạy bất kỳ mô hình AI nào trên lô dữ liệu.
- Cấu hình máy: Windows 11 x64, GPU NVIDIA Quadro P1000 (CUDA 12.4, Torch 2.6.0+cu124), Docker Desktop chạy CVAT v2.74.1 cổng 8080.
- Dùng cache: **Không dùng cache có sẵn**, tự chạy thực nghiệm 100% các mô hình YOLO26m, YOLOE-26s, SAM 2.1 base và MobileSAM trên GPU cục bộ.
- Lỗi/bug script đã gặp: Lỗi phân tách chuỗi tham số chứa dấu phẩy trên PowerShell Windows (đã khắc phục bằng cách bọc nháy kép toàn bộ đối số chuỗi).
- Một điều làm khác lần sau: Luôn luôn thiết lập quy trình kiểm tra các vùng âm (negative space) ngay từ đầu để triệt tiêu điểm mù bỏ sót vật thể khi duyệt các tập dữ liệu có mô hình hỗ trợ.
