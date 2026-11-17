# Báo cáo Day 08 — <<2A202603343 - Phạm Duy Thanh>>

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | motorbike = 0.1; person, car, bicycle, bus, truck = 0.3 (mặc định) | r01: bắt chắc person / motorbike / car / bicycle. r02: một ngưỡng chung 0.5 làm số box giảm 215 → 139 nhưng không sửa được lỗi sai lớp, nên chọn ngưỡng theo lớp. Ngưỡng lấy từ `qc/threshold_sweep.md` (nguồn r05): motorbike 0.1 đạt recall 90% (9/10), precision 35% (9/26); person và car không ngưỡng nào ≤ 0.5 đạt 90%; bicycle (1 vật), bus (1), truck (0) quá ít mẫu nên giữ mặc định. |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt: `three_wheeler` | 0.15 | r01, r02: YOLO26 không có lớp này nên bỏ sót tuk-tuk. r03: YOLOE bắt được tuk-tuk (nhưng sót xích lô ở img_007, nhầm xe bán tải ở img_002). Quét ngưỡng (nguồn r06): ở 0.15 recall 100% (6/6), precision 67% (6/9). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên): <<>>
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: <<chất lượng mask, tốc độ, RAM, quyền truy cập>>
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi sai lớp: YOLO26 gọi tuk-tuk là `car` với điểm cao (img_012: 4 tuk-tuk bị gọi car, điểm 0.65–0.81), gọi car là `truck` (img_001, 0.88), xe bán tải bị gọi `truck` (r01) hoặc `three_wheeler` (r03, img_002); tăng ngưỡng lên 0.5 vẫn giữ các box này (r02). Lỗi bỏ sót: person chỉ đạt recall 83% (10/12) và car 88% (7/8) ngay cả ở ngưỡng 0.1, nên hạ ngưỡng cũng không tìm thêm được; xích lô bị cả hai model bỏ sót (img_007). Cách sửa: khi gộp đặt file YOLOE trước để box `three_wheeler` thắng box `car` / `truck` chồng lên (IoU ≥ 0.6), rồi review tay: đổi lớp sai, thêm box bị sót, kiểm tra kỹ `truck` và `three_wheeler`.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Điểm cao mà sai: img_012 tuk-tuk bị gọi `car` (0.65–0.81); img_001 car bị gọi `truck` (0.88); img_020 box `three_wheeler` 0.86 của YOLOE không khớp vật nào trong gold. Điểm thấp mà đúng: img_036 motorbike 0.12 và 0.22; img_012 three_wheeler 0.19 và 0.24; img_020 car 0.10. Đây là lý do ngưỡng của motorbike (0.1) và three_wheeler (0.15) phải đặt thấp.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Ngưỡng được chọn và chấm trên cùng 4 ảnh gold (38 vật) nên precision / recall ở trên là in-sample, sẽ lạc quan hơn trên 24 ảnh batch. Mẫu rất nhỏ: three_wheeler 6 vật, bicycle 1, bus 1, truck 0, nên ngưỡng chỉ là ước lượng thô. Motorbike = 0.1 giữ recall nhưng precision chỉ 35% (17 box thừa trên gold), người review sẽ phải xóa nhiều box.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | <<>> | <<>> | <<box máy>> |
| edited | <<>> | <<>> | <<>> |
| deleted | <<>> | <<>> | <<>> |
| added | <<>> | <<>> | <<đối tượng sau duyệt>> |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? <<>>
- So với gold: precision / recall của **máy** <<>> (mẫu số: <<đối tượng gold>>)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy: <<.../12>>; loại nào lọt: <<>>; vì sao (bạn đã bỏ bước nào của checklist)? <<>>
- Accepted cao mà bắt bẫy thấp (hoặc ngược lại) nói gì về cách bạn duyệt? <<>>

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay <<... phút/ảnh>> vs duyệt <<... phút/ảnh>>. Thiên lệch (bạn gán gold trước nên đã quen cảnh): <<>>
- Kết luận: <<có / không / chỉ cho lớp nào>>, dựa trên <<số liệu>>
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. <<>>
  2. <<>>
  3. <<>>

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| <<img_0xx>> | <<>> | <<>> | <<>> | <<>> | <<>> |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? <<>>
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: <<>>

## 6. Calibration (Part 6)

- Số quyết định: <<>> (`calibration_decisions.csv`); quan trọng nhất (dẫn `CAL-xx`): <<>>
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: <<>>
- Nguyên nhân gốc: <<guideline mơ hồ / chưa nắm quy tắc / máy>>

## 7. QC và quyết định (Part 6)

- Mẫu: <<n>> ảnh (<<m>> ảnh `can_review` + <<k>> ngẫu nhiên, seed 42); người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt).
- Lỗi theo **đối tượng**: <<.../... = ...%>> · theo **ảnh**: <<.../... = ...%>> (ngưỡng Release ≤ 2%)
- Top lỗi (trong: bỏ sót, thừa, sai lớp, sai vị trí/kích thước, sai thuộc tính, sai mask) + một ảnh ví dụ mỗi loại: <<>>
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): <<>>
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): <<>>

**Quyết định: <<Release / Hold / Rework>>**, ngày <<YYYY-MM-DD>>. Lý do (đi theo Hold → Rework → Release): <<>>
Nếu Rework / Hold, việc cần làm tiếp: <<>>

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): <<>>
- Gold có bị nhiễm không (bạn có nhìn pre-label hay hỏi AI trước khi gán xong)? <<không / có, như sau: ...>>
- Máy / GPU / CVAT dùng; vướng mắc cài đặt: <<>>
- Dùng cache `data/cache/` thay vì tự chạy? <<không / có, vì ...>>
- Lỗi / bug script đã gặp (nếu có): <<>>
- Một điều bạn làm khác lần sau: <<>>
