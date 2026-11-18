# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603323 - Bùi Sỹ Tùng

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày đặc gần camera: từng xe hay `ignore_region`? | Gần camera, mỗi xe còn tách được và box cao ≥ 20 px → vẽ **từng xe**. Chỉ dùng `ignore_region` khi không tách được từng xe (thường ở xa) | img_006 | CAL-05 |
| 4 | Người ngồi trong xe ba bánh có mui che | Người là `person` riêng (theo mục 4.1.4). Với mask: người **không** thuộc mask `three_wheeler`. Box xe vẫn ôm cả xe | img_013, img_004, img_017 | CAL-04 |
| 6 | Box máy chồng nhau quá nhiều: gộp hay giữ? | **Gộp**: mỗi vật đúng 1 box. Chồng nhau chỉ hợp lệ giữa 2 vật khác nhau (người + xe) | img_003 | CAL-02 |

**Chưa chốt** (không có ca trong lô 24 ảnh để làm ảnh ví dụ, nên không đổi nhãn nào của lô này):
- Câu 1 (xe đạp điện): chưa gặp trong lô.
- Câu 3 (hàng nhô ra): vẫn theo mục 4.1.5 của v1.0.
- Câu 5 (kính xe): giữ mục 4.2.3 của v1.0 (kính thuộc vật).
- Câu 7 (conf tối thiểu): không đặt. Mọi box máy đều phải duyệt, vì box conf 0.90 vẫn có thể trùng (img_003 #24 / #10).

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003 | Một xe đẩy hàng có mái che phía sau | YOLOE sinh 3 box `three_wheeler` chồng nhau (#10, #11, #24) và 1 box `three_wheeler` ở chỗ không có xe (#14, conf 0.20). Toàn lô: `three_wheeler` deleted 19/37 box máy | Giữ 1 box ôm cả xe, xoá box trùng và box sai chỗ (CAL-01) | Mục 5: box cùng lớp chồng nhau (IoU ≥ 0.5) trên một vật → giữ 1 |
| img_031 | Mũ lưỡi trai treo trên giá quần áo | YOLO gán `person` conf 0.34 (#496) | Xoá (CAL-03) | Mục 2, `person` cột "Không gồm": thêm *quần áo, mũ treo / bày trong cửa hàng* |
| img_013 | Người lái ngồi trong tuk-tuk khi tách mask | SAM 2.1 với 1 điểm âm vẫn gộp người lái vào mask xe | Mỗi phần người (đầu, thân, chân) cần 1 điểm âm, kèm `--box` (CAL-04) | Mục 4.2: thêm quy tắc 5 "người trong xe không thuộc mask xe" |
| img_022 | Người nhỏ (~16×45 px) đứng sát người lớn | Máy bỏ sót (bẫy missing). Người duyệt cũng để lọt qua 3 lượt | Lỗi người duyệt, không đổi guideline (CAL-06) | Checklist duyệt: quét vùng sát vật lớn; box vẽ thêm phải ôm từ đầu đến chân |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu mở 2, 4, 6. Thêm quy tắc gộp box trùng, loại mũ/quần áo treo khỏi `person`, người trong xe không thuộc mask xe. Câu 1, 3, 5, 7 giữ nguyên v1.0 (chưa có ca) | CAL-01 → CAL-06 |
