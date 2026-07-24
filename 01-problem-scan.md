# 01 — Problem Scan & Quick Assessment

> **Bối cảnh:** Nhóm đóng vai trò AI Product Engineer tại Vin Smart Future, rà soát các điểm nghẽn vận hành có thể cải thiện bằng AI trong hệ sinh thái Vingroup.
>
> **Nguyên tắc sử dụng số liệu:** Các con số trong tài liệu này là **giả định làm việc phục vụ bài lab**, được dùng để xây baseline và thiết kế thử nghiệm. Chúng không phải số liệu vận hành chính thức và phải được xác minh bằng log thực tế, phỏng vấn người vận hành và pilot trước khi ra quyết định sản xuất.

---

## Phase 1 — SCAN: Quét cơ hội

### 1.1. Bảng quét vấn đề theo 4 lenses

| # | Công ty | Lens chính | Vấn đề vận hành quan sát/giả định | Tín hiệu giá trị | Rủi ro nếu làm sai |
|---:|---|---|---|---|---|
| 1 | **Xanh SM** | Stakeholder Pain; Tốn thời gian | Khi tài xế báo pin yếu giữa chuyến, điều phối viên phải chuyển qua nhiều màn hình để xác minh vị trí, mức pin, trạm sạc và tự soạn hướng dẫn. | Thời gian xử lý giả định 15–20 phút/lượt; ảnh hưởng trực tiếp đến tài xế và hành khách. | Chỉ sai trạm có thể khiến xe cạn pin giữa đường; tuyệt đối không được tự gửi lệnh chưa duyệt. |
| 2 | **Vinhomes** | Lặp lại; AI-upgrade | Phản ánh cư dân được viết bằng ngôn ngữ tự do, nhân viên phải đọc, phân loại, đặt mức ưu tiên và chuyển đúng bộ phận. | Có thể rút ngắn thời gian triage và giảm chuyển sai phòng ban. | Phân loại sai sự cố an ninh, cháy nổ hoặc tranh chấp phí có thể làm trễ SLA nghiêm trọng. |
| 3 | **VinFast** | Tốn thời gian; Lặp lại | Cố vấn dịch vụ đọc mô tả lỗi của khách hàng và lịch sử sửa chữa để chuẩn bị phiếu tiếp nhận, trong khi thuật ngữ khách dùng thường không nhất quán. | Giảm thời gian nhập liệu và chuẩn hóa mô tả triệu chứng trước khi kỹ thuật viên kiểm tra. | AI không được tự chẩn đoán an toàn xe hoặc phê duyệt thay linh kiện. |
| 4 | **Vinpearl** | AI-upgrade | Nhân viên chăm sóc khách phải trả lời lặp lại các yêu cầu về giờ check-in, tiện ích, đổi lịch và chính sách theo từng cơ sở. | Tăng tốc soạn phản hồi đa ngôn ngữ, nhất quán theo nguồn chính sách đã duyệt. | Chính sách giá và hoàn/hủy thay đổi; trả lời sai có thể tạo cam kết thương mại ngoài thẩm quyền. |
| 5 | **Vinmec** | Stakeholder Pain; Tốn thời gian | Nhân viên y tế mất thời gian tổng hợp ghi chú rời rạc thành bản nháp hướng dẫn sau khám/xuất viện. | Giảm thao tác hành chính để nhân viên dành thời gian cho người bệnh. | Dữ liệu nhạy cảm và rủi ro y khoa rất cao; bác sĩ phải duyệt, AI không được kê đơn hay thay đổi chỉ định. |
| 6 | **Xanh SM** | Lặp lại | Ghi chú hủy chuyến từ tổng đài, tài xế và ứng dụng không đồng nhất, gây khó cho việc tìm nguyên nhân gốc theo tuần. | Tự động gom nhóm chủ đề giúp Product/Ops phát hiện pattern sớm. | Có thể gán sai nguyên nhân hoặc suy diễn ý định của khách hàng/tài xế. |
| 7 | **VinFast** | AI-upgrade; Stakeholder Pain | Phản hồi bảo hành từ nhiều kênh cần được tóm tắt và gắn với điều khoản áp dụng trước khi chuyên viên ra quyết định. | Giảm thời gian đọc hồ sơ dài và tăng khả năng truy vết bằng trích dẫn nguồn. | AI không được tự phê duyệt/từ chối bảo hành hoặc bịa điều khoản. |

### 1.2. Sàng lọc nhanh

Thang điểm: **1 = thấp/khó**, **5 = cao/dễ**. “Khả năng kiểm soát rủi ro” càng cao nghĩa là càng dễ giới hạn bằng Human-in-the-loop, rule và fallback.

| Ứng viên | Tần suất/độ lặp | Giá trị thời gian | Dữ liệu có thể tiếp cận | Khả năng kiểm soát rủi ro | Tổng /20 |
|---|---:|---:|---:|---:|---:|
| Xanh SM — hỗ trợ xe pin yếu | 4 | 5 | 4 | 4 | **17** |
| Vinhomes — triage phản ánh cư dân | 5 | 4 | 4 | 3 | **16** |
| Xanh SM — phân tích lý do hủy chuyến | 4 | 4 | 4 | 5 | **17** |
| VinFast — chuẩn bị phiếu tiếp nhận | 4 | 4 | 3 | 3 | **14** |
| Vinpearl — soạn phản hồi chính sách | 5 | 3 | 4 | 3 | **15** |
| Vinmec — bản nháp hướng dẫn sau khám | 4 | 5 | 2 | 1 | **12** |

**Top 3 đưa vào Quick Assessment:**

1. Xanh SM — hỗ trợ điều phối xe pin yếu.
2. Vinhomes — triage phản ánh cư dân.
3. Xanh SM — phân tích lý do hủy chuyến.

---

## Phase 2 — QUICK-ASSESS

## Quick Problem Card #1 — Xanh SM: Điều phối xe pin yếu

| Thuộc tính | Nội dung |
|---|---|
| **Bài toán một câu** | Khi tài xế Xanh SM báo pin yếu ngoài hiện trường, điều phối viên cần nhanh chóng tổng hợp vị trí, mức pin và trạm sạc để đưa ra một phương án an toàn, nhưng hiện phải tra cứu và soạn hướng dẫn thủ công trên nhiều hệ thống. |
| **Công ty thành viên** | Xanh SM (GSM) |
| **Actor đang đau** | Điều phối viên chịu áp lực SLA; tài xế chờ chỉ dẫn trong tình huống gấp; hành khách có thể bị gián đoạn chuyến đi. |
| **Workflow hiện tại** | (1) Nhận yêu cầu → (2) Xác minh biển số, pin và vị trí → (3) Tra cứu GPS → (4) Tra cứu trạm sạc → (5) Đánh giá an toàn và soạn hướng dẫn → (6) Duyệt/gửi hoặc gọi hỗ trợ. |
| **Bottleneck** | Bước 4–5: chuyển đổi giữa dashboard, so sánh khoảng cách/khả dụng và viết lại thông tin; giả định **9 phút**, chiếm hơn một nửa tổng thời gian khoảng **17 phút/lượt**. |
| **AI có thể hỗ trợ** | Tổng hợp dữ liệu đã truy xuất, nhận diện ý định, soạn bản nháp dễ hiểu và giải thích phương án. Rule engine vẫn quyết định ngưỡng pin/khoảng cách; điều phối viên duyệt hành động. |
| **Success metrics** | P90 thời gian từ khi nhận đủ dữ liệu đến khi có phương án được duyệt **< 3 phút**; ≥90% draft được duyệt sau không quá một lần sửa; 100% test pin <5% và trạm >5 km kích hoạt phương án sạc lưu động; 0 tin nhắn tự gửi. |
| **Quick Architecture** | **Hybrid Rule + LLM Feature + Human-in-the-loop**, không dùng agent tự hành. |

### Phản biện nhanh

- **Vì sao không chỉ dùng rule?** Rule phù hợp để quyết định ngưỡng an toàn nhưng không xử lý tốt mô tả tự do, thiếu dữ kiện và việc soạn giải thích phù hợp ngữ cảnh.
- **Vì sao không dùng agent?** Hành động điều phối có ảnh hưởng an toàn và chi phí; cho phép agent tự gọi cứu hộ/tự gửi tin là vượt quá mức rủi ro chấp nhận được ở giai đoạn thử nghiệm.
- **Điều kiện để làm:** Có API/dữ liệu sandbox cho GPS, pin, danh sách trạm; có playbook cứu hộ; có người vận hành tham gia thiết kế và đánh giá.

---

## Quick Problem Card #2 — Vinhomes: Triage phản ánh cư dân

| Thuộc tính | Nội dung |
|---|---|
| **Bài toán một câu** | Phản ánh cư dân viết bằng ngôn ngữ tự do cần được phân loại, phát hiện mức khẩn cấp và chuyển đến đúng bộ phận, nhưng nhân viên đang đọc và route thủ công. |
| **Công ty thành viên** | Vinhomes |
| **Actor đang đau** | Nhân viên CSKH, ban quản lý tòa nhà, đội kỹ thuật/an ninh và cư dân đang chờ phản hồi. |
| **Workflow hiện tại** | (1) Tiếp nhận ticket → (2) Đọc nội dung/ảnh → (3) Phân loại chủ đề và mức ưu tiên → (4) Chọn đơn vị xử lý → (5) Soạn xác nhận cho cư dân. |
| **Bottleneck** | Bước 2–4; giả định **6–10 phút/ticket**, dễ chuyển nhầm khi cư dân mô tả nhiều vấn đề trong một yêu cầu. |
| **AI có thể hỗ trợ** | Gợi ý category, urgency, tóm tắt và bộ phận nhận; rule ưu tiên tuyệt đối từ khóa/chỉ báo cháy, an ninh, thang máy mắc kẹt. |
| **Success metrics** | ≥90% category khớp nhãn chuyên viên; ≥99% recall trên nhóm sự cố khẩn cấp trong bộ test; giảm median triage xuống **< 90 giây**; 100% ticket khẩn cấp được người trực xác nhận. |
| **Quick Architecture** | **Classifier/LLM Feature + Rule escalation + Human review**. |

### Phản biện nhanh

- Cần taxonomy chuẩn và dữ liệu ticket đã ẩn danh; nếu nhãn lịch sử không nhất quán thì AI chỉ học lại sự thiếu nhất quán.
- Các sự cố khẩn cấp phải có rule độc lập, không phụ thuộc hoàn toàn vào xác suất của LLM.
- Chưa chọn làm deep-dive vì phạm vi category và quy trình giữa các khu đô thị có thể khác nhau, cần chuẩn hóa trước.

---

## Quick Problem Card #3 — Xanh SM: Phân tích lý do hủy chuyến

| Thuộc tính | Nội dung |
|---|---|
| **Bài toán một câu** | Lý do hủy chuyến nằm trong ghi chú tự do của nhiều bên khiến đội vận hành khó nhận diện pattern lỗi theo khu vực, khung giờ và phiên bản ứng dụng. |
| **Công ty thành viên** | Xanh SM (GSM) |
| **Actor đang đau** | Ops Analyst, Product Manager và quản lý đội xe. |
| **Workflow hiện tại** | (1) Xuất dữ liệu → (2) Làm sạch ghi chú → (3) Đọc mẫu → (4) Gắn nhãn thủ công → (5) Tổng hợp dashboard và báo cáo tuần. |
| **Bottleneck** | Bước 2–4; giả định **6–8 giờ/tuần** cho một analyst, kết quả khó tái lập giữa những người gắn nhãn. |
| **AI có thể hỗ trợ** | Tóm tắt và gắn đa nhãn có dẫn chứng từ ghi chú gốc; nhóm các trường hợp “khác” để đề xuất taxonomy mới. |
| **Success metrics** | Macro-F1 ≥0,85 trên tập kiểm định; giảm thời gian chuẩn bị báo cáo xuống **< 2 giờ/tuần**; 100% kết luận có liên kết về dữ liệu nguồn; không dùng kết quả để kỷ luật cá nhân tự động. |
| **Quick Architecture** | **Batch LLM classification + Analytics**, không cần agent. |

### Phản biện nhanh

- Đây là use case rủi ro thấp và phù hợp pilot, nhưng tác động không tức thời bằng sự cố pin yếu.
- Cần cơ chế chống suy diễn: mô hình chỉ gắn nhãn từ taxonomy và trả về “không đủ thông tin” khi bằng chứng yếu.
- Không được dùng nhãn AI làm căn cứ duy nhất để đánh giá tài xế.

---

## 2.4. Quyết định chọn bài toán

Nhóm chọn **Quick Problem Card #1 — Hỗ trợ điều phối xe Xanh SM pin yếu** để deep-dive.

### Lý do chọn

1. **Giá trị vận hành trực tiếp:** mỗi phút chậm trễ làm tăng rủi ro xe cạn pin và gián đoạn chuyến.
2. **Phân vai công nghệ rõ:** rule xử lý điều kiện an toàn; LLM xử lý ngôn ngữ và soạn thảo; con người phê duyệt.
3. **Có thể prototype hẹp:** thử bằng dữ liệu giả lập, không cần kết nối hệ thống sản xuất hoặc tự động gửi hành động.
4. **Đo lường được:** thời gian xử lý, tỷ lệ chấp nhận draft, tỷ lệ vi phạm boundary và số lần fallback đều có thể ghi log.

### Vì sao chưa chọn hai bài còn lại?

- **Vinhomes triage:** cần chuẩn hóa taxonomy và escalation matrix giữa các địa điểm trước.
- **Phân tích hủy chuyến:** an toàn và dễ pilot hơn, nhưng tác động là back-office theo tuần, không trực tiếp giải quyết sự cố thời gian thực.

### Giả thuyết sản phẩm cần kiểm chứng

> Nếu điều phối viên được cung cấp một bản nháp phương án đã tổng hợp dữ liệu, được khóa bằng rule an toàn và luôn yêu cầu phê duyệt, thì P90 thời gian xử lý sự cố pin yếu có thể giảm từ baseline giả định 17 phút xuống dưới 3 phút mà không phát sinh hành động tự động ngoài thẩm quyền.

