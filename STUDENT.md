# Thông tin học viên

- Họ và tên: Bùi Sỹ Tùng
- MSHV: 2A202603323
- Lớp (L2 / L3 / khác): <<>>
- Tài khoản GitHub: BuffDarkAI
- Repo bài nộp (URL): https://github.com/BuffDarkAI/K5-L2b-Day08-buisytung_2A202603323-AI-Assisted-Labeling
- Commit chốt bài (hash): <<>>
- Máy: Windows 11, NVIDIA RTX 3050 Ti Laptop 4GB (CUDA 12.4)
- CVAT (Docker): <<phiên bản, ví dụ v2.81.0>>

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 85.2% / 5.4% / 8.8% (443 / 28 / 46, mẫu số: 520 box máy); added: 16.5% (93, mẫu số: 564 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 52/74 = 70.3% / 52/80 = 65.0% (80 đối tượng gold, 4 ảnh)
- Bắt bẫy: 12/12 = 100% sau 4 vòng duyệt; lần duyệt mù đầu tiên chỉ 5/12 = 41.7%
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 4.17 phút/ảnh (100 phút / 24 ảnh); cộng 20 phút duyệt lại bẫy thì 5.0 phút/ảnh
- Lỗi QC: 40.0% theo ảnh (2/5), 4.2% theo đối tượng (5/120); cận trên Wilson 95% của tỉ lệ ảnh lỗi: 77%
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 2/5 = 40% > 2% và có lỗi hệ thống 'thừa' (box `three_wheeler` trùng của YOLOE) chiếm 4/5 lỗi
