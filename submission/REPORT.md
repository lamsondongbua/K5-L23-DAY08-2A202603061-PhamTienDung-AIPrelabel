# Báo cáo Day 08 — 2A202603061 - Phạm Tiến Dũng

> Một file duy nhất cho cả bài (đã gộp recipe, QC và ghi chú cá nhân). Mọi con số kèm **mẫu số** và nguồn (file nào). Viết bằng lời của bạn.

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Vì sao (dẫn `run_id` trong `model_log.csv`) |
|---|---|---|---|
| person | YOLO26m | 0.45 | `r03`, `r04`: conf 0.5 làm số person giảm 131 -> 87 (mất người nhỏ/xa), nên chọn ngưỡng vừa; sweep: recall 94%, precision 58% (16 vật gold) |
| motorbike | YOLO26m | 0.10 | sweep: recall 100% nhưng precision chỉ 33% (9 vật gold); ngưỡng thấp vì xe máy nhỏ, dày; chấp nhận box thừa để review xoá |
| car | YOLO26m | 0.30 (mặc định) | không ngưỡng nào <= 0.5 đạt recall 90% (tối đa 88%, 8 vật gold) |
| bicycle / bus / truck | YOLO26m | 0.30 (mặc định) | gold chỉ có 4 / 1 / 0 vật (< 5): mẫu quá ít, không chọn ngưỡng riêng |
| three_wheeler | YOLOE-26s, prompt: `three_wheeler` | 0.15 | `r05`: bắt đúng tuk-tuk (img_002, 0.93) khi YOLO26 không có box; sweep: recall 100%, precision 78% (7 vật gold) |

- Chuỗi ngưỡng từ `threshold_sweep.md`: `--thr "person=0.45,motorbike=0.1,three_wheeler=0.15"` (lớp còn lại dùng `--conf 0.3`). Lệnh `merge_prelabels.py` đầy đủ: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "person=0.45,motorbike=0.1,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- **Phiên bản SAM bạn chọn** (SAM 1 / 2.1 / 3) và lý do: SAM 2.1 base (`sam2.1_b.pt`, `r06`); mask bám tốt người ngồi trong xe, nhưng chậm ~7.95 giây/ảnh trên CPU, viền lởm chởm ở gầm/chắn bùn.
- Lỗi nào **không sửa được bằng đổi ngưỡng** (ví dụ tuk-tuk bị gọi `car`)? Sửa thế nào? YOLO26 là closed-set: ở img_013 gọi tuk-tuk là `car` (0.42), ở img_002 không có box cho tuk-tuk; hạ ngưỡng không tạo ra lớp mới. Dùng YOLOE (prompt `three_wheeler`) làm nguồn riêng, rồi sửa nhãn khi review.
- Box **điểm cao mà sai** hoặc **điểm thấp mà đúng** (ảnh nào): điểm thấp mà sai: img_013 tuk-tuk gọi `car` 0.42; img_002 YOLOE báo nhầm tuk-tuk 0.37-0.40 trên xe máy và pickup; điểm cao mà lỏng: img_002 tuk-tuk 0.93 đúng lớp nhưng box rộng hơn thân xe; img_013 xe van nền gọi `truck` 0.47 (sai lớp).
- Giới hạn: chọn ngưỡng trên chính gold (4 ảnh) nên số đo máy trên gold là lạc quan: precision / recall ở trên là in-sample; chỉ 4 ảnh, lớp có 1-9 vật nên đổi một nhãn gold cũng làm số đổi (ví dụ car 78% -> 88% recall tối đa sau khi sửa một nhãn).

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted | 429 | 77,0% | 557 box máy |
| edited | 45 | 8,1% | 557 box máy |
| deleted | 77 | 13,8% | 557 box máy |
| added | 22 | 4,4% | 496 đối tượng sau duyệt (trong đó 10 là đổi lớp, ~12 là máy sót thật) |

- Lớp nào máy tệ nhất? Vì sao (dẫn ảnh)? three_wheeler: xoá 20/41 box (13 giữ nguyên, 8 sửa); motorbike: xoá 43 box (ngưỡng 0.1 thấp, ảnh dày xe máy). Dẫn ảnh: img_006 và img_024 (cụm xe máy đỗ / vệt mờ ban đêm, nhiều box conf < 0.3 chồng nhau), img_003 (three_wheeler conf 0.18-0.25 ở mép ảnh).
- So với gold: precision / recall của **máy** 55,1% (KTC 95%: 44–66%) / 95,6% (KTC 95%: 85–99%) (mẫu số: 45 đối tượng gold, 4 ảnh; lạc quan vì chọn ngưỡng trên chính gold)

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt bẫy (lần đo 2): 11/12 = 91,7% (KTC 95%: 65–99%); loại nào lọt: `missing` ở img_003 (2/3 bẫy missing bắt được; loose 3/3, extra 3/3, wrong_class 3/3); vì sao (bạn đã bỏ bước nào của checklist)? img_003 có nhiều vật nhỏ ở nền xa; bẫy `missing` là box bị xoá nên không có gì hiển thị để nhận ra; nhiều khả năng đã bỏ bước quét ảnh trước khi nhìn nhãn máy và đếm vật (bước 0 của checklist).
- Lần đo 1 (trước khi duyệt lại): 8/12 = 66,7%, lọt 3 bẫy `missing` (img_026, 003, 029) và 1 bẫy `loose` (img_035).

## 4. Máy có đáng dùng không?

- Thời gian (từ `time_log.csv`): gán tay 15 phút / 4 ảnh = 3,75 phút/ảnh vs duyệt 110 phút / 24 ảnh = 4,6 phút/ảnh. Thiên lệch (bạn gán gold trước nên đã quen cảnh): gán tay chỉ 4 ảnh, 24 ảnh batch dày hơn và có 12 bẫy; thời gian duyệt gồm cả việc xoá nhiều box thừa nên so sánh chỉ mang tính tham khảo.
- Kết luận: chỉ cho person và car, không cho motorbike / three_wheeler ở ngưỡng hiện tại, dựa trên: máy recall 95,6% trên gold nhưng precision 55,1% (45 vật gold); duyệt 4,6 phút/ảnh không nhanh hơn gán tay 3,75 phút/ảnh; three_wheeler bị xoá 20/41 box, motorbike xoá 43; QC mẫu còn 5/8 ảnh lỗi chủ yếu là box thừa.
- **Ba quy tắc giảm thiên kiến của bạn** (mỗi quy tắc kiểm tra được bằng số hoặc việc làm):
  1. Ẩn nhãn máy và tự đếm vật của từng ảnh trước khi bật nhãn máy: kiểm bằng log đếm đủ 24/24 ảnh, mục tiêu bắt đủ 3/3 bẫy `missing`.
  2. Lọc theo conf và xem lại mọi box conf < 0.3: kiểm bằng số box conf < 0.3 còn lại trong `batch_reviewed.json` bằng 0 hoặc có ghi chú.
  3. Mỗi ảnh quét 2 lượt (nền xa rồi nền gần) trước khi chuyển ảnh: kiểm bằng cột log 24/24 ảnh và bẫy `loose` / `missing` bắt ≥ 5/6 ở lần duyệt đầu.

## 5. Mask SAM cho 3 vật (Part 5)

| Ảnh | SAM (phiên bản) | Số click | Mask gồm người lái / hàng hoá? | Lỗi quan sát | File trong `masks/` |
|---|---|---|---|---|---|
| img_013 | SAM 2.1 base (`sam2.1_b.pt`) | 3 (2 dương, 1 âm ở người lái) | Có: mask dính cả người lái và một phần hành khách | Click âm 1 điểm không tách được người lái; viền lởm chởm ở chắn bùn/bánh trước | `tuktuk_img013_sam21.jpg` |
| img_013 | SAM 2.1 base | 5 (2 dương, 3 âm quanh người lái) | Gần như loại người lái nhưng mask lốm đốm ở vùng áo/tay, hành khách vẫn dính một phần | Thêm click âm giúp nhưng mask rách, không sạch | `tuktuk_img013_sam21_neg3.jpg` |
| img_013 | MobileSAM (`mobile_sam.pt`) | 3 (2 dương, 1 âm) | Không: loại người lái và hành khách, nhưng mất một phần buồng lái phía trước | Tách người tốt hơn SAM 2.1 với cùng click, nhưng cắt mất vùng cabin/kính trước (2 polygon) | `tuktuk_img013_mobilesam.jpg` |
| img_004 | SAM 2.1 base | 3 (2 dương, 1 âm ở tuk-tuk xanh bên cạnh) | Không có người lái trong mask; mask gọn theo thân xe xanh-cam | Sạch; biển TAXI trên mái bị khuyết một phần; xe che khuất bởi xe khác nên phải thêm click âm | `tuktuk_img004_sam21.jpg` |
| img_017 | SAM 2.1 base | 3 (2 dương, 1 âm ở xe bên trái) | Không có người; mask gồm cả khoang ghế/hàng phía sau | Tách tốt khỏi xe liền kề; viền bám sát thân và bánh xe | `tuktuk_img017_sam21.jpg` |

- Đối chiếu guideline mục 4.2: quy tắc nào giúp, quy tắc nào thiếu? mục 4.2 giúp ở điểm bám biên phần nhìn thấy và không vẽ bóng đổ; thiếu quy tắc về người ngồi trong xe (SAM 2.1 dính người lái ở img_013, MobileSAM loại được) và kính chắn gió, nên v1.1 bổ sung: mask xe không gồm người ngồi trong xe (CAL-05).
- *(Tuỳ chọn)* Interactor SAM trong CVAT: làm được / không (lỗi gì, đã thử gì); không làm (tuỳ chọn): chỉ dùng `sam2_points.py` offline với click dương/âm tương tự Interactor.

## 6. Calibration (Part 6)

- Số quyết định: 5 (`calibration_decisions.csv`: 3 `loi_may`, 2 `cau_hoi_mo`); quan trọng nhất (dẫn `CAL-xx`): CAL-01 (tuk-tuk bị gọi `car`) và CAL-02 (box conf thấp gây 7/8 lỗi QC).
- Quy tắc mới đã vào `guideline_v1.1_calibrated.md`: xe ba bánh có mui luôn là `three_wheeler`; box conf < 0.3 không thấy vật rõ thì xoá, vệt mờ / cụm dày dùng `ignore_region`; van chở khách = `car`; mask xe không gồm người ngồi trong xe. Chốt câu hỏi mở 2 và 5; 5 câu còn lại chưa chốt.
- Nguyên nhân gốc: chủ yếu là **máy** (ngưỡng thấp) và **cách duyệt** (để lọt box conf thấp), không phải guideline mơ hồ; một phần nhỏ là guideline thiếu (câu 2, 5).

## 7. QC và quyết định (Part 6)

- Mẫu: 8 ảnh (3 ảnh `can_review` + 5 ngẫu nhiên, seed 42), 191 đối tượng; người kiểm: bạn (**hạn chế**: bạn vừa là người duyệt; phiếu `qc_review.csv` là nháp do AI điền từ ảnh `work/qc/review/`, cần bạn xác nhận).
- Lỗi theo **đối tượng**: 8/191 = 4,2% · theo **ảnh**: 5/8 = 62,5% (ngưỡng Release ≤ 2%)
- Top lỗi: thừa 7 (img_003, img_006, img_013, img_024, img_027), sai lớp 1 (img_013: van gán truck); bỏ sót, sai vị trí, sai thuộc tính, sai mask: 0 quan sát được.
- **Giới hạn mẫu nhỏ** (cận trên 95% trong `qc_report_auto.md`): 5/8 ảnh lỗi, cận trên Wilson 95% = 86%; ngưỡng 2% không thể được chứng minh bằng 8 ảnh.
- Script kiểm được / không kiểm được (ví dụ ôm sát, cùng bỏ sót với máy, theo checklist thật): script chỉ đếm theo phiếu; không kiểm được box có ôm sát không và vật cả tôi lẫn máy cùng bỏ sót.

**Quyết định: Rework**, ngày 2026-11-18. Lý do (đi theo Hold → Rework → Release): guideline không phải nguyên nhân chính nên không Hold; ảnh lỗi 5/8 = 62,5% > 2% và lỗi `thua` lặp lại (7/8 lỗi) nên Rework; chưa thể Release.
Nếu Rework / Hold, việc cần làm tiếp: xoá các box thừa conf < 0.3 trong CVAT, đổi `ignore_region` cho vệt mờ / cụm dày, đổi truck -> car ở img_013, export `qc_fixed.json`, chạy lại `build_release.py` và `qc_sample.py --force`, đồng thời tăng ngưỡng `motorbike` / `three_wheeler`.

## 8. Ghi chú cá nhân

- **Dùng AI để làm bài** (công cụ nào, việc gì, kiểm tra thế nào; "không dùng" nếu không): Claude Code (trợ lý AI) chạy các lệnh script, chọn điểm click SAM, soạn nháp nhận xét model_log, phiếu `qc_review.csv` và các mục REPORT.
- Máy / GPU / CVAT dùng; Windows 11, PowerShell/Git Bash, CPU (torch 2.15.0+cpu, `check_env.py` báo không có GPU), CVAT Docker 2.81; vướng: terminal VS Code chạy `python` ngoài `.venv`, và Task `batch-review` báo `Label 'motorbike' is not registered` do Task chưa thuộc Project có đủ 8 nhãn.
- Dùng cache `data/cache/` thay vì tự chạy? không, tự chạy `prelabel.py` cho cả 36 ảnh.
- Lỗi / bug script đã gặp file export đặt tên `batch_review.json` thay vì `batch_reviewed.json` làm `review_stats.py` báo thiếu file; gõ thiếu `python scripts/` khi chạy `validate_labels.py`.
- Một điều bạn làm khác lần sau: chọn ngưỡng `motorbike` / `three_wheeler` cao hơn ngay từ đầu, và đếm vật trước khi nhìn nhãn máy để không để lọt bẫy `missing`.
