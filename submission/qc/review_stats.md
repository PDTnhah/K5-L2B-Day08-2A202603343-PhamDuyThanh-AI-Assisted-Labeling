# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 621; đối tượng sau duyệt: 584; ngưỡng accepted: IoU >= 0.9
- Đã **loại 25 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 544 | 544/621 = 87.6% | 621 box máy |
| edited (sửa) | 15 | 15/621 = 2.4% | 621 box máy |
| deleted (xoá) | 62 | 62/621 = 10.0% | 621 box máy |
| added (vẽ thêm) | 25 | 25/584 = 4.3% | 584 đối tượng sau duyệt |

Trong số deleted + added, **13 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 12.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 259 | 1 | 7 | 4 |
| bicycle | 6 | 0 | 2 | 4 |
| motorbike | 217 | 6 | 19 | 5 |
| car | 48 | 1 | 5 | 2 |
| bus | 3 | 0 | 0 | 5 |
| truck | 1 | 0 | 5 | 3 |
| three_wheeler | 10 | 7 | 24 | 2 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 38 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 34 | 44 | 4 | 43.6% (KTC 95%: 33–55%) | 89.5% (KTC 95%: 76–96%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 11/12 = 91.7% (KTC 95%: 65–99%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - loose: 3/3
  - missing: 2/3
  - extra: 3/3
  - wrong_class: 3/3
  - lọt: img_024.jpg (missing)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
