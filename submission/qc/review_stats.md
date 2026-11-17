# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 505; đối tượng sau duyệt: 499; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 460 | 460/505 = 91.1% | 505 box máy |
| edited (sửa) | 10 | 10/505 = 2.0% | 505 box máy |
| deleted (xoá) | 35 | 35/505 = 6.9% | 505 box máy |
| added (vẽ thêm) | 29 | 29/499 = 5.8% | 499 đối tượng sau duyệt |

Trong số deleted + added, **4 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 25.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 261 | 1 | 6 | 5 |
| bicycle | 6 | 1 | 3 | 3 |
| motorbike | 128 | 6 | 4 | 12 |
| car | 37 | 0 | 2 | 7 |
| bus | 3 | 0 | 0 | 0 |
| truck | 3 | 0 | 3 | 1 |
| three_wheeler | 22 | 2 | 17 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 87 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 56 | 19 | 31 | 74.7% (KTC 95%: 64–83%) | 64.4% (KTC 95%: 54–74%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 6/12 = 50.0% (KTC 95%: 25–75%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - wrong_class: 3/3
  - extra: 2/3
  - missing: 1/3
  - loose: 0/3
  - lọt: img_005.jpg (missing), img_029.jpg (loose), img_025.jpg (loose), img_021.jpg (loose), img_009.jpg (missing), img_035.jpg (extra)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
