# Thông tin học viên

- Họ và tên: Bạch Sỹ Núi
- MSHV: 2A202603267
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: nuibachcode
- Repo bài nộp (URL): https://github.com/nuibachcode/K5-L2B-Day08-BachSyNui-2A202603267-AI-Assisted-Labeling
- Commit chốt bài (hash): 7cc82d9
- Máy: Windows 11, không có GPU (CPU)
- CVAT (Docker): v2.74.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 460/505 = 91.1% / 10/505 = 2.0% / 35/505 = 6.9% (mẫu số: 505 box máy); added: 29/499 = 5.8% (mẫu số: 499 đối tượng sau duyệt)
- So với gold: precision / recall của **máy**: 74.7% (56/75) / 64.4% (56/87) (mẫu số: 87 đối tượng gold)
- Bắt bẫy: 6/12 = 50.0%
- Thời gian: gán tay 6.0 phút/ảnh (24 phút/4 ảnh); duyệt có pre-label 2.5 phút/ảnh (60 phút/24 ảnh)
- Lỗi QC: 20.0% theo ảnh (1/5), 1.0% theo đối tượng (1/103)
- **Quyết định:** REWORK — lý do một dòng: Tỷ lệ ảnh có lỗi ở mẫu QC là 20.0% (> 2%) và tỷ lệ bắt bẫy đạt 50.0% (< 80%), cần rà soát lại các ảnh bị lọt bẫy theo đúng cây quyết định.
