# Báo cáo Day 5 — Segmentation Data Lab

- Mã học viên theo lớp: 2A202602069
- Ngày / CVAT local: 17/09/2026 / http://localhost:8080
- Công cụ đã dùng: Brush, Polygon, AI gợi ý tự động (Intelligent Scissors / SAM)

---

## 1. Bài đã nộp

Toàn bộ 9 task (3 tier chính và 6 checkpoint) đã được gán nhãn, kiểm tra chất lượng (QC), lưu trên CVAT và xuất khẩu thành công vào thư mục `submissions/`:

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

*Tất cả các file nộp đều đã vượt qua kiểm tra cấu trúc của script `scripts/inspect_submissions.py` với trạng thái `[OK]` (đầy đủ ảnh, định dạng RLE/Polygon và nhãn lớp khớp với taxonomy).*

---

## 2. Một quyết định trước khi dùng gợi ý

- **Ảnh, vị trí và object Medium đầu tiên tự vẽ:** Ảnh `000000181542.jpg` (ảnh 1 trong task `medium_instance`), đối tượng ô tô (`car`) màu tối đỗ sát lề đường ở nửa dưới bên trái khung hình.
- **Class và quy tắc tôi dùng để chọn biên:** 
  - Gán nhãn class `car`.
  - Quy tắc biên: Chỉ gán nhãn sát theo phần thân vỏ kim loại và lốp xe nhìn thấy được trong thực tế trên ảnh; phần đuôi xe bị một cột biển báo che khuất thì dừng mask ngay sát mép của cột chắn, tuyệt đối không tự ý vẽ ước lượng hoặc nối dài mask ra sau vật che.
- **Nếu dùng gợi ý sau đó (vùng gợi ý sai/đúng, hành động sửa/giữ và lý do):**
  - Khi gán nhãn cho các xe ô tô đỗ xa ở ngã tư, tôi có kích hoạt công cụ gợi ý tự động.
  - Vùng gợi ý nhận diện tương đối tốt đường nét mui xe và kính lái, nhưng có xu hướng bắt nhầm phần bóng đổ (shadow) dưới gầm xe trên mặt đường nhựa thành thân xe, làm mask bị phình to ra mặt đường.
  - Tôi đã dùng công cụ Brush nhỏ (chế độ Erase) để gọt bỏ phần bóng râm bên dưới gầm xe, chỉ giữ lại ranh giới lốp xe tiếp xúc với mặt đường và giữ lại phần thân vỏ phía trên do AI đề xuất.

---

## 3. Một lỗi tôi tìm thấy và sửa

- **Task/ảnh/vùng:** Task `cp2_slice`, ảnh `000000287413.jpg`, cụm 2 xe ô tô cùng màu đỗ song song sát nhau.
- **Lỗi thuộc loại:** Gộp-tách (gộp nhầm 2 instance thành 1 mask).
- **Bằng chứng tôi nhìn thấy:** Hai xe ô tô đỗ sát nhau; khi tô nét ban đầu, mask bị dính liền qua khoảng hẹp giữa hai thân xe, dẫn đến trong danh sách Objects chỉ có 1 object `car` duy nhất bao trùm cả hai xe.
- **Quy tắc và hành động sửa:** 
  - Quy tắc Instance Segmentation yêu cầu mỗi cá thể đếm được phải là một mask độc lập có ID riêng.
  - Tôi đã phóng to (zoom in) tối đa, dùng Brush/Eraser kích thước 1–2px để cắt đứt dải pixel nối giữa hai xe dựa trên khe sáng nhìn thấy giữa hai thân xe, sau đó tạo thêm một Object riêng biệt để chia thành 2 instance `car` độc lập.
- **Sau sửa đã Save và export lại chưa?** Đã bấm Save trên CVAT và export lại thành `cp2_slice.zip`. Script `inspect_submissions.py` xác nhận đạt 16 annotations COCO RLE hợp lệ.
- **Kết quả tự đánh giá với Ground Truth (qua script `scoring/scorecard.py`):**
  - **Tổng điểm tự đánh giá 3 tier:** **39.4 / 82 điểm** (chi tiết lưu tại `reports/tiers/SCORECARD.md`).
  - `easy_semantic`: Đạt **14.9 / 20 điểm** (mIoU: 0.735, Coverage: 95.5%). Các lớp cơ bản đạt IoU rất cao: `road` đạt **0.979**, `sky` đạt **0.895**, `building` đạt **0.820**; lớp cần cải thiện là `sidewalk` (IoU **0.322**) do ranh giới bó vỉa bị màu bùn đất che lấp.
  - `medium_instance`: Đạt **10.5 / 32 điểm** (Metric: 0.548, Mean Matched IoU: **0.762**, R@0.5: **0.72**). Bắt đúng 51 True Positives (nộp 67 so với 71 ground truth).
  - `hard_panoptic`: Đạt **14.0 / 30 điểm** (PQ: 0.411, SQ: **0.658**, RQ: 0.509). Trong đó lớp `sky` đạt PQ **0.907**, `building` đạt PQ **0.685**, `car` đạt PQ **0.591** (bắt đúng 21 xe).

---

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| :--- | :--- | :--- | :--- |
| **1. `easy_semantic` & `cp4_curb`:** Đoạn ranh giới vỉa hè (`sidewalk`) và lòng đường (`road`) khi bề mặt vỉa hè bám bụi đất có màu xám tương đồng với nhựa đường | Dễ bị nhầm toàn bộ thành `road` do màu sắc đồng nhất; hoặc chia tách `sidewalk` | Dựa vào gờ bó vỉa (curb) có cao độ nổi lên so với mặt đường và phân chia công năng giao thông (lòng đường xe chạy vs vỉa hè cho người đi bộ) | Quyết định gán theo mép gờ bó vỉa: phần gờ và mặt sàn nâng cao gán `sidewalk`, phần lòng đường gán `road`. *Câu hỏi cho coach:* Với đoạn bó vỉa bị mòn bằng mặt đường thì quy ước lấy theo rãnh thoát nước hay mép cây cối? |
| **2. `cp1_holes`:** Kính chắn gió và cửa sổ ô tô trong suốt nhìn thấy người và hậu cảnh phía sau xe | Khoét lỗ thủng (hole) để gán nhãn người/nền bên trong, hay tô trùm kín thành một khối `car` | Guideline bài lab quy định kính và các chi tiết cơ học trên xe là bộ phận gắn liền với cấu trúc xe | Quyết định không khoét lỗ kính, vẽ bao trùm toàn bộ kính chắn gió và kính cửa xe vào mask của `car`. |
| **3. `cp5_occlusion`:** Xe ô tô bị cột đèn/thân cây chắn ngang chia thành 2 mảng nhìn thấy tách rời nhau | Tạo thành 2 object `car` riêng lẻ hay gom chung thành 1 instance | Quy tắc Occlusion trong Instance: cùng một thực thể dù bị chia cắt về mặt thị giác vẫn thuộc về 1 instance duy nhất | Quyết định gán cả 2 mảng nhìn thấy vào cùng 1 Object ID trong CVAT, không tách thành 2 đối tượng rời rạc. |
