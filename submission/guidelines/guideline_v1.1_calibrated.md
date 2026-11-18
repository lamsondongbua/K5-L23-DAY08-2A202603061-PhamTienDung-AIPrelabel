# Label contract — Phố châu Á — BỔ SUNG v1.1 — 2A202603061 - Phạm Tiến Dũng

> Chỉ phần thay đổi so với v1.0. Mỗi quy tắc khớp một dòng trong `qc/calibration_decisions.csv`.

## 1. Câu hỏi mở đã chốt (từ mục 6 của v1.0)

| # | Câu hỏi | Quyết định | Ảnh ví dụ | `case_id` |
|---|---|---|---|---|
| 2 | Khối xe máy đỗ dày gần camera: từng xe hay `ignore_region`? | Từng xe nếu thấy rõ và cao >= 20 px; cụm chồng nhau không tách được thì một `ignore_region` | img_006, img_024 | CAL-03 |
| 5 | Kính chắn gió / người trong xe có thuộc mask xe không? | Kính thuộc xe; người lái / hành khách **không** thuộc mask xe, có mask riêng | img_013 | CAL-05 |
| 1, 3, 4, 6, 7 | xe đạp điện; hàng nhô; người trong xe ba bánh (20%); box chồng ở ảnh đêm; mức conf tối thiểu | **Chưa chốt**: lô này chưa có đủ ca để quyết; giữ nguyên v1.0 | — | — |

## 2. Ca khó mới (từ lỗi của máy, box thừa, QC)

| Ảnh | Tình huống | Máy làm gì (model nào) | Quyết định của bạn | Quy tắc mới (vào mục nào của v1.0) |
|---|---|---|---|---|
| img_013, img_002 | Tuk-tuk có mui chở khách | YOLO26 gọi `car` (0.42) hoặc không có box; YOLOE bắt đúng (0.93) nhưng box lỏng | Luôn `three_wheeler`, siết box khi review (CAL-01) | mục 4.3: xe có mui 3 bánh không bao giờ là `car` |
| img_003, img_006, img_024, img_027 | Box conf thấp trên vệt mờ / cụm đỗ / mép ảnh | Ngưỡng thấp (motorbike 0.1, three_wheeler 0.15) sinh box thừa | Xoá box conf < 0.3 không thấy vật rõ; vệt mờ / cụm dày dùng `ignore_region` (CAL-02) | mục 4.1 |
| img_013 | Xe van trắng ở nền | YOLO26 gán `truck` 0.47 | Là `car` (CAL-04) | mục 2 |

## 3. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do (ca nào, `case_id`) |
|---|---|---|
| v1.0 | Bản giảng viên | — |
| v1.1 | Chốt câu 2, 5; thêm quy tắc xe ba bánh, box conf thấp, van = car | CAL-01 đến CAL-05 |
