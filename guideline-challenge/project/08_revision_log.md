# Revision log

| Version | Đổi gì | Vì sao | Bằng chứng |
|---|---|---|---|
| v1 | Bản nháp đầu tiên | Khởi tạo bài toán Segmentation 5 class biển báo GTSDB với 3 attribute boolean | `01_problem_statement.md` v1, `03_cvat_labels.json` |
| v2 | Bổ sung 2 Edge Cases: (1) Biển chỉ hướng di chuyển tới Quận A/Quận B; (2) Biển hình thoi màu vàng (Đường ưu tiên) | Giải quyết bất đồng calibration giữa annotator về phân loại biển chỉ đường và biển ưu tiên | GTS09, GTS11, `06_calibration_report.csv` dòng 1-3 |
