# Báo cáo Day 08 — 2A202603267 - Bạch Sỹ Núi

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

---

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / motorbike | YOLO26m | 0.30 (mặc định) | Tại `r01` (conf 0.25) bắt được nhiều xe/người ở xa nhưng có box nhiễu; tại `r02` (conf 0.50) recall giảm mạnh (xe máy giảm từ 55 xuống 32 box). Ngưỡng mặc định 0.30 là điểm cân bằng tốt nhất giữa precision và recall. |
| bicycle | YOLO26m | 0.20 | Từ `qc/threshold_sweep.md`: tại ngưỡng 0.20 đạt Recall 100% ($\ge 90\%$), Precision 86% trên 6 vật mẫu gold. |
| car | YOLO26m | 0.50 | Từ `qc/threshold_sweep.md`: tại ngưỡng 0.50 đạt Recall 90% ($\ge 90\%$), Precision 69% trên 10 vật mẫu gold (ô tô to, rõ, ít bị che khuất). |
| bus / truck | YOLO26m | 0.30 (mặc định) | Trong 4 ảnh gold có ít hơn 5 vật (bus có 2, truck có 0) nên theo nguyên tắc thống kê không chọn ngưỡng riêng mà dùng mặc định 0.30. |
| three_wheeler | YOLOE-26s | 0.15 | Tại `r03`: YOLOE dùng open-vocabulary text prompt (`tuk-tuk, cyclo, cargo tricycle`) bắt được xe ba bánh mà YOLO26 bỏ sót. Từ `threshold_sweep.md`: ngưỡng 0.15 đạt Recall 100%, Precision 78%. |

- Lệnh `merge_prelabels.py` đã dùng:
  ```bash
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "bicycle=0.2,car=0.5,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
- **Phiên bản SAM bạn chọn:** Chọn **SAM 2.1 base** (`sam2.1_b.pt`). Lý do: Kiến trúc SAM 2.1 cho chất lượng mặt nạ phân đoạn sắc nét, bám sát mép viền chi tiết cơ thể người và khung xe tốt hơn rõ rệt so với SAM 1, không đòi hỏi xin quyền phê duyệt (gated) như SAM 3, và hoạt động hoàn toàn ổn định trên máy CPU RAM 16GB (khoảng ~7.4s/ảnh).
- **Lỗi không sửa được bằng đổi ngưỡng:** Xe ba bánh (tuk-tuk, xe ba gác, xích lô) bị YOLO26 nhận nhầm thành `car` hoặc `truck`. Nguyên nhân do YOLO26 huấn luyện trên tập COCO 80 lớp cố định (closed-set) không có khái niệm `three_wheeler`. Không ngưỡng nào có thể sửa được lỗi này. Cách giải quyết: Sử dụng mô hình Open-Vocabulary (YOLOE) với prompt văn bản riêng, sau đó con người review để chỉnh sửa nhãn.
- **Box điểm cao mà sai / điểm thấp mà đúng:**
  - Điểm cao mà sai: Ở `img_002.jpg`, YOLO26 gán nhãn `car` cho xe tuk-tuk với độ tin cậy rất cao (> 0.70).
  - Điểm thấp mà đúng: Ở `img_011.jpg`, xe đạp ở xa và xe ba bánh ở mép ảnh có conf chỉ 0.18 - 0.22 nhưng đúng là vật thể thật.
- **Giới hạn:** Chọn ngưỡng dựa trên 4 ảnh gold (in-sample) nên chỉ số precision/recall của máy trên gold mang tính lạc quan; mẫu 4 ảnh quá nhỏ nên chỉ xem đây là ngưỡng ước lượng khởi đầu.

---

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 460 | 460/505 = 91.1% | 505 box máy |
| edited (sửa kích thước/vị trí) | 10 | 10/505 = 2.0% | 505 box máy |
| deleted (xoá box) | 35 | 35/505 = 6.9% | 505 box máy |
| added (vẽ thêm) | 29 | 29/499 = 5.8% | 499 đối tượng sau duyệt |

- **Lớp máy tệ nhất:**
  - `three_wheeler`: Tỷ lệ xóa cao (xóa 17/52 box máy) do YOLOE ở ngưỡng nhạy sinh một số box thừa bao trùm vào khoảng trống hoặc xe máy bên cạnh.
  - `motorbike`: Tỷ lệ bỏ sót cao nhất (phải vẽ thêm 12 box) ở các ảnh có dòng xe máy đông đúc bị che khuất (`img_005.jpg`, `img_018.jpg`).
- **So với gold:** Precision của máy = 74.7% (56/75 TP/(TP+FP)), Recall của máy = 64.4% (56/87 TP/(TP+FN), mẫu số: 87 đối tượng trong 4 ảnh gold).

---

## 3. Thiên kiến tự động hóa và ảnh bẫy

- **Bắt bẫy:** **6/12 = 50.0%** (KTC 95%: 25–75%).
  - Bắt trọn: `wrong_class` 3/3 (100% phát hiện khi máy gán sai loại xe).
  - Bắt được: `extra` 2/3 (phát hiện box thừa).
  - Bỏ sót: `missing` 2/3 (lọt ở `img_005.jpg`, `img_009.jpg`), `loose` 3/3 (lọt toàn bộ 3 bẫy hộp lỏng ở `img_021.jpg`, `img_025.jpg`, `img_029.jpg`), `extra` 1/3 (lọt ở `img_035.jpg`).
- **Nguyên nhân lọt:** Đây là biểu hiện kinh điển của **Thiên kiến tự động hóa (Automation Bias)**. Khi mắt người nhìn thấy AI đã vẽ sẵn một box bao quanh vật thể, người duyệt có xu hướng dễ dãi "chấp nhận" (nhìn thấy có box là cho qua) mà bỏ qua bước phóng to kiểm tra mép hộp (dẫn đến lọt toàn bộ 3 bẫy loose). Ở các ảnh mật độ xe quá dày như `img_005.jpg`, mắt bị quá tải nên không phát hiện ra vật thể chưa có box (bẫy missing).
- **Ý nghĩa:** Tỷ lệ `accepted` rất cao (91.1%) nhưng bắt bẫy chỉ đạt 50.0% cảnh báo rằng người duyệt đã tin tưởng máy quá mức và chưa tuân thủ chặt chẽ Bước 1 của checklist review (ẩn nhãn máy để tự quét trước).

---

## 4. Máy có đáng dùng không?

- **Thời gian (từ `time_log.csv`):**
  - Gán tay thuần túy (gold): 24 phút / 4 ảnh = **6.0 phút/ảnh**.
  - Duyệt có pre-label: 60 phút / 24 ảnh = **2.5 phút/ảnh**.
  - Tiết kiệm: Khoảng **58% thời gian** gán nhãn so với vẽ tay từ đầu.
  - Thiên lệch: Người duyệt làm bài gold trước nên khi sang duyệt batch đã hiểu rõ bối cảnh và quy tắc hơn.
- **Kết luận:** **CÓ ĐÁNG DÙNG**, đặc biệt hiệu quả vượt trội cho các lớp phổ biến như `person` (accepted 261/268 = 97.4%) và `car` (accepted 37/39 = 94.9%). Tuy nhiên, với các lớp đặc thù (`three_wheeler`) và cảnh ùn tắc giao thông, con người bắt buộc phải là chốt chặn kiểm soát.
- **Ba quy tắc giảm thiên kiến:**
  1. *Quy tắc Blind Scan (Quét mù ban đầu):* Luôn tắt hiển thị nhãn máy trong 15-20 giây đầu tiên để mắt tự quét tìm các vật thể bị bỏ sót trước khi nhìn thấy gợi ý của AI.
  2. *Quy tắc Kiểm tra mép đối xứng:* Với mỗi box của AI, bắt buộc kiểm tra xem cạnh đáy và 2 cạnh bên có ôm sát bánh xe và thân người không (sai số không quá 3px).
  3. *Quy tắc Ngắt quãng thị giác (Pomodoro):* Duyệt tối đa 8 ảnh phải rời mắt khỏi màn hình 2 phút để tránh hiện tượng mệt mỏi nhận thức dẫn đến lướt qua bẫy.

---

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click / prompt | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| `img_013.jpg` | SAM 2.1 base | 2 điểm dương + 1 điểm âm | Không gồm người lái (người lái tách riêng) | Ban đầu mask hơi lan sang kính chắn gió của xe bên cạnh, sau khi thêm 1 click âm đã ôm gọn thân xe | `masks/tuktuk_img013.jpg` |
| `img_002.jpg` | SAM 2.1 base | 1 điểm dương + Bounding box | Mask ôm trọn thân xe tuk-tuk và mui xe | Phía dưới bánh xe hơi bị lem nhẹ vào bóng đổ trên mặt đường | `masks/tuktuk_img002.jpg` |
| `img_032.jpg` | SAM 2.1 base | 1 điểm dương + Bounding box | Bao gồm cả thùng hàng phía sau xe ba gác | Phần khung sắt thanh mảnh phía trước xe bị đứt nét nhẹ | `masks/tuktuk_img032.jpg` |

- **Đối chiếu guideline mục 4.2:** Quy tắc loại trừ bóng đổ và phân tách người lái xe ra khỏi phương tiện rất rõ ràng và hữu ích. Cần bổ sung quy tắc hướng dẫn xử lý phần nan hoa bánh xe và các chi tiết dạng lưới kim loại mỏng.

---

## 6. Calibration (Part 6)

- **Số quyết định:** 4 quyết định trong `submission/qc/calibration_decisions.csv`.
- **Quan trọng nhất:**
  - `CAL-01`: Chuẩn hóa mọi phương tiện 3 bánh (tuk-tuk, ba gác, xích lô) đều phải gán `three_wheeler`, không gán nhầm sang car/truck.
  - `CAL-02`: Chốt câu hỏi mở: Mọi phương tiện 2 bánh có động cơ điện (e-bike) đều xếp vào `motorbike`.
- **Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:** Đã cập nhật mục 2 (phương tiện 3 bánh), mục 3.1 (xe đạp điện), và mục 4.1 (tiêu chuẩn mép hộp $\le 3\text{px}$).
- **Nguyên nhân gốc:** Kết hợp giữa sự thiếu sót của guideline v1.0 (chưa quy định xe đạp điện) và giới hạn từ điển đóng của mô hình AI (YOLO26).

---

## 7. QC và quyết định (Part 6)

- **Mẫu:** 5 ảnh ngẫu nhiên (`img_003.jpg`, `img_006.jpg`, `img_014.jpg`, `img_031.jpg`, `img_035.jpg`), 103 đối tượng, seed 42. Người kiểm: Bạch Sỹ Núi (hạn chế: tự kiểm tra lại bài của chính mình).
- **Lỗi theo đối tượng:** 1/103 = **1.0%** · **Lỗi theo ảnh:** 1/5 = **20.0%** (ngưỡng Release $\le 2\%$).
- **Top lỗi:** Thừa (`thua`, 1 lỗi tại `img_035.jpg` do bẫy extra trên lề đường bị lọt).
- **Giới hạn mẫu nhỏ:** Với 1 ảnh lỗi trong 5 ảnh, cận trên khoảng tin cậy 95% (Wilson) lên tới **62%**. Mẫu 5 ảnh là quá nhỏ để có thể chứng minh về mặt thống kê rằng tỷ lệ lỗi thực tế đạt dưới 2%.
- **Script kiểm được / không kiểm được:** Script chỉ kiểm tra được sự tồn tại của file, tính hợp lệ của định dạng JSON và đếm số lượng lỗi theo phiếu; script không thể tự đánh giá hộp có ôm sát vật thật hay không nếu không có mắt người đối chiếu.

**Quyết định: REWORK**, ngày 2026-11-17.
- **Lý do:** Đi theo cây quyết định (Hold $\rightarrow$ Rework $\rightarrow$ Release):
  1. Guideline đã rõ ràng và không có ảnh hỏng $\rightarrow$ Không thuộc diện HOLD.
  2. Tỷ lệ ảnh có lỗi ở mẫu QC là 20.0% (vượt quá ngưỡng cho phép 2.0%) VÀ tỷ lệ bắt bẫy chỉ đạt 50.0% (dưới ngưỡng yêu cầu 80.0%) $\rightarrow$ Bắt buộc ra quyết định **REWORK**.
- **Việc cần làm tiếp:** Rà soát lại 6 ảnh bị lọt bẫy (đặc biệt là 3 ảnh có hộp lỏng `img_021`, `img_025`, `img_029` và 1 ảnh có box thừa `img_035`), siết chặt quy trình review trước khi đóng gói Release chính thức.

---

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Sử dụng trợ lý AI Antigravity để hỗ trợ chạy script terminal, chẩn đoán lỗi môi trường Docker và tổng hợp số liệu thống kê; toàn bộ việc vẽ 4 ảnh gold và duyệt 24 ảnh trên CVAT do học viên tự thực hiện 100%.
- **Gold có bị nhiễm không:** **HOÀN TOÀN KHÔNG**. 4 ảnh gold được tự tay vẽ từ đầu ở Part 1 trước khi chạy bất kỳ model AI nào.
- **Máy / GPU / CVAT dùng:** Windows 11, chạy hoàn toàn bằng CPU (16 GB RAM), Docker CVAT v2.74.1.
- **Dùng cache:** **KHÔNG**, tự chạy toàn bộ các mô hình trên máy thật.
- **Một điều bạn làm khác lần sau:** Sẽ phóng to từng vật thể để kiểm tra mép hộp kỹ càng hơn, tránh chủ quan với các hộp đã được AI vẽ sẵn dẫn đến lọt bẫy loose.
