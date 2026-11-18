# Báo cáo QC tự động (nháp)

> **Gợi ý** do script tính; bạn xác nhận. Script chỉ đếm theo phiếu kiểm bằng mắt người.

## Mẫu
- 5 ảnh (0 ca khó + 5 ngẫu nhiên), 115 đối tượng.
- taxonomy v1.0

## Số đo
| Chỉ số | Giá trị | Mẫu số |
|---|---|---|
| Lỗi **theo đối tượng** | 5/115 = 4.3% | 115 đối tượng trong mẫu |
| **Ảnh có lỗi** (dùng cho ngưỡng) | 1/5 = 20.0% | 5 ảnh trong mẫu |
| Máy: accepted / edited / deleted | 443/520 = 85.2% / 28/520 = 5.4% / 46/520 = 8.8% | 520 box máy |
| Máy: added | 93/564 = 16.5% | 564 đối tượng sau duyệt |
| Bắt lỗi bẫy | 12/12 = 100.0% | 12 bẫy |

## Top 3 loại lỗi
| Loại lỗi | Số lỗi | Ảnh ví dụ |
|---|---|---|
| bo_sot | 5 | img_024.jpg |

**Lỗi hệ thống**: 'bo_sot' lặp lại -> sửa quy tắc / recipe, không chỉ sửa từng ảnh.

## Giới hạn thống kê
- 1 ảnh lỗi trong 5 ảnh mẫu: cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là **62%**. Ngưỡng 2% không thể được **chứng minh** bằng mẫu nhỏ như vậy: hãy nói rõ giới hạn này trong báo cáo.

## Gợi ý quyết định
**REWORK**

- ảnh có lỗi 20.0% > 2%
- lỗi hệ thống: 'bo_sot' chiếm 5/5 lỗi

Cây quyết định: Hold (guideline chưa rõ / dữ liệu hỏng) -> Rework (> ngưỡng, lỗi lặp, bắt bẫy / nhất quán thấp) -> Release.
