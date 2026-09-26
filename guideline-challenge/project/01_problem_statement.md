# Problem statement + downstream contract

## Bài toán

Phân vùng ngữ nghĩa (Polygon Segmentation) và phân loại 5 nhóm biển báo giao thông (`prohibitory`, `mandatory`, `danger`, `supplementary`, `other`) cùng 3 thuộc tính thuộc tính quan sát (`occluded`, `truncated`, `blurred`) cho hình ảnh giao thông thực tế thuộc tập GTSDB, tập trung xử lý chính xác ranh giới mặt biển báo, các trường hợp biển bị che khuất một phần, bị cắt mép hoặc quay lưng/mờ xa.

## Downstream contract

1. **Downstream task / model / user là ai?** 
   Mô hình Perception (Object Detection & Instance Segmentation) cho hệ thống lái xe tự động (Autonomous Driving / ADAS) cần nhận diện nhanh và chính xác loại biển báo trên đường để đưa ra quyết định điều khiển phương tiện (phanh, giảm tốc, đổi làn).
2. **Output annotation nào thực sự cần?**
   - **Geometry:** Polygon segmentation bao kín mặt trước biển báo (Signboard face).
   - **Class (5 loại):** `prohibitory` (Biển Cấm), `mandatory` (Biển Chỉ dẫn/Hiệu lệnh), `danger` (Biển Nguy hiểm), `supplementary` (Biển phụ), `other` (Khác: mặt lưng, biển tên đường, biển mờ nhỏ không đọc được).
   - **Attribute (3 dạng boolean):** `occluded` (Bị che khuất), `truncated` (Bị cắt mép), `blurred` (Bị mờ/out-of-focus).
3. **Failure nào gây hậu quả lớn nhất?**
   - **Critical Failure 1:** Bỏ sót (False Negative) biển Cấm (`prohibitory`) hoặc biển Nguy hiểm (`danger`), dẫn tới xe đi vào đường cấm hoặc gặp nguy hiểm.
   - **Critical Failure 2:** Phân loại nhầm lẫn giữa Biển Cấm và Biển Chỉ dẫn.
   - **Major Failure:** Bỏ qua thuộc tính `occluded` hoặc `truncated` khi biển thực sự bị che khuất/cắt mép, làm lệch trọng số huấn luyện mô hình.
4. **Khi ambiguity không resolve được, ai / ở đâu là escalation path?**
   Annotator gắn tag `image_escalate` cho hình ảnh trên CVAT và ghi lại `sample_id` + câu hỏi trong log để QA Lead / Domain Expert xem xét và đưa ra quyết định cuối cùng (Gold Decision).

## Scope

- **Trong scope (bắt buộc label):**
  - Mặt trước của tất cả các biển báo giao thông xuất hiện trên đường (Facing the camera/ego vehicle).
  - Biển phụ gắn dưới biển chính.
  - Các biển báo mờ, bị cắt ở rìa ảnh, hoặc bị cây cối/vật cản che khuất một phần (gán class tương ứng + attribute `blurred` / `truncated` / `occluded`).
- **Ngoài scope (ignore / không khoanh):**
  - Cột/trụ đỡ biển báo, khung giá treo kim loại.
  - Biển báo quay lưng hoàn toàn về phía camera (không nhìn thấy mặt biển) -> Gán class `other` + attribute `occluded`/`blurred` theo đúng quy tắc xử lý trường hợp nhiễu.
  - Biển hiệu quảng cáo, bảng hiệu thương mại của cửa hàng.
- **Geometry tolerance:**
  - Polygon vẽ ôm sát mép thực tế của mặt biển báo (Visible Signboard), sai số ranh giới cho phép $\le 2\text{ px}$.
  - Sai số IoU (Intersection over Union) giữa các annotator $\ge 85\%$.

## Output chấm được

- **Class label:** 1 trong 5 polygon class (`prohibitory`, `mandatory`, `danger`, `supplementary`, `other`).
- **Attributes:** Checkbox `occluded` (true/false), `truncated` (true/false), `blurred` (true/false) xuất hiện đầy đủ trong thuộc tính của từng polygon instance.
- **Escalation:** Tag `image_escalate` mức ảnh (image-level tag) trên CVAT khi gặp ảnh quá mờ không thể xác định loại biển hoặc có tranh chấp.

## Dữ liệu và giới hạn

- Nguồn ảnh: Bộ dữ liệu GTSDB (German Traffic Sign Detection Benchmark) gồm 28 ảnh mẫu giao thông đường bộ Đức (`data/gtsdb/GTS01.png` tới `GTS28.png`).
- Kích thước ảnh chuẩn: 1360 x 800 px.
- Giới hạn: Ảnh góc nhìn camera hành trình tĩnh, một số ảnh có điều kiện ánh sáng yếu, biển nhỏ ở khoảng cách xa hoặc bị che bởi tán cây.
