# Edge-case library

Tối thiểu **8 card**, khuyến nghị 10–12.

---

CASE ID: EC01
Sample: GTS09
Scene: Đường giao lộ ban ngày
Observation: Thấy biển báo hình thoi màu vàng có viền màu trắng ở bên phải đường
Decision: LABEL
Expected: class: mandatory, occluded: false, truncated: false, blurred: false
Rationale: Biển hình thoi màu vàng là biển Đường Ưu Tiên (Priority Road / German Sign 306). Theo taxonomy thuộc nhóm hiệu lệnh/chỉ dẫn quyền ưu tiên (mandatory).
Common mistake: Nhầm sang class danger vì tưởng là biển cảnh báo nguy hiểm. Biển nguy hiểm bắt buộc phải là hình tam giác viền đỏ.
Diversity: ambiguous semantics

---

CASE ID: EC02
Sample: GTS11
Scene: Đường quốc lộ giao lộ
Observation: Thấy bảng hiệu màu xanh lam ghi hướng di chuyển đi thẳng tới Quận A, rẽ phải tới Quận B
Decision: LABEL
Expected: class: mandatory, occluded: false, truncated: false, blurred: false
Rationale: Biển chỉ hướng di chuyển tới các địa danh/quận hướng dẫn luồng giao thông thuộc nhóm chỉ dẫn (mandatory).
Common mistake: Nhầm sang class other vì tưởng là biển tên đường thông thường.
Diversity: ambiguous semantics

---

CASE ID: EC03
Sample: GTS02
Scene: Mép phải khung ảnh
Observation: Biển tam giác cảnh báo nguy hiểm nằm sát ranh giới phải của ảnh và bị mép ảnh cắt đứt 1 phần
Decision: LABEL
Expected: class: danger, occluded: false, truncated: true, blurred: false
Rationale: Ranh giới biển bị đường viền khung ảnh cắt đứt nên phải bật thuộc tính truncated=true.
Common mistake: Quên bật attribute truncated=true.
Diversity: truncation

---

CASE ID: EC04
Sample: GTS05
Scene: Tuyến đường có nhiều cây xanh
Observation: Biển tròn màu xanh chỉ dẫn rẽ phải bị tán cây đè lên góc trên khoảng 25% diện tích
Decision: LABEL
Expected: class: mandatory, occluded: true, truncated: false, blurred: false
Rationale: Biển bị tán cây che khuất trên 10% diện tích, vẽ theo phần nhìn thấy và bật occluded=true.
Common mistake: Tự suy đoán vẽ vòng tròn amodal đè lên lá cây hoặc quên bật occluded=true.
Diversity: occlusion

---

CASE ID: EC05
Sample: GTS08
Scene: Cột biển báo bên đường
Observation: Trên cùng 1 cột thép có 1 biển cấm 50km/h phía trên và 1 biển hình chữ nhật nhỏ phía dưới
Decision: LABEL
Expected: 2 instances: instance 1 class prohibitory, instance 2 class supplementary
Rationale: Mỗi mặt biển báo là 1 instance độc lập, bắt buộc tách thành 2 nhãn riêng.
Common mistake: Khoanh gộp cả 2 biển thành 1 nhãn chung.
Diversity: conflict

---

CASE ID: EC06
Sample: GTS12
Scene: Khoảng cách xa trên đường
Observation: Biển báo ở xa trên 60m vỡ hạt pixel không thể đọc chữ hay hình biểu tượng bên trong
Decision: LABEL
Expected: class: other, occluded: false, truncated: false, blurred: true
Rationale: Biển quá mờ xa không thể xác định 4 class chính -> Gán class other + bật blurred=true.
Common mistake: Đoán mò class cấm hay chỉ dẫn khi không đủ bằng chứng hình ảnh.
Diversity: small_far

---

CASE ID: EC07
Sample: GTS18
Scene: Mặt sau biển báo
Observation: Biển báo quay mặt lưng màu xám về phía camera
Decision: LABEL
Expected: class: other, occluded: true, truncated: false, blurred: false
Rationale: Biển quay mặt lưng không nhìn thấy nội dung -> Gán class other + bật occluded=true.
Common mistake: Không gán nhãn hoặc khoanh gộp cả cột đỡ.
Diversity: ambiguity

---

CASE ID: EC08
Sample: GTS17
Scene: Vùng ảnh lóa sáng ánh nắng mặt trời
Observation: Ảnh bị lóa sáng mạnh ở góc trên không chắc chắn có biển báo hay không
Decision: ESCALATE
Expected: Tag image_escalate cấp ảnh
Rationale: Ảnh nghi vấn không đủ dữ kiện, gắn cờ để QA Lead kiểm tra và chốt Gold Decision.
Common mistake: Annotator tự đoán mò gán nhãn không có căn cứ.
Diversity: escalation
