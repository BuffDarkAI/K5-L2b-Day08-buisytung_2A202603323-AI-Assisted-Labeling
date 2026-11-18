# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 520; đối tượng sau duyệt: 564; ngưỡng accepted: IoU >= 0.9
- Đã **loại 26 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 443 | 443/520 = 85.2% | 520 box máy |
| edited (sửa) | 28 | 28/520 = 5.4% | 520 box máy |
| deleted (xoá) | 46 | 46/520 = 8.8% | 520 box máy |
| added (vẽ thêm) | 93 | 93/564 = 16.5% | 564 đối tượng sau duyệt |

Trong số deleted + added, **9 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 84.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 252 | 10 | 8 | 30 |
| bicycle | 6 | 0 | 2 | 5 |
| motorbike | 118 | 11 | 9 | 25 |
| car | 53 | 0 | 2 | 20 |
| bus | 2 | 0 | 1 | 4 |
| truck | 1 | 0 | 5 | 3 |
| three_wheeler | 11 | 7 | 19 | 6 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 80 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 56 | 18 | 24 | 75.7% (KTC 95%: 65–84%) | 70.0% (KTC 95%: 59–79%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 12/12 = 100.0% (KTC 95%: 76–100%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - wrong_class: 3/3
  - missing: 3/3
  - extra: 3/3

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
