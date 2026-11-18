# Thông tin học viên

- Họ và tên: Phạm Tiến Dũng
- MSHV: 2A202603061
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: https://github.com/lamsondongbua
- Repo bài nộp (URL): https://github.com/lamsondongbua/K5-L23-DAY08-2A202603061-PhamTienDung-AIPrelabel.git
- Commit chốt bài (hash): tạm thời chưa có
- Máy: Windows 11, GPU NVIDIA RTX 3050 Laptop 4 GB (CUDA 12.4 / PyTorch 2.6.0)
- CVAT (Docker): v2.81.0

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 429 / 45 / 77 (mẫu số: 557 box máy); added: 22 (mẫu số: 496 đối tượng)
- So với gold: precision / recall của **máy** 55,1% / 95,6% (45 đối tượng gold)
- Bắt bẫy: 11/12 = 91,7% (lần đo 2; lần đo 1 là 8/12)
- Thời gian: gán tay 3,75 phút/ảnh (15 phút / 4 ảnh); duyệt có pre-label 4,6 phút/ảnh (110 phút / 24 ảnh)
- Lỗi QC: 62,5% theo ảnh (5/8), 4,2% theo đối tượng (8/191)
- **Quyết định:** Rework — lý do một dòng: ảnh lỗi 5/8 = 62,5% > 2%, lỗi thừa lặp lại (7/8 lỗi) do ngưỡng thấp
