# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 729; đối tượng sau duyệt: 544; ngưỡng accepted: IoU >= 0.9
- Đã **loại 22 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 454 | 454/729 = 62.3% | 729 box máy |
| edited (sửa) | 41 | 41/729 = 5.6% | 729 box máy |
| deleted (xoá) | 221 | 221/729 = 30.3% | 729 box máy |
| added (vẽ thêm) | 49 | 49/544 = 9.0% | 544 đối tượng sau duyệt |

Trong số deleted + added, **10 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 39.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 249 | 16 | 107 | 16 |
| bicycle | 6 | 0 | 2 | 5 |
| motorbike | 144 | 17 | 79 | 8 |
| car | 38 | 3 | 9 | 12 |
| bus | 2 | 0 | 1 | 3 |
| truck | 2 | 0 | 2 | 3 |
| three_wheeler | 13 | 5 | 21 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 48 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 43 | 24 | 5 | 64.2% (KTC 95%: 52–75%) | 89.6% (KTC 95%: 78–95%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - missing: 3/3
  - wrong_class: 2/3
  - loose: 3/3
  - lọt: img_003.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
