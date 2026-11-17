# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603267 - Bạch Sỹ Núi

> **Chỉ viết phần thay đổi so với v1.0** (không copy lại toàn bộ file). Chốt các câu hỏi mở ở mục 6 của v1.0 và bổ sung quy tắc mới rút ra từ lỗi của máy / bẫy lọt / box `can_review`. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp đúng một dòng trong `submission/qc/calibration_decisions.csv`.

---

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 1 | Xe đạp điện / xe máy điện nhỏ: `bicycle` hay `motorbike`? | Mọi phương tiện 2 bánh có động cơ (kể cả động cơ điện, e-bike) đều xếp vào lớp `motorbike`. Lớp `bicycle` chỉ dành cho xe đạp dùng sức người đạp thuần túy. | `img_018`, `img_024` | CAL-02 |
| 2 | Khối xe máy đỗ dày đặc che khuất nhau: từng xe hay `ignore_region`? | Xe nhìn thấy rõ $\ge 30\%$ diện tích thân xe thì gán riêng từng box. Cụm xe dày đặc $(\ge 5\text{ xe})$ chen chúc không thể tách rời biên giới thì dùng 1 box `ignore_region` bao quanh. Không dùng `ignore_region` đè lên xe đã vẽ rõ. | `img_005`, `img_028` | CAL-03 |

---

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| `img_002`, `img_013` | Xe ba bánh có mui / tuk-tuk / ba gác chở đồ | YOLO26 nhận nhầm thành `car` hoặc `truck` do tập nhãn COCO không có `three_wheeler` | Luôn đổi nhãn thành `three_wheeler`. Kết hợp text-prompt YOLOE để bắt ứng viên, con người review bắt buộc | Mục 2: Xe 3 bánh có động cơ hay không có động cơ, có mui hay không mui đều là `three_wheeler` (CAL-01) |
| `img_021`, `img_029` | Bẫy hộp lỏng (box bị vẽ thừa khoảng trống xung quanh xe/người) | Pre-label cố tình cài bẫy nới rộng viền hộp > 20px so với vật | Người duyệt bị lọt do thiên kiến tự động hóa; quy định siết chặt bước 4: phóng to ảnh kiểm tra 4 mép hộp | Mục 4.1: Mép hộp chữ nhật phải bám sát điểm nhô ra xa nhất của vật thể, sai số cho phép $\le 3\text{ px}$ (CAL-04) |

---

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên viết sẵn | — |
| v1.1 | Chốt phân loại xe đạp điện vào `motorbike`; chuẩn hóa quy tắc gán `ignore_region` cho cụm xe máy dày đặc; bổ sung quy tắc nhận diện xe ba bánh tránh nhầm với car/truck; siết tiêu chuẩn ôm sát mép của bounding box ($\le 3\text{ px}$). | CAL-01, CAL-02, CAL-03, CAL-04 |
