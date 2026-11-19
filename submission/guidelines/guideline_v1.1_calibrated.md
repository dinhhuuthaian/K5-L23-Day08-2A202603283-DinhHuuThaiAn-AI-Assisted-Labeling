# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603283 - Đinh Hữu Thái An

> **Chỉ viết phần thay đổi so với v1.0** (không copy cả file). Chốt các câu hỏi mở ở mục 6 của v1.0 (hoặc nói rõ câu nào chưa chốt, vì sao)
> và thêm quy tắc mới rút ra từ lỗi của máy / lỗi của bạn. Mỗi ca khó **có ảnh ví dụ** (`img_0xx`) và khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 4 | Người ngồi trong xe ba bánh có mui: ngưỡng 20% có hợp lý không | Giữ 20%, tính theo phần thân nhìn thấy; người đạt ngưỡng gán person riêng, box người chồng lên box xe; box xe ôm cả mái che, thùng và bánh | img_003, img_013 | CAL-02 |
| 2 | Khối xe máy dày gần camera: từng xe hay ignore_region | Gán từng vật rõ, tách được; ignore_region chỉ khi cao < 20 px hoặc cụm không tách được | img_035, img_018 | CAL-04 |
| 6 | Box chồng nhau quá nhiều: gộp hay giữ | Giữ, không gộp; hai người ngồi chung một xe máy là hai box person | img_029 | CAL-03 |

Câu 1 (xe đạp điện), 3 (hàng nhô ra), 5 (kính xe, chỉ áp cho mask) và 7 (mức `conf` tối thiểu) **chưa chốt** trong lượt này: cần thêm ảnh ví dụ trước khi quyết định.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003 | Xe đẩy trái cây có mui, thùng, bánh | Bẫy wrong_class cài sẵn gán car cho nửa dưới xe; YOLOE tách xe thành nhiều box three_wheeler vụn (6 box bị xoá) | Một box three_wheeler ôm cả xe (`can_review`), người ngồi gán person riêng | Mục 2: xe có mui hoặc thùng, kể cả xe đẩy, là three_wheeler, một xe một box (CAL-01) |
| img_013 | Tuk-tuk có hành khách ngồi | YOLOE ra box three_wheeler nhưng nhiều box phải sửa | Sửa box tay, gán person riêng cho hành khách | Không đổi guideline (CAL-05) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 4 (người trong xe ba bánh có mui), câu 2 và 6 (khối xe dày, box chồng); làm rõ xe đẩy có mui là three_wheeler | CAL-01, CAL-02, CAL-03, CAL-04 |
