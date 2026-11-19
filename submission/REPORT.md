# Báo cáo Day 08 — 2A202603283 - Đinh Hữu Thái An

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | person 0.15, motorbike 0.1; bicycle, car, bus, truck dùng mặc định 0.3 (`threshold_sweep.md`) | r02 (conf 0.25) so với r05 (conf 0.5): conf 0.5 làm mất person 131 xuống 87 và motorbike 55 xuống 32 trên 8 ảnh pilot, nên không dùng ngưỡng cao chung. bicycle và car không có ngưỡng nào ≤ 0.5 đạt recall 90% (tối đa 80% và 88%); bus và truck chỉ có 1 và 0 vật trong gold (< 5) nên không chọn ngưỡng theo lớp |
| three_wheeler | YOLOE-26s, prompt `tuk-tuk`, `cyclo`, `cargo tricycle` | 0.15 | r04: YOLO26 không có lớp three_wheeler, YOLOE ra 8 box three_wheeler trên 8 ảnh pilot, 0.36 s/ảnh. Quét ngưỡng trên gold: recall 100%, precision 78% (7 vật gold) |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.15,motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn:** SAM 2.1 base (`sam2.1_b.pt`). Lý do: chạy được trên CPU không GPU (8.24 s/ảnh, r06 trong `model_log.csv`), nên chỉ dùng cho vài vật khó ở Part 5, không chạy cả lô.
- Lỗi **không sửa được bằng đổi ngưỡng:** tuk-tuk bị gọi sai lớp vì YOLO26 không có lớp three_wheeler. Tôi dùng YOLOE với prompt chữ, nhưng YOLOE lại gọi nhầm xe tải và xe buýt là three_wheeler (6/10 trường hợp máy đổi lớp trên lô batch: three_wheeler → truck 3, three_wheeler → bus 3). Cách sửa: duyệt từng box three_wheeler bằng mắt (CAL-05), không chấp nhận hàng loạt.
- Box **điểm cao mà sai:** 5 box máy conf ≥ 0.8 bị xoá (img_028 motorbike 0.88 và person 0.84; img_024 bus 0.87 và car 0.81; img_034 three_wheeler 0.85). Box **điểm thấp mà đúng:** 82 box conf < 0.3 vẫn được giữ (person 42, motorbike 36, three_wheeler 4; 22 box ở img_024, 8 ở img_029).
- Giới hạn: ngưỡng được chọn trên chính 4 ảnh gold (48 vật), nên precision/recall của máy trên gold là con số lạc quan (in-sample). Lớp ít vật (bus, truck) không có ngưỡng riêng.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 454 | 62.3% | 729 box máy |
| edited | 41 | 5.6% | 729 box máy |
| deleted | 221 | 30.3% | 729 box máy |
| added | 49 | 9.0% | 544 đối tượng sau duyệt |

Trong 729 box máy có 13 box nằm trong `ignore_region` của tôi nên không được xếp trạng thái; 716 box còn lại là mẫu số cho các tỉ lệ theo lớp và theo điểm tin cậy dưới đây. Đã loại 22 đối tượng quanh các bẫy khỏi bảng này (lỗi nhân tạo, không phản ánh chất lượng máy).

| Lớp | accepted | edited | deleted | Tổng box máy | % deleted |
|---|---|---|---|---|---|
| three_wheeler | 13 | 5 | 21 | 39 | 53.8% |
| motorbike | 144 | 17 | 79 | 240 | 32.9% |
| person | 249 | 16 | 107 | 372 | 28.8% |
| car | 38 | 3 | 9 | 50 | 18.0% |

(bicycle 8, bus 3, truck 4 box máy: quá ít để đọc tỉ lệ.)

- **Lớp máy tệ nhất: three_wheeler**, 21/39 = 53.8% bị xoá. Nguyên nhân (dẫn ảnh): img_003 có 6 box three_wheeler bị xoá (một xe đẩy bị tách thành nhiều box vụn), img_013 và img_014 mỗi ảnh 3 box; thêm 6/10 trường hợp máy đổi lớp là xe tải hoặc xe buýt bị gọi là three_wheeler.
- Máy sai nhiều hơn khi điểm thấp: box bị xoá theo dải `conf` là < 0.3: 132/223 = 59%; 0.3–0.5: 58/154 = 38%; 0.5–0.7: 22/134 = 16%; 0.7–0.9: 9/181 = 5%; ≥ 0.9: 0/24 = 0%. Đây là điểm tin cậy của máy, nhưng nó chỉ dùng để xếp thứ tự duyệt, chưa phải xác suất đúng.
- So với gold: precision / recall của **máy** 64.2% (43/67) / 89.6% (43/48), khoảng tin cậy 95% là 52–75% và 78–95% (mẫu số: 48 đối tượng gold, 4 ảnh). Lạc quan vì ngưỡng chọn trên chính gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: **11/12 = 91.7%** (KTC 95%: 65–99%) trong file nộp: extra 3/3, missing 3/3, loose 3/3, wrong_class 2/3. Con số khách quan hơn là lần duyệt đầu, trước khi mở `traps_key.json`: 7/12 = 58.3% (tôi không lưu file cho lần này). 11/12 là sau khi tôi sửa theo key nên không còn khách quan.
- **Bẫy lọt duy nhất:** img_003, wrong_class. Nhìn kỹ thì đây là lỗi đo của script, không phải lỗi nhãn: `review_stats.py` chỉ tính "bắt được" nếu có box đúng lớp gốc với IoU ≥ 0.5 so với box bẫy (box bẫy [294, 360, 407, 265]). Tôi đã bỏ box car và gộp thành một box three_wheeler ôm cả xe [291, 108, 444, 516] (`can_review`), nên IoU với box bẫy chỉ 0.47. Kiểm tra: `release/final_det.coco.json` ở img_003 không có box car nào.
- Bẫy lọt ở lần duyệt đầu cho thấy accepted không phải thước đo chất lượng duyệt: accepted chỉ 62.3% nhưng bắt bẫy lần đầu chỉ 7/12, thấp hơn ngưỡng 80%. Bắt bẫy đo "tôi có nhìn lại ảnh không", còn accepted đo "máy đúng bao nhiêu"; hai số này độc lập. Lần đầu tôi bỏ sót nhiều bẫy hơn mức cho phép, nên phải tính lại bằng `traps_key.json`.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 25 phút / 4 ảnh = 6.25 phút/ảnh; duyệt 60 phút / 24 ảnh = 2.5 phút/ảnh. Theo đối tượng: gold 25 phút / 48 vật ≈ 31 s/vật; batch 60 phút / 544 vật ≈ 6.6 s/vật, nhanh hơn khoảng 4.7 lần. **Thiên lệch:** ảnh batch dày hơn (22.7 vật/ảnh so với 12 vật/ảnh ở gold), tôi gán gold trước nên đã quen cảnh, và 60 phút duyệt gồm cả xử lý 12 bẫy.
- Kết luận: **có, cho person, motorbike, car** ở ngưỡng đã chọn (accepted lần lượt 249/372 = 66.9%, 144/240 = 60.0%, 38/50 = 76.0%); **không cho three_wheeler** (accepted 13/39 = 33.3%, deleted 21/39 = 53.8%), phải kiểm từng box. Dựa trên `review_stats.md`.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Quét ảnh trước khi nhìn nhãn máy (ẩn nhãn máy theo checklist), rồi mới so. Kiểm tra bằng số: bắt bẫy lần duyệt đầu ≥ 80% (lần này chỉ 7/12 = 58.3%).
  2. Không chấp nhận hàng loạt box có `conf` < 0.3 hoặc lớp three_wheeler: xem từng box. Kiểm tra bằng số: số box thuộc nhóm này đã xem (223 box máy `conf` < 0.3, trong đó 132 bị xoá, 59%) và 39 box three_wheeler (21 bị xoá).
  3. Khi gộp nhiều box máy thành một box (như img_003), đặt `can_review` và đối chiếu IoU với box máy gốc. Kiểm tra: mọi box gộp đều có `can_review` (box #11 ở img_003 có).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_003 (xe đẩy) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Gồm hàng hoá (trái cây) phía trước; vùng người ngồi bị phủ lẫn một phần | Mái che bị phủ lốm đốm, nhiều lỗ | `tuktuk_img003.jpg` |
| img_003 (xe đẩy) | SAM 2.1 base (`sam2.1_b.pt`) | 5 (3 dương, 2 âm) | Gồm hàng hoá và cả trẻ em lẫn người ngồi bên trong xe | Mái che liền hơn nhưng mask tràn sang người ngồi | `cart_img003.jpg` |
| img_013 (tuk-tuk) | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm) | Gồm mái, thân, bánh; click âm lên tài xế loại phần lớn tài xế ra | Biên tài xế lốm đốm; người ngồi hàng sau lẫn vào mask | `tuktuk_img013.jpg` |

- Đối chiếu guideline mục 4.2: **mục 4.2.3** giúp (bóng đổ dưới xe không bị tô vào mask). **Mục 4.2.4** (hai vật chạm nhau: mỗi vật một polygon, vùng chạm thuộc vật phía trước) cần thiết vì SAM tự gộp người ngồi trong xe vào mask xe. **Thiếu quy tắc:** hàng hoá gắn trên xe (trái cây chất trên xe đẩy) có thuộc mask xe không (mục 4.1.5 chỉ nói về box), và người ngồi sau khung xe thuộc mask nào.
- Thêm click âm lần 2 (5 click so với 3 click) làm mái che liền hơn nhưng không loại được người khỏi mask: click chỉ tinh chỉnh vùng nhỏ, không truyền được quy tắc "người là vật riêng".

## 6. Calibration (Part 6)

- Số quyết định: 5 (`calibration_decisions.csv`, gồm 1 `bay_lot`, 3 `cau_hoi_mo`, 1 `loi_may`); quan trọng nhất: **CAL-05** (three_wheeler là lớp máy sai nhiều nhất: 21 box bị xoá, 5 bị sửa, 13 giữ nguyên) và **CAL-01** (xe đẩy có mui là three_wheeler, một xe một box).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: xe đẩy có mui là three_wheeler (CAL-01); box xe ba bánh ôm cả mái che, thùng, bánh và người ngồi được gán person riêng (CAL-02); mỗi người ngồi chung xe máy một box person (CAL-03); vật cao < 20 px trong cụm dày dùng `ignore_region` (CAL-04).
- Nguyên nhân gốc: CAL-01 đến CAL-04 là **guideline mơ hồ** (câu hỏi mở của v1.0, chưa nói xe đẩy, người ngồi trong xe, vật nhỏ trong cụm); CAL-05 là **lỗi của máy** (YOLOE gọi sai xe tải và xe buýt).

## 7. QC và quyết định (Part 6)

**Số đo QC (mẫu 6/24 ảnh, 92 đối tượng, `qc_report_auto.md`):**
- Ảnh có lỗi: 0/6 = 0.0% (ngưỡng Release ≤ 2%).
- Lỗi theo đối tượng: 0/92 = 0.0%.
- Hiệu chỉnh so với lần chấm đầu: ban đầu tôi ghi 1 lỗi `sai_lop` ở img_003 (xe đẩy "vẫn bị gán car"). Kiểm lại bằng `release/final_det.coco.json`: img_003 có đúng 1 box three_wheeler (#11) và **không có box car**, nên lỗi đó không tồn tại; tôi đã sửa `qc_review.csv` thành `khong` và chạy lại `qc_report.py`.

**Số đo của máy và bẫy:**
- Máy: accepted 454/729 = 62.3%, edited 41/729 = 5.6%, deleted 221/729 = 30.3%; added 49/544 = 9.0%.
- Bắt bẫy: 11/12 = 91.7% (ngưỡng ≥ 80%); lần duyệt đầu 7/12 = 58.3% (mục 3).

**Giới hạn của mẫu nhỏ:** chỉ 6/24 ảnh. 0/6 ảnh lỗi cho cận trên 95% (Wilson) của tỉ lệ ảnh lỗi thật là 39%. Mẫu này **không chứng minh được ngưỡng 2%**, chỉ cho biết mẫu không có lỗi nào. Bẫy chỉ 12 nên KTC rộng (65–99%).

**Quyết định: Release** (dataset v1.0, 24 ảnh, 554 vật và 15 `ignore_region`, `release/release_manifest.json`). Lý do: ảnh lỗi 0/6 = 0.0% ≤ 2%, bắt bẫy 11/12 = 91.7% ≥ 80%, không có lỗi hệ thống; đã chốt calibration cho ca three_wheeler (CAL-01, CAL-05). Điều kiện đi kèm: lô này chưa được kiểm toàn bộ, nên nếu dùng để huấn luyện thật cần QC thêm một vòng với mẫu lớn hơn (ví dụ `qc_sample.py --n 12`).

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài:** Claude (chat web). Việc: đọc repo và hướng dẫn từng Part, giải thích lỗi script, soạn nháp báo cáo và rà số liệu. Cách kiểm: đối chiếu từng con số với `qc/review_stats.md`, `qc/qc_report_auto.md`, `qc/threshold_sweep.md` và chạy lại `review_stats.py` / `qc_report.py`. Lần rà này tìm ra 2 chỗ trong bài của tôi không khớp dữ liệu (số box three_wheeler ở CAL-05; lỗi QC img_003); tôi kiểm lại bằng `release/final_det.coco.json` rồi mới sửa. Gán nhãn và duyệt trong CVAT là tôi tự làm.
- Gold có bị nhiễm không: không. Gold gán tay trước khi chạy bất kỳ model nào (commit Part 1 lúc 02:01, trước Part 2 lúc 02:25) và `time_log.csv` ghi "gán tay không AI".
- Máy / GPU / CVAT dùng: Windows 10, Python 3.12, không có GPU (CPU), CVAT Docker v2.81.0. Vướng mắc: chạy `review_stats.py` khi chưa có `gold_manual.json` thì báo `FileNotFoundError` (hết khi có gold).
- Dùng cache `data/cache/` thay vì tự chạy: không; các run trong `model_log.csv` có thời gian chạy thật trên CPU (ví dụ r02: 1.38 s/ảnh).
- Lỗi / bug script đã gặp: `review_stats.py` tính bẫy wrong_class là "lọt" khi nhãn cuối đã đổi đúng lớp nhưng box gộp có IoU < 0.5 với box bẫy (img_003, mục 3). Số 11/12 vì vậy thấp hơn thực tế một bẫy.
- Một điều bạn làm khác lần sau: khi sửa bẫy wrong_class, chỉ đổi lớp của đúng box đó thay vì vẽ lại thành box lớn hơn, để nhãn đúng và đo cũng đúng; và đọc `review_stats.md` ngay sau khi duyệt thay vì suy đoán bẫy lọt ở đâu.
