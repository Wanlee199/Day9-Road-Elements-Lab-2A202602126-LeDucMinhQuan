# Ontology + CVAT setup

Bảng ontology là **source of truth** cho schema CVAT: `03_cvat_labels.json` khớp từng dòng với bảng này.

## Ontology table

| Name | Geometry | Type (class / attribute) | Allowed values | Default | Mutable? | Rationale |
|---|---|---|---|---|---|---|
| `prohibitory` | polygon | class | N/A | N/A | No | Biển báo nhóm Cấm (hình tròn viền đỏ/nền trắng hoặc đỏ). downstream task cần phân loại trực tiếp. |
| `mandatory` | polygon | class | N/A | N/A | No | Biển báo nhóm Chỉ dẫn / Hiệu lệnh (hình tròn/vuông/chữ nhật màu xanh). downstream task cần phân loại trực tiếp. |
| `danger` | polygon | class | N/A | N/A | No | Biển báo nhóm Nguy hiểm / Cảnh báo (hình tam giác đều đỉnh hướng lên, viền đỏ nền vàng/trắng). |
| `supplementary` | polygon | class | N/A | N/A | No | Biển phụ đặt phía dưới biển chính để thuyết minh thêm (chữ nhật nhỏ nền trắng viền đen). |
| `other` | polygon | class | N/A | N/A | No | Biển không thuộc 4 loại trên: biển tên đường, biển mờ/nhỏ không thể đọc loại, biển lật mặt lưng. |
| `occluded` | N/A | attribute | `true`, `false` | `false` | No | Thuộc tính của polygon. Đánh dấu `true` nếu mặt biển bị che khuất $\ge 10\%$ diện tích bởi vật cản. |
| `truncated` | N/A | attribute | `true`, `false` | `false` | No | Thuộc tính của polygon. Đánh dấu `true` nếu ranh giới biển bị cắt bởi rìa/mép của khung ảnh. |
| `blurred` | N/A | attribute | `true`, `false` | `false` | No | Thuộc tính của polygon. Đánh dấu `true` nếu mặt biển bị mờ do khoảng cách xa, out-of-focus hoặc chuyển động. |
| `image_escalate` | tag | class (image tag) | N/A | N/A | No | Tag cấp độ ảnh (Image-level tag). Đánh dấu ảnh có nghi vấn hoặc tranh chấp cần QA Lead xử lý. |

## Class hay attribute

- **Tại sao 5 nhóm biển là Class?**
  5 nhóm biển báo (`prohibitory`, `mandatory`, `danger`, `supplementary`, `other`) có ý nghĩa ngữ nghĩa hoàn toàn riêng biệt, hình dáng học đặc trưng khác nhau, và mô hình downstream (ADAS/Perception) xử lý logic lái xe riêng cho từng nhóm (ví dụ: phát hiện biển Cấm sẽ dừng/không rẽ, biển Nguy hiểm sẽ giảm tốc).
- **Tại sao Occluded, Truncated, Blurred là Attribute?**
  Các trạng thái bị che khuất, cắt mép hay mờ là thuộc tính quan sát thời điểm của cùng một instance biển báo. Nếu tách thành Class riêng (ví dụ `prohibitory_occluded`, `prohibitory_blurred`...) sẽ bùng nổ tổ hợp class (5 x 2 x 2 x 2 = 40 classes), làm giảm hiệu quả gán nhãn và dư thừa taxonomy.
- **Default value safety:**
  Tất cả 3 attribute (`occluded`, `truncated`, `blurred`) mặc định là `false`. Annotator chỉ bật sang `true` khi phát hiện điều kiện vi phạm tương ứng.

## CVAT

- **Phiên bản CVAT:** CVAT v2.x (Docker local container).
- **Tên task calibration:** `sign-seg-calib-v1`
- **Guide của task đã dán `02_guideline.md`?:** Có (đã dán bản văn bản v1 vào Task Description).
- **Nhóm dùng Track hay Shape, vì sao:** Dùng **Shape** (Polygon) vì tập dữ liệu GTSDB là các ảnh đơn lẻ tĩnh (single frames), không phải video clip theo dõi thời gian.

## Setup test

Thành viên test mở task CVAT và xác minh:
1. Đủ 5 polygon label (`prohibitory`, `mandatory`, `danger`, `supplementary`, `other`) và 1 tag label (`image_escalate`).
2. Mỗi polygon label có đúng 3 checkbox attribute: `occluded`, `truncated`, `blurred` với mặc định là `false`.
3. Công cụ vẽ Polygon hiển thị mượt mà, phím tắt `N` hoạt động để vẽ tiếp polygon mới.
