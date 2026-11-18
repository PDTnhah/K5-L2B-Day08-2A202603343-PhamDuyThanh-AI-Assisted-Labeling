# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603343 - Phạm Duy Thanh

> Bổ sung cho `guideline/guideline_v1.0_label_contract.md`: chỉ ghi phần thay đổi so với v1.0. Mỗi dòng khớp một `case_id` trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: gán từng xe hay `ignore_region`? | Xe cao từ 20 px và còn thấy từ 20% thì **gán từng xe**, kể cả khi đỗ sát nhau. Chỉ gom `ignore_region` cho cụm từ 5 xe cao dưới 20 px (thường ở xa). Không dùng `ignore_region` cho xe gần camera chỉ vì khó vẽ. | img_005, img_018 | CAL-04 |
| 5 | Kính chắn gió / cửa sổ có thuộc mask xe không? | **Thuộc xe** (cả box lẫn mask), kể cả khi nhìn xuyên thấy người hay nội thất. | img_004, img_017 | CAL-05 |
| 7 | Có đặt mức `conf` tối thiểu để chấp nhận box máy không cần sửa? | **Không đặt.** `conf` chỉ dùng để xếp thứ tự xem, mọi box đều phải nhìn. Lý do: nhiều box điểm cao vẫn sai lớp (tuk-tuk bị gọi `car` 0.65–0.81, car bị gọi `truck` 0.88, xe tải bị gọi `three_wheeler` 0.85). | img_012, img_001, img_034 | CAL-06 |

**Chưa chốt:** câu 1 (xe đạp điện), câu 3 (hàng nhô ra), câu 4 (người ngồi trong xe ba bánh có mui), câu 6 (box chồng nhau trong ảnh đêm). Trong 24 ảnh batch chưa gặp đủ ca rõ ràng để quyết; giữ như v1.0 và tick `can_review` khi gặp.

## 2. Ca khó mới (từ lỗi của máy, bẫy lọt, box `can_review`)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_003, img_034, img_013, img_018 | Xe tải, xe buýt (thường ở xa) | YOLOE (prompt `three_wheeler`, ngưỡng 0.15) gọi là `three_wheeler`: 24/41 box `three_wheeler` bị xoá; 3 box thực ra là `truck`, 3 box là `bus`; img_034 sai với điểm 0.85 | Gán theo số bánh và thân xe | **Mục 2, dòng `three_wheeler`**: thêm dấu hiệu nhận biết: đúng 3 bánh (1 trước, 2 sau) hoặc khung tuk-tuk / xích lô / ba gác. Từ 4 bánh có thùng hàng → `truck`; xe chở khách lớn → `bus`. (CAL-01) |
| img_013, img_024 | Xe van chở khách thân hộp cao (kiểu Hiace) | YOLO26 vẽ cả `car` (0.72) và `truck` (0.47) chồng lên cùng một xe (img_013); gọi `truck` (0.53) ở img_024 | `bus` | **Mục 2, dòng `car` và `bus`**: v1.0 mơ hồ vì `car` gồm "minivan" còn `bus` gồm "minibus". Thêm: van chở khách thân hộp cao, từ 3 hàng ghế (kiểu Hiace / Transit) → `bus`; minivan gia đình thân thấp → `car`. (CAL-03) |
| img_027 | Xích lô ở mép phải ảnh | YOLO26 gọi `motorbike` (0.47) | `three_wheeler` + `person` riêng cho người đạp | Không đổi quy tắc (mục 5 v1.0 đã có ca xích lô); thêm img_027 làm ảnh ví dụ. (CAL-02) |
| img_013 | Mask tuk-tuk có người lái ngồi trong khoang | SAM 2.1 (2 click dương, 1 click âm) giữ gần hết người lái trong mask | Tách người khỏi mask xe | **Mục 4.2.4**: người ngồi trên / trong xe luôn là polygon riêng; vùng chồng giữa người và xe thuộc người. (CAL-07) |
| img_017 | Tuk-tuk hông hở, nhìn xuyên thấy cửa hàng phía sau | SAM 2.1 tô kín khoảng hở giữa mái và thân | Bỏ khoảng hở khỏi mask | **Mục 4.2.2**: khoảng hở của khung xe (hông xe, giữa mái và thân) tính là lỗ; lớn hơn khoảng 5% diện tích xe thì bỏ khỏi polygon; cột khung mái vẫn thuộc xe. (CAL-08) |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu hỏi mở 2, 5, 7; thêm dấu hiệu nhận biết `three_wheeler` (đếm bánh); phân biệt van chở khách (`bus`) với minivan (`car`); thêm quy tắc mask cho người trong xe và khung xe hở; thêm ảnh ví dụ xích lô | CAL-01, CAL-03, CAL-04, CAL-05, CAL-06, CAL-07, CAL-08 (CAL-02 chỉ thêm ảnh ví dụ; CAL-09 là lỗi người duyệt, không đổi guideline) |
