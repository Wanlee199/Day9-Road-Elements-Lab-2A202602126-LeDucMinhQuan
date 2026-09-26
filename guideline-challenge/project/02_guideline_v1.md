# Annotation Guideline — Traffic Sign Segmentation (GTSDB)

**Version:** v1

---

## 1. Objective + Scope

### 1.1 Mục tiêu (Objective)
Tài liệu này hướng dẫn chi tiết quy trình đánh nhãn phân vùng ngữ nghĩa (Polygon Segmentation / Mask) và phân loại 5 nhóm biển báo giao thông cùng 3 thuộc tính trạng thái cho tập dữ liệu GTSDB. Dữ liệu gán nhãn sẽ phục vụ huấn luyện mô hình nhận diện biển báo cho hệ thống trợ lái và xe tự hành (ADAS/Autonomous Driving).

### 1.2 Phạm vi (Scope)
- **Trong phạm vi (In scope):** Tất cả các biển báo giao thông đường bộ hướng mặt trước về phía xe/camera xuất hiện trong khung ảnh, bao gồm cả biển báo mờ, bị che khuất một phần, bị cắt ở viền ảnh, biển phụ đi kèm.
- **Ngoài phạm vi (Out of scope):** Cột/trụ đỡ biển báo, giá treo kim loại, biển hiệu quảng cáo thương mại, biển tên cửa hàng.

---

## 2. Annotation Unit

- **Loại nhãn:** Instance Segmentation (Polygon hoặc Brush Mask - Kiểu nhãn `any`).
- **Quy tắc định danh Instance mới:**
  - Mỗi mặt biển báo hình tròn, tam giác, hình chữ nhật hoặc hình vuông là **1 Instance độc lập**.
  - Đối với chùm biển báo lắp trên cùng một cột đỡ (ví dụ: 1 biển Cấm phía trên và 1 biển Phụ phía dưới): **Vẽ 2 Polygon/Mask tách biệt**, KHÔNG gộp chung hai biển thành một hình.
  - Không vẽ nối liền các biển báo bị ngăn cách bởi vật cản.

---

## 3. Geometry Rule

- **Công cụ gắn nhãn:** Công cụ **Polygon** hoặc **Draw a mask (Brush/Cọ tô)** trên CVAT.
- **Quy tắc viền ranh giới (Visible Contour):**
  - Chỉ khoanh phần **mặt biển báo thực sự nhìn thấy được** (Visible Signboard Face).
  - **KHÔNG khoanh cột/trụ đỡ** biển báo.
  - **KHÔNG khoanh phần bị che khuất** (Không dùng phương pháp Amodal segmentation, không tự ước lượng đường cong bị lá cây hay cột đèn che).
  - **KHÔNG khoanh biển báo quay mặt lưng** (mặt sau xám/nền kim loại không nhìn thấy mặt biển). Nếu biển báo bị quay mặt lưng hoặc nghiêng > 80 độ làm mất hoàn toàn hình dạng mặt biển -> Phân loại vào class `other` + bật thuộc tính `occluded=true` / `blurred=true`.
- **Mật độ điểm nút / Cọ tô:**
  - Đặt các điểm nút ôm sát đường viền mặt biển báo.
  - Biển hình tam giác/chữ nhật/vuông: Đặt các điểm tại chính xác các đỉnh góc.
  - Biển hình tròn: Đặt từ 8 đến 12 điểm nút trải đều trên chu vi để đảm bảo đường cong tròn mượt, không tạo hình đa giác méo mó.
- **Dung sai (Tolerance):** Sai số ranh giới Polygon/Mask cho phép $\le 2\text{ px}$ so với viền thực của mặt biển.

---

## 4. Taxonomy

### Bảng Phân loại Class (5 Classes)

| Class Name | Tên Tiếng Việt | Dấu hiệu nhận biết & Hình dạng |
|---|---|---|
| `prohibitory` | Biển Cấm | Hình tròn viền đỏ nền trắng/vàng (hoặc hình tròn nền đỏ chữ trắng như biển STOP, cấm đi ngược chiều). |
| `mandatory` | Biển Chỉ dẫn / Hiệu lệnh | Hình tròn, hình vuông hoặc hình chữ nhật màu xanh lam (Blue) chỉ hướng đi, làn đường, lối dành cho người đi bộ. |
| `danger` | Biển Nguy hiểm / Cảnh báo | Hình tam giác đều, đỉnh hướng lên trên, viền đỏ, nền vàng hoặc trắng chứa biểu tượng cảnh báo nguy hiểm phía trước. |
| `supplementary` | Biển Phụ | Hình chữ nhật nhỏ nền trắng viền đen, đặt ngay bên dưới biển chính để thuyết minh khoảng cách, thời gian, loại xe. |
| `other` | Khác / Không xác định | Biển không thuộc 4 loại trên: biển mờ xa không đọc được loại, biển bị che khuất > 80%, biển quay mặt lưng, biển tên đường / số hiệu đường. |

### Bảng Thuộc tính Attribute (3 Boolean Attributes)

Tất cả 3 thuộc tính bên dưới áp dụng cho từng Instance, dạng **Checkbox (true / false)** với giá trị mặc định là `false`:

| Attribute Name | Mô tả điều kiện bật `true` | Ví dụ cụ thể |
|---|---|---|
| `occluded` | Mặt biển báo bị vật cản (cây cối, xe khác, cột đèn, biển khác) che khuất $\ge 10\%$ diện tích. | Cành cây che mất một phần số speed limit trên biển cấm. |
| `truncated` | Ranh giới mặt biển báo bị đường mép/rìa ảnh cắt đứt một phần. | Biển báo nằm ở góc trên bên trái ảnh chỉ xuất hiện một nửa. |
| `blurred` | Mặt biển báo bị mờ do khoảng cách xa, độ phân giải thấp, rung camera hoặc điều kiện thời tiết (mưa, sương mù). | Biển báo ở xa $\ge 50\text{m}$ bị vỡ hạt pixel, không nhìn rõ chi tiết biểu tượng. |

---

## 5. Inclusion / Exclusion

### Bắt buộc gán nhãn (Inclusion):
1. Tất cả biển báo giao thông rõ ràng thuộc 4 nhóm `prohibitory`, `mandatory`, `danger`, `supplementary`.
2. Biển báo bị che một phần hoặc bị cắt rìa ảnh (gán class tương ứng + bật attribute `occluded=true` / `truncated=true`).
3. Biển báo bị mờ nhưng vẫn phân biệt được loại dựa vào màu sắc & hình dáng (gán class tương ứng + bật `blurred=true`).
4. Biển báo quay mặt lưng, biển quá nhỏ/mờ không nhận diện được loại, biển tên đường (gán class `other`).

### Bỏ qua - Không gán nhãn (Exclusion / Ignore):
1. Cột đỡ, chân đế, xà ngang treo biển báo.
2. Biển quảng cáo thương mại, băng rôn thương hiệu cửa hàng.
3. Vùng biến dạng lóa sáng hoàn toàn không còn dấu vết hình học của biển báo.

---

## 6. Visibility / Occlusion

1. **Che khuất một phần ($10\% \le \text{Diện tích bị che} \le 80\%$):**
   - Khoanh Polygon/Mask theo chính xác phần mặt biển nhìn thấy được.
   - Bật `occluded = true`.
2. **Che khuất nặng ($> 80\%$ diện tích mặt biển):**
   - Không thể suy đoán chính xác class chính -> Gán class `other` + bật `occluded = true`.
3. **Mờ / Out of focus:**
   - Nếu vẫn rõ hình dáng (tròn/tam giác/xanh/đỏ) -> Gán class đúng + bật `blurred = true`.
   - Nếu mờ vỡ hạt pixel hoàn toàn -> Gán class `other` + bật `blurred = true`.
4. **Bị cắt ở rìa ảnh (Truncated):**
   - Vẽ ranh giới Polygon/Mask theo đúng phần mặt biển còn lại trong ảnh.
   - Bật `truncated = true`.

---

## 7. Ambiguity / Escalation

Khi gặp các trường hợp ranh giới mơ hồ, tranh chấp hoặc không đủ dữ kiện xác định:

1. **Trường hợp tranh chấp Class:**
   - Nếu không chắc chắn biển thuộc loại `prohibitory`, `mandatory` hay `danger` và không ai trong nhóm đồng thuận -> Phân loại tạm thời vào `other`, bật `blurred=true` hoặc `occluded=true`.
2. **Quy trình Leo thang (Escalation Path trên CVAT):**
   - Annotator bấm chọn công cụ **Setup tag** -> Chọn tag `image_escalate` cho hình ảnh.
   - QA Lead sẽ kiểm tra lại toàn bộ danh sách ảnh bị tag `image_escalate` để ra quyết định đóng Gold Decision.

---

## 8. Temporal Rule

Task này thực hiện trên dữ liệu ảnh tĩnh (Single frames GTSDB).
- **Áp dụng:** Không áp dụng quy tắc Temporal / Tracking qua nhiều frame.

---

## 9. Examples

| sample_id | Đối tượng / Hiện trạng | Expected Output (Class + Attributes) | Rule áp dụng |
|---|---|---|---|
| GTS01 | Biển cấm tốc độ 30 km/h rõ ràng ở bên phải đường | Class: `prohibitory`<br>Attributes: `occluded=false`, `truncated=false`, `blurred=false` | Mục 4 - Biển Cấm chuẩn |
| GTS02 | Biển cảnh báo tam giác viền đỏ nằm sát mép phải ảnh bị cắt một phần | Class: `danger`<br>Attributes: `occluded=false`, `truncated=true`, `blurred=false` | Mục 4 & 6 - Truncated |
| GTS05 | Biển báo tròn xanh chỉ dẫn rẽ phải bị tán cây che góc trên | Class: `mandatory`<br>Attributes: `occluded=true`, `truncated=false`, `blurred=false` | Mục 4 & 6 - Occluded |
| GTS08 | Cụm 1 biển cấm 50km/h và 1 biển phụ bên dưới | 2 Polygon/Mask tách biệt:<br>1. Class `prohibitory`<br>2. Class `supplementary` | Mục 2 - Instance separation |
| GTS12 | Biển báo ở xa mờ vỡ pixel không đọc được nội dung | Class: `other`<br>Attributes: `occluded=false`, `truncated=false`, `blurred=true` | Mục 4 & 6 - Blurred & Other |
| GTS18 | Biển báo quay mặt lưng màu xám về phía camera | Class: `other`<br>Attributes: `occluded=true`, `truncated=false`, `blurred=false` | Mục 3 & 5 - Rear facing |

---

## 10. Common Mistakes & Quality Assurance

1. **Lỗi 1: Khoanh gộp cả Cột/Trụ đỡ biển báo.**
   - *Cách khắc phục:* Chỉ vẽ polygon/mask bao quanh phần viền tấm kim loại mặt biển báo (Signboard), cắt bỏ phần cột thép bên dưới.
2. **Lỗi 2: Gộp chung Biển Chính và Biển Phụ vào 1 Polygon/Mask.**
   - *Cách khắc phục:* Tách thành 2 nhãn riêng biệt (1 class chính, 1 class `supplementary`).
3. **Lỗi 3: Vẽ Polygon/Mask tràn ra ngoài viền biển quá rộng (Răng cưa/dư thừa không gian).**
   - *Cách khắc phục:* Giảm Opacity trong tab Appearance trên CVAT để nhìn rõ viền thực tế của biển, bám sát mép biển ($\le 2\text{ px}$).
4. **Lỗi 4: Quên bật Attribute Checkbox khi biển bị che hoặc mờ.**
   - *Cách khắc phục:* Sử dụng chế độ **Attribute Annotation mode** trên CVAT sau khi vẽ xong để rà soát từng đối tượng.
5. **Tiêu chuẩn QA:**
   - IoU giữa các Annotator $\ge 85\%$.
   - Tỷ lệ đúng Class = 100% đối với các biển rõ ràng.
