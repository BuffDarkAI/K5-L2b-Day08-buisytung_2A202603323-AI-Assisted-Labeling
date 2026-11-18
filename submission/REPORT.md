# Báo cáo Day 08 — 2A202603323 - Bùi Sỹ Tùng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | mặc định `--conf 0.3` cho cả 6 lớp | `threshold_sweep.md`: không ngưỡng nào ≤ 0.5 đạt recall 90% (person tối đa 84%/37 vật, motorbike 63%/19, bicycle 67%/6); bus (1 vật) và truck (0 vật) quá ít để chọn. Chạy ứng viên ở conf 0.05 (`r05`) rồi lọc lúc merge. **Ghi chú:** sau khi sửa 4 box gold `person` → `car`, chạy lại sweep thì gợi ý thêm `car=0.5` (recall 90%, precision 69% trên 10 vật). Nhưng lô 24 ảnh đã được pre-label và duyệt với `car` ở 0.3 từ trước, nên mình giữ nguyên recipe cũ, không merge lại |
| three_wheeler | YOLOE-26s, prompt `three_wheeler` | **0.15** | `threshold_sweep.md`: recall 100%, precision 78% trên 7 vật gold (`r06`). Ở `r01`/`r02`, YOLO26 không có lớp ba bánh nên tuk-tuk bị gọi `car` |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --conf 0.3 --thr three_wheeler=0.15 --split batch,gold --out work/prelabel/merged_review/pre.coco.json` → 606 box trên 28 ảnh (YOLOE đứng trước nên được ưu tiên khi chồng box, `--iou 0.6` mặc định)
- **Phiên bản SAM bạn chọn** và lý do: **SAM 2.1 base** (`sam2.1_b.pt`). Viền mái, bánh và khung bám tốt hơn hẳn MobileSAM trên cùng click ở img_013; chạy trên GPU CUDA khoảng 1.7 s/ảnh (`r04`). MobileSAM nhẹ hơn nhưng viền lởm chởm, mask vỡ đôi (xem mục 5).
- Lỗi nào **không sửa được bằng đổi ngưỡng**:
  - YOLO26 không có lớp ba bánh, nên tuk-tuk bị gọi `car` → phải thêm YOLOE với prompt `three_wheeler`.
  - YOLOE lại sinh nhiều box `three_wheeler` chồng nhau trên cùng một xe (img_003: #10, #11, #24) → phải gộp tay khi duyệt (CAL-01). Đổi ngưỡng không gỡ được box trùng vì box trùng có conf 0.80–0.90.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng**:
  - img_003 #10 `three_wheeler` conf 0.80 là box trùng của #24 (conf 0.90): điểm cao mà sai.
  - img_031 #496 `person` conf 0.34 là cái mũ treo trên giá: sai.
  - img_031 #493 `bicycle` conf 0.45 đúng dù điểm thấp.
- Giới hạn: ngưỡng 0.15 cho `three_wheeler` được chọn và chấm trên **cùng 4 ảnh gold (7 vật)**, nên recall 100% là lạc quan. Trên lô 24 ảnh, `three_wheeler` vẫn bị deleted 19/37 box máy = 51%, tức ngưỡng thấp đã kéo theo nhiều box sai.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 443 | 443/520 = 85.2% | 520 box máy |
| edited | 28 | 28/520 = 5.4% | 520 box máy |
| deleted | 46 | 46/520 = 8.8% | 520 box máy |
| added | 93 | 93/564 = 16.5% | 564 đối tượng sau duyệt |

(Nguồn `qc/review_stats.md`; đã loại 26 đối tượng quanh bẫy. 9 ca là đổi lớp, nên số vật máy sót thật ≈ 93 − 9 = 84.)

- Lớp nào máy tệ nhất:
  - **`three_wheeler`**: deleted 19/37 box máy = 51%, edited 7/37 = 19%. YOLOE ở ngưỡng 0.15 sinh box trùng và box ở chỗ không có xe (img_003 #11, #14).
  - **`truck`**: deleted 5/6 box máy = 83%, nhưng mẫu quá nhỏ để kết luận.
  - **`car`** bị **sót** nhiều nhất: added 20, trên 73 car sau duyệt = 27%.
- So với gold: máy có precision **56/74 = 75.7%** (KTC 95%: 65–84%) và recall **56/80 = 70.0%** (KTC 95%: 59–79%), mẫu số là 80 đối tượng gold trên 4 ảnh. Đây là số **sau khi sửa gold** (xem mục 8). Trước khi sửa: 52/74 = 70.3% và 52/80 = 65.0%.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy:
  - **Lần duyệt đầu (mù): 5/12 = 41.7%**. Lọt 7 bẫy: missing ×2 (img_035, img_022), extra ×2 (img_031, img_005), loose ×2 (img_029, img_018), wrong_class ×1 (img_027).
  - Sau đó mình duyệt lại 3 vòng: 8/12, rồi 10/12, rồi **12/12**. Từ vòng 2 mình đã biết tên ảnh bẫy bị lọt, và ở 2 vòng cuối đã dùng tọa độ trong `traps_key.json`. Vì vậy chỉ con số **5/12** phản ánh đúng chất lượng duyệt.
  - Vì sao lọt: mình nhìn nhãn máy trước, không quét ảnh trước theo checklist. Box extra trông "hợp lý" nên giữ lại. Box loose lệch ít nên không để ý. Người nhỏ đứng sát người lớn (img_022) thì bị bỏ qua.
- Accepted cao (85.2%) mà bắt bẫy lần đầu thấp (41.7%) cho thấy **thiên kiến tự động hóa**: mình mặc định tin box máy. QC sau đó còn tìm ra 3 box `three_wheeler` trùng ở img_003 và 1 "person" là cái mũ ở img_031. Các box này vẫn được tính là accepted, tức là accepted 85.2% đang **thổi phồng** chất lượng máy.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay **15/4 = 3.75 phút/ảnh** vs duyệt **100/24 = 4.17 phút/ảnh**. Tính theo đối tượng thì gần như bằng nhau: gán tay 15/80 = 0.19 phút/đối tượng, duyệt 100/564 = 0.18 phút/đối tượng. Như vậy **pre-label không giúp nhanh hơn rõ rệt**. Nếu cộng thêm 20 phút cho 3 vòng duyệt lại để bắt đủ 12/12 bẫy (`trap_recheck`), tổng thời gian duyệt là 120/24 = **5.0 phút/ảnh** (0.21 phút/đối tượng), tức còn **chậm hơn** gán tay. Thiên lệch:
  - Mình gán gold trước, nên lúc duyệt đã quen CVAT và quen kiểu cảnh phố; điều này làm thời gian duyệt có lợi hơn thực tế.
  - Ngược lại, ảnh batch đông hơn (564/24 ≈ 23.5 đối tượng/ảnh, so với 80/4 = 20 ở gold). Thời gian duyệt còn gồm cả việc xoá box trùng `three_wheeler` và các box rác.
  - Chỉ có 4 ảnh gold, nên con số 3.75 phút/ảnh rất kém chắc chắn.
- Kết luận: **có, nhưng chỉ cho `person` và `motorbike`**. Lợi ích nằm ở việc đỡ phải vẽ box hơn là ở thời gian: thời gian/đối tượng gần như không đổi (0.18 so với 0.19 phút), và nếu tính cả vòng duyệt lại bẫy thì duyệt còn chậm hơn (0.21 phút). Hai lớp này có accepted 252/270 = 93.3% và 118/138 = 85.5% (mẫu số là số box máy của lớp đó, theo bảng theo lớp trong `review_stats.md`). **Không** nên tin `three_wheeler` (deleted 19/37 box máy = 51%) và `car` (added 20/73 = 27% số car sau duyệt bị máy sót). Precision/recall của máy so với gold (75.7% / 70.0% trên 80 vật) cho thấy pre-label chỉ là điểm khởi đầu, không thay được bước duyệt.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. **Quét ảnh trước, xem nhãn sau**: ẩn hết box (nút con mắt), đếm nhẩm số người/xe, rồi mới bật nhãn máy lên và so số đếm với số box. Kiểm bằng số: bắt bẫy ở **lần duyệt mù đầu tiên** phải ≥ 80% (lần này chỉ đạt 5/12 = 41.7%).
  2. **Không tin box nào chỉ vì conf cao**: với `three_wheeler`, xem từng box và xoá box trùng IoU ≥ 0.5 trên cùng một xe (CAL-01). Kiểm bằng số: sau duyệt, số cặp box cùng lớp có IoU ≥ 0.5 phải bằng 0 (img_003 có box trùng ở conf 0.80).
  3. **Có người khác kiểm QC**: mẫu QC phải được người không duyệt lô đó soi lại (lần này mình tự kiểm). Kiểm được: cột `reviewer` trong `qc_review.csv` khác người duyệt, và lỗi theo ảnh của mẫu ≤ 2% thì mới Release.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 base (`sam2.1_b.pt`) | 2 pos + 1 neg | **Có**: dính gần hết người lái, cả bé áo hồng và khách ngồi sau khoang xe | 1 điểm âm trên lưng chỉ khoét vài đốm; khoang hở bị lấp kín (45.4% ảnh) | `tuktuk_img013_sam2.jpg` |
| img_013 | MobileSAM (`mobile_sam.pt`) | 2 pos + 1 neg (cùng tọa độ) | Ít hơn: thân người lái và khách phía sau phần lớn bị loại; còn dính mặt, chân người lái | Mask vỡ thành 2 polygon, viền sàn/bậc lên xuống lởm chởm (35.2% ảnh) | `tuktuk_img013_sam1.jpg` |
| img_013 | SAM 2.1 base + `--box` | 3 pos + 5 neg | **Không**: người lái (đầu, thân, chân), bé áo hồng, khách sau đều tách ra | Còn vài đốm đen quanh vùng chân người lái / đầu xe; tay vịn mảnh còn hở (37.7% ảnh) | `tuktuk_img013_sam2_moreneg.jpg` |
| img_004 | SAM 2.1 base | 2 pos + 1 neg | Không (không có người trên xe) | Mask loang xuống mặt đường dưới bánh trước (điểm dương 700,560 nằm sát mặt đất) | `tuktuk_img004_sam2.jpg` |
| img_004 | SAM 2.1 base + `--box` | 2 pos + 2 neg | Không | Sạch, bám đúng thân, mái, bánh; hết vệt loang | `tuktuk_img004_sam2_box.jpg` |
| img_017 | SAM 2.1 base | 2 pos + 1 neg | Không (xe trống) | Bám thân/mái/bánh tốt; điểm âm mẫu (645,355) rơi lên mái xe nhưng không làm hụt mái | `tuktuk_img017_sam2.jpg` |

**SAM 2.1 vs MobileSAM (cùng click trên img_013):** với 1 điểm âm, MobileSAM loại người lái tốt hơn nhưng viền kém và mask bị vỡ đôi; SAM 2.1 bám viền mái vòm, bánh và hoa văn khung tốt hơn hẳn nhưng "nuốt" cả người trong khoang. 1 điểm âm **không đủ** tách người lái; cần 1 điểm âm cho mỗi phần (đầu, thân, chân) cộng `--box` giới hạn vùng.

- Đối chiếu guideline mục 4.2: quy tắc 2 (bỏ lỗ thủng lớn) giúp nhận ra lần chạy đầu ở img_013 sai, vì khoang hở nhìn xuyên thấy người phía sau bị lấp kín; quy tắc 3 (bóng đổ không thuộc vật) giúp nhận ra vệt loang ở img_004. **Thiếu:** 4.2 chưa nói rõ người **ngồi bên trong** xe (người lái, khách) có thuộc mask xe không; quy tắc 4 ("vùng chạm thuộc vật phía trước") mơ hồ khi người lái vừa ở trước vừa ở trong khung xe. Đề xuất v1.1: ghi rõ người lái, hành khách và hàng hoá nhô ra là vật riêng, không thuộc mask phương tiện.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: chưa thử.

## 6. Calibration (Part 6)

- Số quyết định: **6** (`calibration_decisions.csv`: CAL-01 → CAL-06). Nguồn gồm 2 `loi_may`, 3 `cau_hoi_mo`, 1 `bay_lot`. Quan trọng nhất là **CAL-01**: 1 vật = 1 box, gộp box `three_wheeler` trùng. Đây là lỗi hệ thống của YOLOE, và cũng là lỗi QC gặp nhiều nhất.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  - Gộp box cùng lớp chồng nhau (IoU ≥ 0.5) trên một vật.
  - `person` không gồm quần áo hay mũ treo trong cửa hàng.
  - Người ngồi trong xe không thuộc mask xe.
  - Xe máy đỗ dày gần camera thì vẽ từng xe.
  - Đã chốt câu mở 2, 4, 6. Câu 1, 3, 5, 7 giữ nguyên v1.0 vì chưa có ca trong lô.
- Nguyên nhân gốc:
  - Chủ yếu do **máy** (YOLOE sinh box trùng, YOLO nhận nhầm mũ thành người) cộng với **người duyệt tin máy** (để lọt các box đó).
  - Một phần do **guideline mơ hồ**: v1.0 chưa có quy tắc cho box trùng và người trong xe.

## 7. QC và quyết định (Part 6)

### Vòng 1 (phiếu lưu ở `qc/qc_review_round1.csv`, báo cáo ở `qc/qc_report_round1.md`)

- Mẫu: **5 ảnh** (0 ảnh `can_review` + 5 ngẫu nhiên, seed 42: img_003, img_006, img_014, img_031, img_035), **120 đối tượng**. Người kiểm là chính mình (**hạn chế**: mình vừa là người duyệt, dễ mù với chính lỗi của mình).
- Lỗi theo **đối tượng**: **5/120 = 4.2%**. Theo **ảnh**: **2/5 = 40.0%**, so với ngưỡng Release ≤ 2%.
- Top lỗi:
  - **thừa** 4 lỗi: img_003 có #10, #11 `three_wheeler` trùng và #14 ở chỗ không có xe; img_031 có #496 "person" thực ra là cái mũ.
  - **sai lớp** 1 lỗi: img_003 #17 là box 200×161 ôm xe máy nhưng gán `person`, cần xác nhận lại.
  - Không thấy lỗi bỏ sót, sai vị trí/kích thước hay sai mask ở mức nhìn overlay.
- **Giới hạn mẫu nhỏ**: với 2 ảnh lỗi trên 5, cận trên Wilson 95% của tỉ lệ ảnh lỗi thật là **77%**. Kể cả mẫu sạch 0/5 thì cận trên vẫn khoảng 43%. Mẫu 5 ảnh chỉ phát hiện được lỗi lớn, **không thể chứng minh** lô đạt ≤ 2%.
- Script kiểm được / không kiểm được:
  - **Kiểm được**: định dạng COCO, đủ 24 ảnh, box hợp lệ, cảnh báo box < 20 px, tỉ lệ bắt bẫy, accepted/edited/deleted/added.
  - **Không kiểm được**: box có ôm sát vật thật không; vật mà cả máy và mình cùng bỏ sót; box trùng hay box gán cho vật không phải người (mũ). Box trùng thậm chí còn được tính là "accepted". Mình có thực sự quét ảnh trước khi nhìn nhãn máy hay không cũng không script nào kiểm được.
  - Tất cả những thứ không kiểm được ở trên chỉ phát hiện được bằng mắt.

**Quyết định vòng 1: Rework**, ngày 2026-11-18. Lý do, đi theo thứ tự Hold → Rework → Release:
- **Hold?** Không. Câu mở còn lại (1, 3, 5, 7) không có ca trong lô nên không làm đổi nhãn nào; dữ liệu ảnh không hỏng.
- **Rework?** Có. Ảnh lỗi 2/5 = 40% > 2%, và có **lỗi hệ thống**: "thừa" chiếm 4/5 lỗi, lặp lại ở 2/5 ảnh. Bắt bẫy cuối đạt 12/12, nhưng lần duyệt mù chỉ 5/12.

Việc cần làm tiếp:
1. Trên CVAT, xoá img_003 #10, #11, #14 và img_031 #496; kiểm lại img_003 #17.
2. Rà **toàn bộ 26 box `three_wheeler`** trong lô để tìm box trùng (theo CAL-01), vì đây là lỗi hệ thống chứ không chỉ ở 1 ảnh.
3. Export các ảnh đã sửa thành `submission/labels/qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force`, rồi ghi vòng QC 2 vào báo cáo.

### Vòng 2 (sau khi sửa: `labels/qc_fixed.json`, phiếu `qc/qc_review.csv`, báo cáo `qc/qc_report_auto.md`)

- **Đã sửa trên CVAT** (`qc_fixed.json` gồm 4 ảnh, thay bản cũ trong `release/final_det.coco.json`, xem `release_manifest.json`):
  - img_003: xoá 3 box `three_wheeler` trùng hoặc sai chỗ (#10, #11, #14 của vòng 1), 1 box `three_wheeler` sai ở người đi xe máy bên trái, 2 box `person` đè lên chiếc xe máy bên phải (#17 và một box trùng của nó), 1 `ignore_region` thừa ở mép phải (chỉ có sạp hàng); kéo lại box xe máy đỗ ở giữa.
  - img_031: xoá #496 (cái mũ).
  - img_004: kéo dài box người bên trái xuống tới chân (cao 60 → 105 px).
  - img_027: kéo dài box người ngồi sau (cao 31 → 80 px), xoá 1 box `person` chỉ ôm nửa trên của người đội mũ vàng (trùng với box cả người).
  - Release sau sửa: 24 ảnh, **572 đối tượng** (trước: 581); `three_wheeler` còn 22 (trước: 26).
- Mẫu: `qc_sample.py --seed 42 --force` cho lại **đúng 5 ảnh cũ**, **113 đối tượng**. Kiểm lại bằng ảnh có ID trong `work/qc/review/`. Người kiểm: mình, kèm trợ lý AI (Claude) soi overlay và cắt phóng to từng vùng nghi ngờ; mình xác nhận lại. Tức là vẫn **chưa có người thứ hai độc lập**.
- Lỗi theo **đối tượng**: **2/113 = 1.8%** (vòng 1: 5/120 = 4.2%). Theo **ảnh**: **2/5 = 40.0%** (không đổi), so với ngưỡng ≤ 2%.
  - img_014 #162/#163 `motorbike` gần trùng (IoU 0.87) trên cùng đống xe đỗ: **thừa**.
  - img_031 #487 `person`: box dừng ở ngang hông trong khi chân vẫn nhìn thấy qua nan bánh xe đạp: **sai vị trí/kích thước**. Vòng 1 đã bỏ sót lỗi này, đúng kiểu mù với lỗi của chính mình.
  - Không còn lỗi hệ thống: "thừa `three_wheeler`" không còn xuất hiện; 2 lỗi còn lại khác loại nhau.
- Cận trên Wilson 95% của tỉ lệ ảnh lỗi vẫn là **77%** (2/5). Mẫu 5 ảnh vẫn không chứng minh được lô đạt ≤ 2%.

**Quyết định vòng 2: vẫn Rework**, ngày 2026-11-18.
- **Hold?** Không (như vòng 1).
- **Rework?** Có: ảnh lỗi 2/5 = 40% > 2%. Lỗi theo đối tượng đã giảm còn 1.8% và hết lỗi hệ thống, nhưng ngưỡng Release tính theo ảnh.
- Việc tiếp theo: sửa img_014 #162/#163 và img_031 #487 trên CVAT, thêm vào `qc_fixed.json`; rà các ảnh còn lại của lô theo 2 kiểu lỗi này (box trùng ở cụm xe đỗ; box người bị cắt khi chân nhìn thấy qua bánh xe); rồi lấy mẫu QC **mới** (seed khác) để không chấm lại đúng những ảnh đã sửa.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? **Có, một phần.** Lúc gán gold mình không nhìn pre-label. Nhưng **sau** khi đã có số so sánh với máy, mình sửa 4 box gold bị gán nhầm lớp `person` cho xe hơi (img_012 ×3, img_020 ×1). Nguyên nhân là CVAT để sẵn nhãn mặc định `person` khi vẽ box. Các box này được phát hiện nhờ so với máy, và đã nhìn ảnh để xác nhận là xe hơi. Mình **không** đổi box gold nào chỉ vì khác máy: các ca lấp lửng (img_001 `car`/`truck`, `motorbike`/`bicycle`) giữ nguyên. Gold vẫn còn 20 box cao < 20 px (img_001, img_036) chưa gom vào `ignore_region`. Sửa gold làm precision/recall của máy tăng từ 70.3%/65.0% lên 75.7%/70.0%.
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
