# 02 - Deep-Dive Report: Vin Smart Future (Xanh SM — Phân luồng hỗ trợ pin)

**Tên nhóm:** [ĐIỀN TÊN NHÓM]
**Thành viên:**
| Họ và tên | MSSV |
|---|---|
| [ĐIỀN] | [ĐIỀN] |
| [ĐIỀN] | [ĐIỀN] |
| [ĐIỀN] | [ĐIỀN] |

**Bài toán đã chọn:** Quick Problem Card 1 — Phân luồng hỗ trợ pin Xanh SM (xem chi tiết lựa chọn tại `01-problem-scan.md`).

---

# 🏗️ Phase 3 — DEEP-DIVE

## 3.1. Current-State Workflow Mapping

> Bản nháp text-diagram dưới đây dùng để nhóm đối chiếu trước khi vẽ tay lên giấy A3 và chụp ảnh lưu thành `04-workflow-diagram.png`.

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Tài xế báo   │     │ Kiểm tra pin │     │ Kiểm tra     │     │ Chọn phương  │
│ sự cố pin    │ ──→ │ và GPS xe    │ ──→ │ trạm sạc còn │ ──→ │ án an toàn   │
│              │     │              │     │ chỗ trống    │     │ (sạc/cứu hộ) │
│ Ai: Tài xế   │     │ Ai: Dispatch │     │ Ai: Dispatch │     │ Ai: Dispatch │
│ ⏱ 1 phút     │     │ ⏱ 1 phút     │     │ ⏱ 3 phút 🔴  │     │ ⏱ 2 phút     │
│ In: Cuộc gọi │     │ In: Biển số  │     │ In: Toạ độ   │     │ In: DS trạm  │
│ Out: Log SC  │     │ Out: Toạ độ  │     │ Out: DS trạm │     │ Out: Phương án│
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ┌──────────────┐
                                                               │ Bước 5       │
                                                               │ Soạn & gửi   │
                                                               │ phản hồi cho │
                                                               │ tài xế       │
                                                               │ Ai: Dispatch │
                                                               │ ⏱ 3 phút 🔴  │
                                                               └──────────────┘
🔴 = Bottleneck (Bước 3 & 5)
🔄 = Handoff: Tài xế → Điều phối viên (Bước 1); Điều phối viên → Tài xế (Bước 5)
⏱ Tổng thời gian xử lý thủ công: ~10 phút/lượt (worksheet gốc ghi nhận median 8 phút,
   số liệu là ước tính giả định, cần kiểm chứng bằng log sự cố thực tế trước khi triển khai).
```

## 3.2. Problem Statement (6-field)

| Field | Nội dung |
|---|---|
| **1. Actor / Operator** | Điều phối viên (Dispatcher) thuộc Trung tâm Điều vận Xanh SM. |
| **2. Current Workflow** | Khi tài xế báo pin yếu/hết pin, điều phối viên tự tra cứu vị trí GPS xe, kiểm tra trạm sạc VinFast còn trụ trống gần nhất, cân nhắc phương án an toàn (đến trạm sạc hay điều xe cứu hộ), rồi soạn và gửi phản hồi hướng dẫn cho tài xế. 5 bước, hoàn toàn thủ công. |
| **3. Bottleneck** | Bước 3 (tra cứu trạm sạc còn chỗ) và Bước 5 (soạn phản hồi) — tổng ~6 phút/lượt, chiếm phần lớn thời gian xử lý và dễ sai sót khi điều phối viên xử lý nhiều sự cố cùng lúc. |
| **4. Business Impact** | Mỗi sự cố mất trung bình 8-10 phút xử lý thủ công; khi có nhiều sự cố đồng thời (giờ cao điểm), thời gian chờ của tài xế kéo dài, tăng rủi ro tài xế cạn pin giữa đường và gián đoạn cuốc xe, ảnh hưởng trải nghiệm khách hàng và doanh thu đội xe. |
| **5. Success Metric** | 1. Giảm median triage time từ 8 phút xuống dưới 2 phút (Efficiency).<br>2. Giữ số vi phạm routing khi pin ở mức nguy cấp (dưới ngưỡng an toàn) bằng 0 (Safety/Quality — không thỏa hiệp). |
| **6. Operational Boundary** | AI được phép: đọc dữ liệu pin/GPS, tra cứu trạm sạc trống, tóm tắt sự cố, phân loại mức khẩn, soạn bản nháp phản hồi (`[DRAFT_ONLY]`). **TUYỆT ĐỐI KHÔNG** được: tự động gửi phản hồi khi chưa có điều phối viên duyệt; tự quyết định hành động khi pin dưới ngưỡng nguy cấp (quyết định này bắt buộc là rule cứng bằng code, không giao cho LLM); bịa dữ liệu GPS/trạm sạc không có thật. **Cần duyệt:** mọi phản hồi trước khi gửi tài xế; mọi trường hợp pin dưới ngưỡng nguy cấp bắt buộc escalate cho điều phối viên xử lý trực tiếp. |

## 3.3. Future-State Flow & AI Fit

**AI Fit:** Hybrid — **Rule/State-Machine** cho quyết định an toàn (ngưỡng pin, chọn phương án cứu hộ) kết hợp **LLM Feature** để tóm tắt sự cố và soạn bản nháp phản hồi. Không chọn Agentic Loop vì quy trình có cấu trúc cố định, rủi ro an toàn cao nếu để AI tự trị hành động mà không qua rule cứng + duyệt của con người.

```text
┌──────────────┐     ┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│ Bước 1       │     │ Bước 2       │     │ Bước 3       │     │ Bước 4       │
│ Tài xế báo   │     │ 🔵 Rule check│     │ 🔵 AI tóm    │     │ 🟢 Dispatch  │
│ sự cố pin    │ ──→ │ ngưỡng pin + │ ──→ │ tắt & draft  │ ──→ │ duyệt & gửi  │
│              │     │ auto-pull GPS│     │ phản hồi     │     │ phản hồi     │
└──────────────┘     └──────────────┘     └──────────────┘     └──────────────┘
                                                                      │
                                                                      ▼
                                                               ↩️ Fallback:
                                                               Nếu pin dưới ngưỡng
                                                               nguy cấp hoặc AI draft
                                                               lỗi/không rõ ràng →
                                                               escalate ngay cho
                                                               điều phối viên xử lý
                                                               thủ công như cũ.
```

---

# 🏁 Phase 5 — EVALUATE

### AI Readiness Checklist
1. [x] Chúng tôi có sẵn dữ liệu mẫu/logs sạch để test? → **Có một phần** — cần thu thập thêm log sự cố pin thực tế để kiểm chứng số liệu giả định (8-10 phút/lượt) trước khi triển khai chính thức.
2. [x] Rủi ro khi AI sai có nằm trong tầm kiểm soát (qua HITL hoặc Fallback)? → **Có** — quyết định an toàn (ngưỡng pin) nằm ở rule cứng bằng code, không giao AI; mọi phản hồi đều qua điều phối viên duyệt trước khi gửi (`[DRAFT_ONLY]`); có fallback rõ ràng khi AI lỗi.
3. [x] Stakeholders sẵn sàng thay đổi quy trình làm việc cũ? → **Có** — thay đổi chỉ ở việc điều phối viên duyệt bản nháp thay vì tự soạn từ đầu, không đảo lộn quy trình vận hành hiện tại.

### Quyết định cuối cùng của Ban Giám Đốc Vin Smart Future
- [x] **GO (Bắt đầu xây dựng Prototype):** Bắt đầu phát triển với scope hẹp.
- [ ] NOT YET
- [ ] NO-GO

**Justification:**
> Bài toán có workflow hẹp, quan sát được rõ ràng, và ranh giới an toàn được thiết kế chặt chẽ: quyết định mang tính an toàn (ngưỡng pin, phương án cứu hộ) hoàn toàn nằm ở logic rule cứng bằng code chứ không giao cho LLM, loại bỏ rủi ro AI "quyết định sai" ở phần quan trọng nhất. AI chỉ đảm nhận phần tóm tắt/soạn thảo — nơi LLM có lợi thế rõ rệt và rủi ro thấp vì luôn có điều phối viên duyệt trước khi gửi. Kết quả test 3 adversarial case (prompt injection, ép bỏ qua bước duyệt, ép đề xuất trạm sạc xa khi pin nguy cấp) đều PASS ở lớp guardrail bằng code, không phụ thuộc hoàn toàn vào việc model có "nghe lời" system prompt hay không — đây là yếu tố kỹ thuật quan trọng nhất để tin tưởng triển khai scope hẹp. Chi phí triển khai thấp (LLM feature đơn giản, không cần Agent phức tạp), phù hợp bắt đầu pilot ở quy mô nhỏ (một khu vực/trung tâm điều vận) trước khi nhân rộng.