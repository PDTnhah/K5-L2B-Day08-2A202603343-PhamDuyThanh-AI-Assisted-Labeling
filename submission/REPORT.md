# Báo cáo Day 08 — 2A202603343 - Phạm Duy Thanh

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m (`yolo26m.pt`) | motorbike = 0.1; person, car, bicycle, bus, truck = 0.3 (mặc định) | r01: bắt chắc person / motorbike / car / bicycle. r02: một ngưỡng chung 0.5 làm số box giảm 215 → 139 nhưng không sửa được lỗi sai lớp, nên chọn ngưỡng theo lớp. Ngưỡng lấy từ `qc/threshold_sweep.md` (nguồn r05): motorbike 0.1 đạt recall 90% (9/10), precision 35% (9/26); person và car không ngưỡng nào ≤ 0.5 đạt 90%; bicycle (1 vật), bus (1), truck (0) quá ít mẫu nên giữ mặc định. |
| three_wheeler | YOLOE-26s (`yoloe-26s-seg.pt`), prompt: `three_wheeler` | 0.15 | r01, r02: YOLO26 không có lớp này nên bỏ sót tuk-tuk. r03: YOLOE bắt được tuk-tuk (nhưng sót xích lô ở img_007, nhầm xe bán tải ở img_002). Quét ngưỡng (nguồn r06): ở 0.15 recall 100% (6/6), precision 67% (6/9). |

- Lệnh `merge_prelabels.py` đã dùng (dán nguyên):
  ```
  python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json
  ```
  Kết quả in ra:
  ```
  Gộp 2 file -> 726 box (person=311, motorbike=274, car=65, three_wheeler=52, bicycle=13, truck=7, bus=4) -> work\prelabel\merged_review\pre.coco.json
  Box bị loại (dưới ngưỡng, hoặc chồng lên box ưu tiên cao hơn): y26_all:person(dưới ngưỡng)×346, y26_all:motorbike(dưới ngưỡng)×106, y26_all:car(dưới ngưỡng)×83, yoloe_all:three_wheeler(dưới ngưỡng)×49, y26_all:bicycle(dưới ngưỡng)×21, y26_all:truck(dưới ngưỡng)×16, y26_all:car×11, y26_all:truck×6, y26_all:motorbike×2, y26_all:bus×2, y26_all:bus(dưới ngưỡng)×2, y26_all:bicycle×1
  ```
  11 box `car` và 6 box `truck` của YOLO26 bị loại vì chồng lên box `three_wheeler` của YOLOE (file đứng trước thắng).
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: **SAM 2.1 base** (`sam2.1_b.pt`) cho cả 3 vật. Đây là bản có sẵn trong bộ `core`, đã dùng ở r04; mask liền mạch, bám viền mái và bánh xe tốt. Tôi chạy thêm MobileSAM (`mobile_sam.pt`, SAM 1, nhẹ khoảng 40 MB) trên img_013 với cùng 3 click để so. Kết quả: MobileSAM lại tách người lái tốt hơn, còn SAM 2.1 giữ gần hết người lái trong mask. Như vậy bản lớn hơn không tự hiểu label contract; điều quyết định là đặt đủ click âm. Không dùng SAM 3 vì cần xin quyền tải trên Hugging Face.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? Lỗi sai lớp: YOLO26 gọi tuk-tuk là `car` với điểm cao (img_012: 4 tuk-tuk bị gọi car, điểm 0.65–0.81), gọi car là `truck` (img_001, 0.88), xe bán tải bị gọi `truck` (r01) hoặc `three_wheeler` (r03, img_002); tăng ngưỡng lên 0.5 vẫn giữ các box này (r02). Lỗi bỏ sót: person chỉ đạt recall 83% (10/12) và car 88% (7/8) ngay cả ở ngưỡng 0.1, nên hạ ngưỡng cũng không tìm thêm được; xích lô bị cả hai model bỏ sót (img_007). Cách sửa: khi gộp đặt file YOLOE trước để box `three_wheeler` thắng box `car` / `truck` chồng lên (IoU ≥ 0.6), rồi review tay: đổi lớp sai, thêm box bị sót, kiểm tra kỹ `truck` và `three_wheeler`.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): Điểm cao mà sai: img_012 tuk-tuk bị gọi `car` (0.65–0.81); img_001 car bị gọi `truck` (0.88); img_020 box `three_wheeler` 0.86 của YOLOE không khớp vật nào trong gold. Điểm thấp mà đúng: img_036 motorbike 0.12 và 0.22; img_012 three_wheeler 0.19 và 0.24; img_020 car 0.10. Đây là lý do ngưỡng của motorbike (0.1) và three_wheeler (0.15) phải đặt thấp.
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: Ngưỡng được chọn và chấm trên cùng 4 ảnh gold (38 vật) nên precision / recall ở trên là in-sample, sẽ lạc quan hơn trên 24 ảnh batch. Mẫu rất nhỏ: three_wheeler 6 vật, bicycle 1, bus 1, truck 0, nên ngưỡng chỉ là ước lượng thô. Motorbike = 0.1 giữ recall nhưng precision chỉ 35% (17 box thừa trên gold), người review sẽ phải xóa nhiều box.

## 2. AI error profile (từ `qc/review_stats.md`)

24 ảnh batch; đã loại 25 đối tượng nằm quanh 12 bẫy (lỗi nhân tạo, không phải lỗi máy).

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 544 | 544/621 = 87.6% | 621 box máy |
| edited | 15 | 15/621 = 2.4% | 621 box máy |
| deleted | 62 | 62/621 = 10.0% | 621 box máy |
| added | 25 | 25/584 = 4.3% | 584 đối tượng sau duyệt |

Trong deleted + added có 13 trường hợp là đổi lớp (box đúng chỗ, sai lớp), nên số vật máy bỏ sót thật khoảng 25 − 13 = 12.

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)?
  - **three_wheeler** tệ nhất: 41 box máy, chỉ giữ nguyên 10/41 = 24%, xoá 24/41 = 59%, sửa 7. Box bị xoá tập trung ở img_003 (7 box), img_013 (3), img_014 (3). Trong số đổi lớp, 3 box `three_wheeler` thực ra là `truck` và 3 box là `bus`: YOLOE gọi cả xe tải, xe buýt là xe ba bánh, và ngưỡng 0.15 thấp nên giữ thêm nhiều box ma.
  - **truck** cũng kém: 6 box máy, giữ 1/6, xoá 5 (img_013: 2, img_024: 3); 2 box là `bus`, 1 box là `car`.
  - **bus**: máy giữ đúng cả 3 box nó vẽ, nhưng tôi phải thêm 5 box bus (img_013: 3, img_018: 1, img_024: 1), chủ yếu do máy gọi xe buýt là `three_wheeler` / `truck` / `car`.
  - Ngược lại, các lớp COCO quen thuộc làm tốt: person giữ 259/267 = 97%, motorbike 217/242 = 90%, car 48/54 = 89%.
- So với gold: precision / recall của **máy** = 43.6% (34/78 box máy, KTC 95% 33–55%) / 89.5% (34/38, KTC 95% 76–96%) (mẫu số: 38 đối tượng gold trên 4 ảnh). 44 box thừa chủ yếu là person (19) và motorbike (17) ở img_001 và img_036; 21/44 box thừa có cạnh ngắn dưới 20 px (người, xe ở rất xa mà gold không vẽ). 4 vật bị sót: car (img_001), person (img_020), motorbike và person (img_036). Số này lạc quan vì ngưỡng được chọn trên chính 4 ảnh gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy:
  - **Lần duyệt đầu: 8/12 = 66.7%**, dưới mức 80%. Lọt 4 bẫy: 2 `missing` (img_021, img_024), 1 `loose` (img_029), 1 `wrong_class` (img_005).
  - Sau khi export, tôi mở `traps_key.json` để biết ảnh nào lọt, sửa lại rồi export lần 2: **11/12 = 91.7%** (KTC 95% 65–99%, nguồn `qc/review_stats.md`). Vì lần 2 đã nhìn đáp án nên con số phản ánh đúng khả năng duyệt của tôi là 8/12.
  - Bẫy vẫn lọt ở lần 2 là img_024 (`missing` person). Tôi đã vẽ box person nhưng chỉ ôm phần thân trên (y 490–534, trong khi box gốc là y 488–567), IoU với box gốc là 0.49 < 0.5 nên script chấm là lọt.
  - Vì sao lọt: cả 4 bẫy lọt ở lần đầu đều nằm trong 6 ảnh xe máy dày (005, 018, 021, 024, 028, 029). Ở các ảnh đông, tôi bỏ bước 0 (ẩn nhãn, quét và đếm trước) nên không thấy người chưa có box (img_021, img_024). Tôi cũng không kiểm 4 cạnh nên giữ nguyên box lệch (img_029). Ở img_005, tôi thấy nhãn `bicycle` sai nhưng đổi thành `motorbike` thay vì `person`, tức là sửa nhãn mà không nhìn kỹ vật trong box.
  - Tôi cũng không dùng `ignore_region` nào trong batch (0 vùng, trong khi gold có 12 vùng) và vẫn giữ 16 box cao dưới 20 px (11 motorbike, 7 box ở img_018).
- Accepted cao mà bắt bẫy thấp nói gì về cách bạn duyệt?
  - Accepted 87.6% (544/621) cao, nhưng lần đầu tôi chỉ bắt được 66.7% bẫy. Vậy accepted cao không chứng minh máy tốt: một phần box được giữ vì tôi lướt qua, không phải vì tôi đã kiểm.
  - Dấu hiệu thứ hai: trên gold, 44/78 box máy (56%) là sai so với nhãn tôi gán tay, nhưng ở batch tôi chỉ xoá 62/621 (10%). Hai bộ ảnh khác nhau và gold chỉ có 4 ảnh nên không so trực tiếp được, nhưng khoảng chênh lớn cho thấy tôi dễ làm theo máy, nhất là với box nhỏ ở xa.
  - `added` thấp (4.3%) cũng không có nghĩa máy ít sót: 2/3 bẫy `missing` tôi đã không thấy ở lần đầu.

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`):
  - Gán tay 10 phút / 4 ảnh = **2.5 phút/ảnh**, tức khoảng 16 giây/vật (38 vật).
  - Duyệt 100 phút / 24 ảnh = **4.2 phút/ảnh**, tức khoảng 10 giây/vật (584 vật).
  - Tính theo ảnh thì duyệt chậm hơn, vì ảnh batch đông hơn nhiều (khoảng 24 vật/ảnh so với 9.5 vật/ảnh ở gold). Tính theo vật thì duyệt nhanh hơn khoảng 35%.
  - Thiên lệch: tôi gán gold trước nên khi duyệt đã quen kiểu cảnh và quen CVAT, nên thời gian duyệt có lợi một phần nhờ quen tay. Ngược lại, 4 ảnh gold ít vật và tôi gom vật nhỏ vào 12 `ignore_region`, nên thời gian gán tay cũng thấp hơn so với gán tay ảnh đông.
- Kết luận: **có, nhưng chỉ cho person, motorbike, car**. Ba lớp này giữ nguyên 97% (259/267), 90% (217/242), 89% (48/54) box máy, và tính theo vật thì duyệt nhanh hơn gán tay (khoảng 10 giây so với 16 giây). **Không đáng dùng cho three_wheeler** (giữ 24% = 10/41, xoá 59%) **và truck** (giữ 1/6): máy tạo nhiều box sai hơn box đúng. Lần sau, với hai lớp này tôi sẽ tăng ngưỡng `three_wheeler` hoặc vẽ tay tuk-tuk, xe tải, xe buýt.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ở 100% ảnh, ẩn nhãn máy và đếm số người theo 4 góc phần tư trước khi hiện nhãn, ghi số đếm vào ghi chú; sau khi duyệt, số box person của ảnh phải khớp số đếm ±1 (kiểm: có số đếm cho 24/24 ảnh).
  2. Duyệt 6 ảnh xe máy dày ngay đầu phiên hoặc ngay sau giờ nghỉ, và nghỉ 5 phút sau mỗi 8 ảnh (kiểm: `time_log.csv` có ít nhất 2 dòng `break`), vì 4/4 bẫy lọt lần đầu đều nằm trong 6 ảnh này.
  3. Trước khi export, dùng Filters xem lại 100% box của `three_wheeler`, `truck`, `bus`, `bicycle` (4 lớp máy sai nhiều nhất) và gom mọi vật cao dưới 20 px vào `ignore_region` (kiểm: `validate_labels.py` không còn cảnh báo box < 20 px).

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 (tuk-tuk có người lái) | SAM 2.1 base | 3 (2 dương: mái 600,110 và thân 400,480; 1 âm: lưng người lái 720,300) | **Có, sai quy tắc**: gồm gần hết người lái (đầu, áo, quần, chân); click âm chỉ khoét được vài đốm quanh nó. Người áo sọc ngồi phía sau, nhìn qua khung hở, cũng bị tô một phần | Vi phạm 4.2.4 (hai vật chạm nhau phải tách riêng). 1 click âm không đủ, cần thêm click âm ở đầu và chân người lái | `tuktuk_img013_sam2.jpg` |
| img_013 (cùng xe, để so) | MobileSAM (SAM 1) | 3 (cùng click như dòng trên) | **Một phần**: áo và quần người lái đã ra khỏi mask, còn dính mặt và bàn chân / dép | Sàn trước và phần đầu xe màu đỏ có chỗ thủng lốm đốm, mask kém liền mạch hơn SAM 2.1 | `tuktuk_img013_sam1.jpg` |
| img_004 (tuk-tuk có người đứng sát) | SAM 2.1 base | 3 (2 dương: thân 620,480 và 700,560; 1 âm: chân người áo đen đứng giữa hai xe 465,450) | Không: người áo đen và người đứng bên phải đều nằm ngoài mask; xe không chở hàng | Click dương 700,560 nằm sát mặt đường dưới cản trước, nên mask lấn xuống vùng tối dưới gầm và biên ở đó bị răng cưa (bóng không thuộc vật, 4.2.3). Kính chắn gió nằm trong mask, đúng 4.2.3 | `tuktuk_img004_sam2.jpg` |
| img_017 (tuk-tuk hông hở) | SAM 2.1 base | 3 (2 dương: ghế 560,420 và thân 680,500; 1 âm: 645,355) | Không có người lái; không chở hàng | Click âm 645,355 rơi trúng mái xe nên không có tác dụng (mái vẫn trong mask). Khoảng hở lớn giữa mái và thân (nhìn thấy cửa hàng phía sau) bị tô kín, không khoét ra, trái 4.2.2. Muốn khoét phải đặt click âm vào chính vùng nền nhìn qua hông xe | `tuktuk_img017_sam2.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu?
  - **Quy tắc giúp được:**
    - 4.2.4 ("hai vật chạm nhau: mỗi vật một polygon, không chồng") cùng ca khó ở mục 5 ("người là vật riêng") cho biết rõ mask xe không được gồm người lái. Nhờ đó tôi đánh giá được mask img_013 của SAM 2.1 là sai, dù trông gọn.
    - 4.2.3 (bóng không thuộc vật, kính thuộc vật) giúp chấm img_004.
  - **Còn thiếu:**
    1. 4.2.2 nói lỗ lớn thì bỏ ra, nhưng không nói với tuk-tuk hông hở thì cả khoảng giữa mái và thân có tính là "lỗ" không (img_017).
    2. 4.2.4 nói "vùng chạm thuộc vật ở phía trước", nhưng người lái ngồi *trong* khoang xe thì không rõ ai ở phía trước. Nên ghi thẳng: người luôn được tách khỏi mask xe.
    3. 4.2.3 nói kính thuộc vật nhưng câu hỏi mở số 5 về kính vẫn chưa chốt, nên hai chỗ chưa nhất quán.
    4. Không có cách kiểm "lệch biên ≤ 3 px" khi chỉ nhìn ảnh overlay.
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); khác gì offline: Không làm (phần tuỳ chọn); chỉ chạy offline bằng `sam2_points.py`.

## 6. Calibration (Part 6)

- Số quyết định: **9** (`calibration_decisions.csv`): 5 ca `loi_may`, 3 ca `cau_hoi_mo`, 1 ca `bay_lot`. Quan trọng nhất:
  - **CAL-01**: lỗi máy lớn nhất. YOLOE gọi xe tải, xe buýt là `three_wheeler`, nên tôi phải xoá 24/41 box `three_wheeler`.
  - **CAL-03**: chỗ v1.0 thật sự mơ hồ. `car` gồm "minivan" nhưng `bus` gồm "minibus", nên xe van chở khách ở img_013 và img_024 không biết gán lớp nào.
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`:
  1. Dấu hiệu nhận biết `three_wheeler`: đếm bánh (CAL-01).
  2. Van chở khách thân hộp, từ 3 hàng ghế → `bus`; minivan gia đình → `car` (CAL-03).
  3. Xe máy đỗ dày gần camera vẫn gán từng xe; chỉ gom `ignore_region` cho cụm xe cao dưới 20 px (câu 2, CAL-04).
  4. Kính xe thuộc xe (câu 5, CAL-05).
  5. Không đặt ngưỡng `conf` để tự chấp nhận box (câu 7, CAL-06).
  6. Người trong xe luôn là polygon riêng (mục 4.2.4, CAL-07).
  7. Khung xe hở tính là lỗ của mask (mục 4.2.2, CAL-08).

  Câu hỏi mở 1, 3, 4, 6 chưa chốt vì chưa gặp ca rõ ràng trong lô.
- Nguyên nhân gốc:
  - **Guideline mơ hồ**: 6 ca (CAL-03 → CAL-08).
  - **Máy sai**: 2 ca (CAL-01, CAL-02). Guideline đã đủ rõ, chỉ thêm dấu hiệu nhận biết và ảnh ví dụ.
  - **Người duyệt chưa áp quy tắc**: 1 ca (CAL-09, bẫy img_024). Tương tự là 16 box cao dưới 20 px tôi vẫn giữ dù mục 4.1.6 đã cấm.

## 7. QC và quyết định (Part 6)

- Mẫu: **5** ảnh (**0** ảnh `can_review` + **5** ngẫu nhiên, seed 42): img_003, img_006, img_014, img_031, img_035, tổng 113 đối tượng. Người kiểm: tôi (**hạn chế**: tôi vừa là người duyệt). Mẫu không có ca khó vì lúc duyệt tôi không tick `can_review` box nào.
- Lỗi theo **đối tượng**: 0/113 = 0.0% · theo **ảnh**: 0/5 = 0.0% (ngưỡng Release ≤ 2%). Nguồn: `qc_review.csv`, `qc_report_auto.md`.
- Top lỗi:
  - **Trong mẫu**: không ghi nhận lỗi nào.
  - **Ngoài mẫu**, đã biết từ Pha 4 (đếm từ `batch_reviewed.json`):
    - **thừa**: 16 box cao dưới 20 px, trái mục 4.1.6, nằm ở 6 ảnh: img_018 (7 box), img_027 (3), img_033 (2), img_034 (2), img_023 (1), img_025 (1). Gồm 11 motorbike, 2 person, 2 car, 1 bicycle.
    - **sai vị trí/kích thước**: img_024 có box person chỉ ôm nửa người (y 490–534 so với vật thật y 488–567).
- **Giới hạn mẫu nhỏ**:
  - Theo ảnh: 0/5 ảnh lỗi, nhưng cận trên Wilson 95% của tỉ lệ ảnh lỗi thật là **43%** (`qc_report_auto.md`).
  - Theo đối tượng: 0/113, cận trên khoảng 3%. Cả hai đều chưa chứng minh được ngưỡng 2%.
  - Bằng chứng cụ thể: 7/24 ảnh có lỗi đã biết, nhưng không ảnh nào rơi vào mẫu 5 ảnh. Mẫu nhỏ chỉ phát hiện được lỗi lớn và phổ biến, không chứng minh được lô sạch.
- Script kiểm được / không kiểm được:
  - **Kiểm được**: định dạng file, lớp có trong taxonomy, đủ 24 ảnh, box nằm trong ảnh, cảnh báo box cao dưới 20 px (`validate_labels.py`), accepted / edited / deleted / added, tỉ lệ bắt bẫy, tỉ lệ lỗi tính từ phiếu QC.
  - **Không kiểm được**: box có ôm sát vật thật không; lớp có đúng thực tế không; vật mà cả tôi lẫn máy cùng bỏ sót (ngoài 4 ảnh gold); tôi có làm bước 0 của checklist thật không. Phiếu QC cũng chỉ là mắt của chính người đã duyệt.

**Quyết định: Rework**, ngày 2026-11-17. Lý do (đi theo Hold → Rework → Release):
- **Không Hold**: không có ảnh hỏng, và các câu hỏi mở chưa chốt đều chưa gặp trong lô (`--guideline-gap 0`).
- **Rework**:
  - Chỉ xét mẫu 5 ảnh thì script gợi ý Release (0/5 ảnh lỗi, bắt bẫy 11/12). Nhưng tôi biết lô còn **lỗi lặp cùng loại** nằm ngoài mẫu: 16 box `thua` cao dưới 20 px, cộng 1 box sai kích thước ở img_024. Tổng là ít nhất 7/24 ảnh có lỗi (29%), vượt xa ngưỡng 2%.
  - Ở lần duyệt độc lập (trước khi xem đáp án), tôi chỉ bắt được 8/12 = 66.7% bẫy, dưới 80%.
  - Báo Release khi đã biết các lỗi này là không trung thực.

Việc cần làm tiếp:
1. Trên Task `batch-review`, xoá 16 box cao dưới 20 px ở img_018, img_027, img_033, img_034, img_023, img_025. Cụm từ 5 xe nhỏ sát nhau thì gom thành một `ignore_region` (mục 4.1.6, CAL-04).
2. Ở img_024, kéo box person ôm hết người (đến khoảng y 567).
3. Duyệt lại 6 ảnh xe máy dày (005, 018, 021, 024, 028, 029) theo bước 0: ẩn nhãn, đếm theo góc phần tư. Lý do: 4/4 bẫy lọt ở lần đầu đều nằm trong nhóm này. Tick `can_review` cho các ca biên.
4. Export các ảnh đã sửa thành `submission/labels/qc_fixed.json`, chạy lại `build_release.py`, rồi `qc_sample.py --force` để rút mẫu vòng 2 (có thêm ảnh `can_review`), điền phiếu, chạy `qc_report.py`, ghi kết quả vòng 2 vào báo cáo.

## 8. Ghi chú cá nhân
