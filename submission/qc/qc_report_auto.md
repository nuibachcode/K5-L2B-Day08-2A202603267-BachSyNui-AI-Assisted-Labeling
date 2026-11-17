# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 103 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 1/103 = 1.0% | 103 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 1/5 = 20.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 460/505 = 91.1% / 10/505 = 2.0% / 35/505 = 6.9% | 505 box máy |
| Máy: added | 29/499 = 5.8% | 499 đối tượng sau duyệt |
| Bắt lỗi bẫy | 6/12 = 50.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| thua | 1 | img_035.jpg |

## Giới hạn thống kê
- 1 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **62%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 20.0% > 2%
- bắt bẫy 50% < 80% -> lô duyệt chưa đáng tin

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
