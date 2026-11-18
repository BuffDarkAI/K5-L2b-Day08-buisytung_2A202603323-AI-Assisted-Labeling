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
- So với gold: precision / recall của **máy** 56/74 = 75.7% / 56/80 = 70.0% (80 đối tượng gold, 4 ảnh; sau khi sửa 4 box gold sai lớp)
- Bắt bẫy: 12/12 = 100% sau 4 vòng duyệt; lần duyệt mù đầu tiên chỉ 5/12 = 41.7%
- Thời gian: gán tay 3.75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 4.17 phút/ảnh (100 phút / 24 ảnh); cộng 20 phút duyệt lại bẫy thì 5.0 phút/ảnh
- Lỗi QC: vòng 1: 40.0% theo ảnh (2/5), 4.2% theo đối tượng (5/120). Vòng 2: 40.0% theo ảnh (2/5), 1.8% theo đối tượng (2/113). Vòng 3 (mẫu mới seed 43, 6 ảnh đã sửa): 20.0% theo ảnh (1/5), 4.3% theo đối tượng (5/115), cận trên Wilson 95% là 62%
- **Quyết định:** Rework — lý do một dòng: vòng 1 có ảnh lỗi 40% và lỗi hệ thống (box `three_wheeler` trùng). Vòng 3 vẫn có ảnh lỗi 1/5 = 20% > 2% (img_024 bỏ sót xe máy)
