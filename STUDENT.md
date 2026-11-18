# Thông tin học viên

- Họ và tên: Phạm Duy Thanh
- MSHV: 2A202603343
- Lớp (L2 / L3 / khác): L2B
- Tài khoản GitHub: PDTnhah
- Repo bài nộp (URL): https://github.com/PDTnhah/K5-L2B-Day08-2A202603343-PhamDuyThanh-AI-Assisted-Labeling
- Commit chốt bài (hash): ecb60127539218b44861d3cdf9bbea4fda9c3982
- Máy: Windows 11, không dùng GPU, Pha 5 chạy trên Colab GPU T4
- CVAT (Docker): 2.74.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 87.6% / 2.4% / 10.0% (544 / 15 / 62, mẫu số: 621 box máy); added: 4.3% (25, mẫu số: 584 đối tượng sau duyệt)
- So với gold: precision / recall của **máy** 43.6% (34/78 box máy) / 89.5% (34/38 đối tượng gold)
- Bắt bẫy: 11/12 = 91.7% (lần duyệt đầu, trước khi xem đáp án: 8/12 = 66.7%)
- Thời gian: gán tay 2.5 phút/ảnh (10 phút / 4 ảnh); duyệt có pre-label 4.2 phút/ảnh (100 phút / 24 ảnh)
- Lỗi QC: 0.0% theo ảnh (0/5), 0.0% theo đối tượng (0/113); cận trên 95% theo ảnh là 43%
- **Quyết định:** Rework — lý do một dòng: mẫu QC 5 ảnh không có lỗi, nhưng ngoài mẫu còn 16 box cao dưới 20 px (trái mục 4.1.6) và box nửa người ở img_024, tức ít nhất 7/24 ảnh có lỗi; lần duyệt đầu chỉ bắt được 8/12 bẫy
