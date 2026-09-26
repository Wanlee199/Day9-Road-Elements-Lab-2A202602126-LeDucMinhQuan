# Annotation Guideline — Traffic Sign Segmentation (GTSDB)

**Version:** v2

---

## 1. Objective + Scope

### 1.1 Mục tiêu (Objective)
Tài liệu này hướng dẫn chi tiết quy trình đánh nhãn phân vùng ngữ nghĩa (Polygon / Brush Mask - Kiểu nhãn `any`) và phân loại 5 nhóm biển báo giao thông cùng 3 thuộc tính trạng thái cho tập dữ liệu GTSDB. Dữ liệu gán nhãn phục vụ huấn luyện mô hình nhận diện biển báo cho hệ thống trợ lái và xe tự hành (ADAS/Autonomous Driving).

### 1.2 Phạm vi (Scope)
- **Trong phạm vi (In scope):** Tất cả các biển báo giao thông đường bộ hướng mặt trước về phía xe/camera xuất hiện trong khung ảnh, bao gồm biển báo mờ, bị che khuất một phần, bị cắt ở viền ảnh, biển phụ, biển chỉ hướng đi và biển ưu tiên.
- **Ngoài phạm vi (Out of scope):** Cột/trụ đỡ biển báo, giá treo kim loại, biển hiệu quảng cáo thương mại, biển tên cửa hàng.

---

## 2. Annotation Unit

- **Loại nhãn:** Instance Segmentation (Polygon hoặc Brush Mask - Kiểu nhãn `any`).
- **Quy tắc định danh Instance mới:**
  - Mỗi mặt biển báo hình tròn, tam giác, hình chữ nhật, hình thoi hoặc hình vuông là **1 Instance độc lập**.
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
  - Biển hình tam giác/chữ nhật/vuông/hình thoi: Đặt các điểm tại chính xác các đỉnh góc.
  - Biển hình tròn: Đặt từ 8 đến 12 điểm nút trải đều trên chu vi để đảm bảo đường cong tròn mượt.
- **Dung sai (Tolerance):** Sai số ranh giới Polygon/Mask cho phép $\le 2\text{ px}$ so với viền thực của mặt biển.

---

## 4. Taxonomy & Edge-Case Class Rules

### Bảng Phân loại Class (5 Classes)

| Class Name | Tên Tiếng Việt | Dấu hiệu nhận biết & Hình dạng |
|---|---|---|
| `prohibitory` | Biển Cấm | Hình tròn viền đỏ nền trắng/vàng (hoặc hình tròn nền đỏ chữ trắng như biển STOP, cấm đi ngược chiều). |
| `mandatory` | Biển Chỉ dẫn / Hiệu lệnh / Ưu tiên | Hình tròn, hình vuông hoặc hình chữ nhật màu xanh lam (Blue) chỉ hướng đi/làn đường. **ĐẶC BIỆT:** Bao gồm biển hình thoi màu vàng viền trắng (Biển đường ưu tiên - Priority Road) và biển chỉ hướng di chuyển tới Địa danh/Quận. |
| `danger` | Biển Nguy hiểm / Cảnh báo | Hình tam giác đều, đỉnh hướng lên trên, viền đỏ, nền vàng hoặc trắng chứa biểu tượng cảnh báo nguy hiểm phía trước. |
| `supplementary` | Biển Phụ | Hình chữ nhật nhỏ nền trắng viền đen, đặt ngay bên dưới biển chính để thuyết minh khoảng cách, thời gian, loại xe. |
| `other` | Khác / Không xác định | Biển không thuộc 4 loại trên: biển mờ xa không đọc được loại, biển bị che khuất > 80%, biển quay mặt lưng, biển tên đường/tên phố không chỉ hướng giao thông. |

---

## 5. Quy tắc xử lý các trường hợp đặc biệt (Edge Cases mới bổ sung v2)

### Edge Case 1: Biển chỉ hướng đi / Biển tên đường dạng mũi tên (Directional & Arrow Street Signs)
- **Mô tả:** Biển báo chỉ hướng di chuyển (ví dụ: *Đi thẳng tới Quận A, rẽ phải tới Quận B*) hoặc **biển tên đường có hình dạng vót nhọn mũi tên ở một đầu** (Arrow-shaped street sign, phổ biến ở Đức/Châu Âu như biển *Zeichen 415/437* chỉ hướng rẽ vào phố).
- **Quy tắc phân loại cụ thể:**
  1. **Tất cả các biển CÓ HÌNH MŨI TÊN hoặc BẢN THÂN TẤM BIỂN DẠNG MŨI TÊN VÓT NHỌN** $\rightarrow$ Bắt buộc phân loại vào **`mandatory`** (Biển chỉ dẫn/hướng di chuyển). Vì nó mang chức năng chỉ dẫn phương tiện rẽ theo hướng đó.
  2. **Chỉ các biển tên đường hình chữ nhật phẳng thuần túy** (không có mũi tên, chỉ gắn cố định báo tên vị trí) $\rightarrow$ Mới phân loại vào **`other`**.

### Edge Case 2: Biển báo hình thoi - Biển Đường Ưu Tiên (Priority Road Sign - German Sign 306)
- **Mô tả:** Biển hình thoi (Diamond shape) có viền ngoài màu trắng, hình thoi bên trong màu vàng.
- **Giải thích ngữ nghĩa:** Đây là biển báo hiệu đoạn đường được quyền ưu tiên lưu thông qua các nút giao (Vorfahrtsstraße).
- **Quy tắc phân loại:** 
  - Phân loại vào **`mandatory`** (Chỉ dẫn / Quy tắc ưu tiên giao thông). 
  - **Lưu ý:** Không nhầm lẫn biển hình thoi này với biển Nguy hiểm (`danger` bắt buộc phải là hình tam giác viền đỏ).

---

## 6. Inclusion / Exclusion

### Bắt buộc gán nhãn (Inclusion):
1. Tất cả biển báo giao thông rõ ràng thuộc 4 nhóm `prohibitory`, `mandatory`, `danger`, `supplementary`.
2. Biển đường ưu tiên hình thoi màu vàng (gán `mandatory`).
3. Biển chỉ hướng đi tới các Quận/Địa danh (gán `mandatory`).
4. Biển báo bị che một phần hoặc bị cắt rìa ảnh (gán class tương ứng + bật attribute `occluded=true` / `truncated=true`).
5. Biển báo bị mờ nhưng vẫn phân biệt được loại (gán class tương ứng + bật `blurred=true`).
6. Biển báo quay mặt lưng, biển tên phố, biển quá nhỏ/mờ không nhận diện được loại (gán class `other`).

### Bỏ qua - Không gán nhãn (Exclusion / Ignore):
1. Cột đỡ, chân đế, xà ngang treo biển báo.
2. Biển quảng cáo thương mại, băng rôn thương hiệu cửa hàng.
3. Vùng biến dạng lóa sáng hoàn toàn không còn dấu vết hình học của biển báo.

---

## 7. Ambiguity / Escalation

1. **Tranh chấp giữa Class `mandatory` và Class `other` đối với Biển Chỉ Đường:**
   - Nếu biển chỉ ghi tên địa danh/quận mà không chắc chắn có phải chỉ hướng giao thông hay không $\rightarrow$ Ưu tiên xem xét có mũi tên chỉ hướng không. Nếu có mũi tên chỉ hướng rẽ/thẳng $\rightarrow$ `mandatory`. Nếu chỉ là bảng tên thuần túy $\rightarrow$ `other`.
2. **Quy trình Leo thang (Escalation Path trên CVAT):**
   - Annotator bấm chọn công cụ **Setup tag** -> Chọn tag `image_escalate` cho toàn bộ bức ảnh khi gặp trường hợp tranh chấp không chốt được.

---

## 8. Temporal Rule

Task này thực hiện trên dữ liệu ảnh tĩnh (Single frames GTSDB).
- **Áp dụng:** Không áp dụng quy tắc Temporal / Tracking qua nhiều frame.

---

## 9. Examples & Edge Case Matrix

| sample_id | Đối tượng / Hiện trạng | Expected Output (Class + Attributes) | Rule áp dụng |
|---|---|---|---|
| GTS01 | Biển cấm tốc độ 30 km/h rõ ràng ở bên phải đường | Class: `prohibitory`<br>Attributes: `occluded=false`, `truncated=false`, `blurred=false` | Mục 4 - Biển Cấm chuẩn |
| GTS02 | Biển cảnh báo tam giác viền đỏ nằm sát mép phải ảnh bị cắt một phần | Class: `danger`<br>Attributes: `occluded=false`, `truncated=true`, `blurred=false` | Mục 4 & 6 - Truncated |
| GTS05 | Biển báo tròn xanh chỉ dẫn rẽ phải bị tán cây che góc trên | Class: `mandatory`<br>Attributes: `occluded=true`, `truncated=false`, `blurred=false` | Mục 4 & 6 - Occluded |
| GTS08 | Cụm 1 biển cấm 50km/h và 1 biển phụ bên dưới | 2 Polygon/Mask tách biệt:<br>1. Class `prohibitory`<br>2. Class `supplementary` | Mục 2 - Instance separation |
| GTS09 | Biển hình thoi màu vàng viền trắng (Đường ưu tiên) | Class: `mandatory`<br>Attributes: `occluded=false`, `truncated=false`, `blurred=false` | Mục 5 - Edge Case 2 (Biển hình thoi) |
| GTS11 | Biển màu xanh chỉ hướng di chuyển thẳng đi Quận A, rẽ phải đi Quận B | Class: `mandatory`<br>Attributes: `occluded=false`, `truncated=false`, `blurred=false` | Mục 5 - Edge Case 1 (Biển chỉ đường) |
| GTS12 | Biển báo ở xa mờ vỡ pixel không đọc được nội dung | Class: `other`<br>Attributes: `occluded=false`, `truncated=false`, `blurred=true` | Mục 4 & 6 - Blurred & Other |
| GTS18 | Biển báo quay mặt lưng màu xám về phía camera | Class: `other`<br>Attributes: `occluded=true`, `truncated=false`, `blurred=false` | Mục 3 & 5 - Rear facing |

---

## 10. Common Mistakes & Quality Assurance

1. **Lỗi 1: Nhầm lẫn Biển Đường Ưu Tiên hình thoi vàng sang Class `danger`.**
   - *Cách khắc phục:* Nhớ rằng biển Nguy hiểm (`danger`) bắt buộc phải là hình tam giác viền đỏ. Biển hình thoi vàng thuộc nhóm `mandatory`.
2. **Lỗi 2: Nhầm lẫn Biển chỉ đường có mũi tên chỉ hướng sang Class `other`.**
   - *Cách khắc phục:* Các biển chỉ hướng di chuyển tới Quận A/Quận B có kèm mũi tên/làn xe được xếp vào nhóm `mandatory` (Chỉ dẫn).
3. **Lỗi 3: Khoanh gộp Cột/Trụ đỡ biển báo.**
   - *Cách khắc phục:* Chỉ vẽ bám sát viền tấm kim loại mặt biển báo ($\le 2\text{ px}$).
4. **Tiêu chuẩn QA:**
   - IoU giữa các Annotator $\ge 85\%$.
   - Tỷ lệ đúng Class = 100% đối với các biển rõ ràng.
