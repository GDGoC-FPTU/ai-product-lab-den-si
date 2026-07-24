# 01 — Problem Scan & Quick Problem Cards (Cá nhân)

---

## 🔍 Phase 1 — SCAN: Danh sách 5 Bài toán Vận hành tại Vingroup

| # | Công ty thành viên | Lens | Mô tả ngắn bài toán & Bottleneck |
|---|--------------------|------|-----------------------------------|
| 1 | **Xanh SM** | **Pain từ người khác / Lặp lại** | Điều vận xe điện (Smart Dispatching) và xử lý sự cố hết pin khi đang chạy: Điều vận viên tốn nhiều thời gian kiểm tra khoảng cách trạm sạc khi xe báo pin yếu (< 5%), dễ gây rủi ro chết máy giữa đường. |
| 2 | **Vinhomes** | **Tốn thời gian** | Xử lý & phân loại phản ánh dịch vụ cư dân: Nhân viên CSKH phải đọc thủ công hàng trăm yêu cầu sửa chữa/khiếu nại mỗi ngày và phân phân loại thủ công tới các Ban quản trị tòa nhà. |
| 3 | **VinFast** | **AI-upgrade** | Trợ lý ảo chẩn đoán lỗi pin và đề xuất lịch bảo dưỡng định kỳ cho chủ xe EV dựa trên dữ liệu telemetry xe điện real-time. |
| 4 | **Vinmec** | **Tốn thời gian** | Phân loại & sắp xếp thứ tự ưu tiên hồ sơ khám bệnh (Medical Triage) tại khoa Cấp cứu / Khám bệnh tổng quát trước khi chuyển bác sĩ chuyên khoa. |
| 5 | **Vinpearl** | **Lặp lại** | Tự động hóa giải đáp thắc mắc đa ngôn ngữ và hỗ trợ đặt lịch dịch vụ vui chơi/nghỉ dưỡng cho khách quốc tế tại VinWonders. |

---

## 🃏 Phase 2 — QUICK-ASSESS: 3 Quick Problem Cards

### ┌─────────────────────────────────────────────────────────────┐
### │ QUICK PROBLEM CARD #1                                       │
### └─────────────────────────────────────────────────────────────┘
* **Bài toán (1 câu):** Tự động hóa trợ lý điều vận Xanh SM để đề xuất cứu hộ sạc pin di động khẩn cấp khi xe điện báo pin yếu (< 5%) hoặc chỉ đường trạm sạc an toàn.
* **Công ty thành viên:** `[X] Xanh SM`  `[ ] VinFast`  `[ ] Vinhomes`  `[ ] Vinmec`  `[ ] Vinpearl`
* **Ai đang đau (Actor)?** Điều vận viên trung tâm (Dispatcher) và Tài xế taxi/xe máy điện Xanh SM.
* **Workflow thủ công hiện tại (3-5 bước):**
  1. Tài xế gọi điện/báo tin nhắn pin yếu khẩn cấp ──> 2. Điều vận viên định vị vị trí xe trên bản đồ ──> 3. Điều vận viên tra cứu các trạm sạc gần nhất thủ công ──> 4. Soạn tin nhắn chỉ đường hoặc gọi đội cứu hộ ──> 5. Gửi thông tin cho tài xế.
* **Bước tốn thời gian/lỗi nhất:** Bước 3 & 4 (Tra cứu khoảng cách và soạn tin nhắn hướng dẫn khẩn cấp, mất ⏱ 8-10 phút/lượt).
* **AI có thể nhảy vào hỗ trợ ở bước nào?** Bước 3 & 4 (AI tự động tính toán dung lượng pin, vị trí GPS, đề xuất lệnh điều xe cứu hộ sạc pin di động hoặc tạo draft tin nhắn hướng dẫn).
* **Đo thành công bằng gì (Metric có số)?** Giảm thời gian xử lý yêu cầu cứu hộ/sạc pin từ **10 phút ──> dưới 1 phút (SLA < 60s)**.
* **Quick Architecture:** `[ ] No AI`  `[ ] Rule`  `[ ] LLM`  `[X] Agent` (Agentic Loop với HITL Human-in-the-loop).

---

### ┌─────────────────────────────────────────────────────────────┐
### │ QUICK PROBLEM CARD #2                                       │
### └─────────────────────────────────────────────────────────────┘
* **Bài toán (1 câu):** Tự động phân loại, trích xuất thông tin và chuyển giao yêu cầu sửa chữa/khiếu nại của cư dân Vinhomes tới đúng bộ phận kỹ thuật tòa nhà.
* **Công ty thành viên:** `[ ] Xanh SM`  `[ ] VinFast`  `[X] Vinhomes`  `[ ] Vinmec`  `[ ] Vinpearl`
* **Ai đang đau (Actor)?** Nhân viên Ban Quản lý (BQL) tòa nhà Vinhomes và Cư dân đô thị.
* **Workflow thủ công hiện tại (3-5 bước):**
  1. Cư dân gửi phản ánh qua ứng dụng/hotline ──> 2. Lễ tân đọc nội dung và ghi chép vào sổ/Excel ──> 3. Xác định loại sự cố (Điện/Nước/Hạ tầng) ──> 4. Gọi điện chuyển ticket cho đội kỹ thuật ──> 5. Phản hồi lại cư dân.
* **Bước tốn thời gian/lỗi nhất:** Bước 2 & 3 (Đọc phản ánh và phân loại thủ công, mất ⏱ 15 phút/ticket).
* **AI có thể nhảy vào hỗ trợ ở bước nào?** Bước 2 & 3 (LLM đọc nội dung phản ánh, phân loại mức độ khẩn cấp, trích xuất vị trí căn hộ và tự động tạo ticket).
* **Đo thành công bằng gì (Metric có số)?** Tăng tỷ lệ phân loại tự động đúng từ **60% ──> trên 95%**, giảm thời gian tạo ticket từ **15 phút ──> dưới 30 giây**.
* **Quick Architecture:** `[ ] No AI`  `[ ] Rule`  `[X] LLM`  `[ ] Agent`.

---

### ┌─────────────────────────────────────────────────────────────┐
### │ QUICK PROBLEM CARD #3                                       │
### └─────────────────────────────────────────────────────────────┘
* **Bài toán (1 câu):** Phân tích dữ liệu telemetry pin xe điện VinFast để dự báo cảnh báo bảo trì pin sớm và đề xuất đặt lịch hẹn tại VinFast Service Workshop.
* **Công ty thành viên:** `[ ] Xanh SM`  `[X] VinFast`  `[ ] Vinhomes`  `[ ] Vinmec`  `[ ] Vinpearl`
* **Ai đang đau (Actor)?** Kỹ sư dịch vụ VinFast và Khách hàng sở hữu xe điện VinFast (VF5, VF8, VF9...).
* **Workflow thủ công hiện tại (3-5 bước):**
  1. Hệ thống xe báo lỗi pin trên màn hình màn điều khiển ──> 2. Khách hàng gọi tới tổng đài VinFast ──> 3. Nhân viên kỹ thuật yêu cầu mang xe tới xưởng kiểm tra ──> 4. Kỹ sư cắm máy đọc lỗi CAN bus ──> 5. Lập báo cáo bảo hành/thay thế.
* **Bước tốn thời gian/lỗi nhất:** Bước 3 & 4 (Kiểm tra và chẩn đoán thủ công tại xưởng khi sự cố đã xảy ra, mất ⏱ 120 phút/xe).
* **AI có thể nhảy vào hỗ trợ ở bước nào?** Bước 1 & 2 (AI phân tích dữ liệu dòng điện/nhiệt độ cell pin thời gian thực, phát hiện bất thường trước khi hỏng hóc và soạn thông báo nhắc bảo dưỡng kèm lịch hẹn).
* **Đo thành công bằng gì (Metric có số)?** Giảm tỷ lệ xe hỏng pin đột ngột giữa đường xuống **dưới 2%**, thời gian đưa ra cảnh báo sớm trước **48 giờ**.
* **Quick Architecture:** `[ ] No AI`  `[X] Rule + LLM`  `[ ] Agent`.
