# 02 — Deep-Dive Report

## Thông tin nhóm — BẮT BUỘC HOÀN THIỆN TRƯỚC KHI NỘP

| Trường | Thông tin |
|---|---|
| **Tên nhóm** | `[ĐIỀN TÊN NHÓM]` |
| **Lớp / Mã lớp** | `[ĐIỀN LỚP]` |
| **Thành viên 1** | `[HỌ VÀ TÊN] — [MSSV]` |
| **Thành viên 2** | `[HỌ VÀ TÊN] — [MSSV]` |
| **Thành viên 3** | `[HỌ VÀ TÊN] — [MSSV]` |
| **Thành viên 4** | `[HỌ VÀ TÊN] — [MSSV]` |

> Xóa các dòng thành viên không sử dụng và thay toàn bộ placeholder trước khi nộp.

---

# Executive Summary

Nhóm đề xuất một **Dispatcher Co-pilot** hỗ trợ điều phối viên Xanh SM xử lý tình huống xe điện pin yếu ngoài hiện trường. Hệ thống không thay điều phối viên ra quyết định và không trực tiếp điều khiển xe. Nó tập hợp dữ liệu đã được cấp quyền, áp dụng rule an toàn, tạo bản nháp phương án và chờ con người phê duyệt.

**Vấn đề:** Quy trình giả định hiện tại mất khoảng **17 phút/lượt**, trong đó 9 phút tập trung ở khâu tra cứu trạm và soạn hướng dẫn. Việc chuyển qua nhiều công cụ vừa chậm vừa dễ bỏ sót điều kiện pin/khoảng cách.

**Mục tiêu pilot:** Đưa P90 thời gian từ lúc có đủ dữ liệu đến lúc điều phối viên duyệt phương án xuống **dưới 3 phút**, trong khi duy trì:

- **0** tin nhắn hoặc lệnh điều phối được tự động gửi;
- **100%** tình huống pin dưới 5% và trạm xa hơn 5 km bị chặn chỉ đường và chuyển sang đề xuất sạc lưu động;
- **100%** quyết định có audit log;
- ít nhất **90%** bản nháp được chấp nhận sau không quá một lần chỉnh sửa.

**Quyết định:** **GO cho prototype/pilot có kiểm soát**, chưa GO cho production hoặc agent tự hành.

---

# Phase 3 — DEEP-DIVE

## 3.1. Quyết định lựa chọn

**Bài toán được chọn:** Hỗ trợ điều phối viên Xanh SM xử lý xe pin yếu ngoài hiện trường.

Nhóm ưu tiên bài toán này vì có tác động trực tiếp đến an toàn vận hành, thời gian phản hồi và trải nghiệm tài xế; đồng thời có thể chia trách nhiệm rõ ràng giữa rule, LLM và con người.

### Scope của prototype

**Trong scope:**

- Nhận input giả lập gồm biển số/ID xe, mức pin, vị trí, danh sách trạm và khoảng cách.
- Kiểm tra đủ/thiếu dữ liệu.
- Áp dụng boundary pin–khoảng cách.
- Soạn bản nháp hướng dẫn hoặc đề xuất điều xe sạc lưu động.
- Gắn `[DRAFT_ONLY]`, hiển thị lý do và yêu cầu điều phối viên duyệt.
- Lưu input, rule result, output, quyết định của người duyệt và timestamp.

**Ngoài scope:**

- Tự gửi tin nhắn cho tài xế.
- Tự điều xe sạc lưu động/cứu hộ.
- Điều khiển phương tiện hoặc thay đổi hành trình trên xe.
- Tự kết luận trạm còn chỗ nếu không có dữ liệu thời gian thực.
- Xử lý tai nạn, cháy nổ, cấp cứu hoặc tình huống an ninh.
- Dùng dữ liệu định vị ngoài mục đích xử lý sự cố đã khai báo.

---

## 3.2. Current-State Workflow

Sơ đồ trực quan nằm tại `04-workflow-diagram.png`.

| Bước | Actor / Công cụ | Input | Hoạt động | Output | Thời gian giả định | Điểm kiểm soát |
|---:|---|---|---|---|---:|---|
| 1 | Tài xế → Tổng đài/điều phối | Cuộc gọi/tin nhắn báo pin yếu | Tiếp nhận yêu cầu và mở ticket | Ticket sự cố | 2 phút | 🔄 Handoff tài xế → điều phối |
| 2 | Điều phối viên | Biển số, mô tả tự do | Hỏi lại mức pin, vị trí, trạng thái hành khách và khả năng di chuyển | Bộ thông tin tối thiểu | 2 phút | Dễ thiếu dữ kiện khi tình huống gấp |
| 3 | Điều phối viên / Dashboard đội xe | ID/biển số | Tra cứu GPS và mức pin hiển thị trên hệ thống | Vị trí + pin xác minh | 2 phút | 🔄 Handoff người → hệ thống |
| 4 | Điều phối viên / Dashboard trạm | Tọa độ xe | Tìm trạm, kiểm tra khoảng cách và trạng thái khả dụng | Danh sách ứng viên | **5 phút** | 🔴 Bottleneck: nhiều màn hình, dữ liệu có thể trễ |
| 5 | Điều phối viên / bản đồ/playbook | Pin + trạm ứng viên | So sánh độ an toàn, chọn phương án và soạn hướng dẫn | Bản nháp phương án | **4 phút** | 🔴 Bottleneck: đánh giá + viết dưới áp lực |
| 6 | Điều phối viên / app tài xế/điện thoại | Bản nháp | Rà soát, gửi hướng dẫn hoặc gọi đội hỗ trợ | Hành động được phê duyệt | 2 phút | 🔄 Handoff điều phối → tài xế/đội hỗ trợ |

**Tổng baseline giả định:** 17 phút/lượt, dao động 15–20 phút tùy mức độ đầy đủ của thông tin và khả năng truy cập dashboard.

### Root-cause analysis

| Hiện tượng | Nguyên nhân gần | Nguyên nhân gốc cần xác minh |
|---|---|---|
| Xử lý lâu | Chuyển đổi giữa cuộc gọi, bản đồ, dashboard xe và dashboard trạm | Thiếu một màn hình tổng hợp theo tình huống |
| Hỏi lại nhiều lần | Mô tả ban đầu không theo cấu trúc | Chưa có intake form bắt buộc cho sự cố pin |
| Phương án không nhất quán | Điều phối viên dựa vào kinh nghiệm cá nhân | Playbook chưa được mã hóa thành rule có version |
| Rủi ro chỉ sai trạm | Khoảng cách và mức pin được đánh giá thủ công | Thiếu hard guardrail trước khi soạn/gửi |
| Khó audit | Quyết định nằm trong cuộc gọi/tin nhắn rời rạc | Chưa lưu đầy đủ input → rule → draft → approval |

### Handoff quan trọng

1. **Tài xế → Điều phối:** thông tin có thể thiếu hoặc mang tính cảm xúc.
2. **Điều phối → Dashboard xe/trạm:** phải chuyển ngữ cảnh và nhập lại định danh.
3. **Điều phối → Tài xế/đội hỗ trợ:** hành động cuối có ảnh hưởng an toàn và chi phí, cần truy vết người duyệt.

---

## 3.3. Problem Statement — 6 fields

| Field | Nội dung chi tiết |
|---|---|
| **1. Actor / Operator** | Actor chính là **điều phối viên Xanh SM**. Actor liên quan gồm tài xế, hành khách, quản lý ca và đội sạc lưu động/cứu hộ. |
| **2. Current Workflow** | Điều phối viên tiếp nhận mô tả tự do, xác minh pin/vị trí, mở dashboard đội xe và trạm sạc, so sánh khả năng tiếp cận, soạn hướng dẫn rồi gửi hoặc liên hệ hỗ trợ. Baseline giả định khoảng 17 phút/lượt. |
| **3. Bottleneck** | Tra cứu trạm và soạn phương án chiếm khoảng 9 phút. Đây là phần vừa cần tổng hợp dữ liệu có cấu trúc, vừa cần diễn đạt ngôn ngữ tự nhiên, lại diễn ra dưới áp lực thời gian. |
| **4. Business Impact** | Chậm phản hồi làm tăng thời gian xe dừng, nguy cơ cạn pin giữa đường, gián đoạn chuyến và tải công việc điều phối. Ở pilot, impact sẽ đo bằng phút xử lý tiết kiệm, tỷ lệ escalation, số lượt sửa draft và số sự cố vi phạm SLA; chưa quy đổi thành doanh thu nếu chưa có dữ liệu thật. |
| **5. Success Metric** | P90 có phương án được duyệt <3 phút; ≥90% draft được chấp nhận sau ≤1 lần sửa; 100% test critical boundary đạt; 0 auto-send; 100% case có audit log; fallback kỹ thuật <30 giây kể từ khi phát hiện lỗi. |
| **6. Operational Boundary** | AI chỉ được tóm tắt và tạo bản nháp. Mọi draft gửi tài xế phải bắt đầu `[DRAFT_ONLY]`. Nếu pin <5% và trạm >5 km, không được đề xuất đi trạm; phải tạo đề xuất `dispatch_mobile_charger`. Điều phối viên phải phê duyệt mọi hành động. Khi thiếu pin, vị trí, khoảng cách hoặc trạng thái trạm, AI phải nói “không đủ dữ liệu” và chuyển xử lý thủ công. |

### Problem statement cô đọng

> Điều phối viên Xanh SM cần một cách nhanh và an toàn để tổng hợp dữ liệu và chuẩn bị phương án cho xe pin yếu, bởi quy trình đa màn hình hiện tại giả định mất 17 phút/lượt và dễ bỏ sót điều kiện an toàn. Một co-pilot kết hợp rule + LLM + Human-in-the-loop phải giúp P90 xử lý dưới 3 phút, không tự gửi hành động và không bao giờ chỉ xe pin dưới 5% đến trạm xa hơn 5 km.

---

## 3.4. Metrics và kế hoạch đo

### North-star metric

**P90 time-to-approved-plan:** thời gian từ lúc hệ thống nhận đủ các trường bắt buộc đến khi điều phối viên bấm duyệt một phương án hợp lệ.

### Guardrail metrics

| Metric | Công thức / cách đo | Ngưỡng pilot |
|---|---|---:|
| Critical-boundary pass rate | Case pin <5%, trạm >5 km được chặn / tổng case tương ứng | **100%** |
| Unauthorized-send rate | Hành động gửi không có approval / tổng hành động | **0%** |
| Audit completeness | Case có đủ input, rule version, draft, approver, timestamp / tổng case | **100%** |
| Hallucinated fact rate | Draft chứa địa chỉ/khoảng cách/trạng thái không có trong source / tổng draft | **0%** |
| Fallback activation time | Thời gian phát hiện lỗi đến khi hiển thị quy trình thủ công | **<30 giây** |

### Quality và adoption metrics

| Metric | Baseline cần đo | Mục tiêu pilot |
|---|---:|---:|
| P90 time-to-approved-plan | Giả định 17 phút | <3 phút |
| Draft acceptance sau ≤1 lần sửa | Chưa có | ≥90% |
| Tỷ lệ điều phối viên dùng co-pilot khi đủ điều kiện | Chưa có | ≥80% |
| Điểm hữu ích từ điều phối viên (1–5) | Chưa có | ≥4,0 |
| Tỷ lệ case phải fallback thủ công | Chưa có | Theo dõi; điều tra nếu >10% |

### Thiết kế đo

- **Tuần 0:** lấy mẫu tối thiểu 50 case lịch sử đã ẩn danh để xác nhận baseline.
- **Offline test:** tối thiểu 200 case, gồm case bình thường, thiếu dữ liệu, input injection và boundary.
- **Shadow mode:** hệ thống tạo draft nhưng không hiển thị cho tài xế; so sánh với quyết định thực tế.
- **Pilot:** 5–10 điều phối viên, một khu vực/một ca vận hành, tối đa 4 tuần.
- **Review hàng tuần:** phân tích false positive/negative, draft bị sửa và mọi lần fallback.

---

## 3.5. AI Fit Matrix

| Thành phần | Rule / State machine | LLM Feature | Agentic Loop | Lựa chọn |
|---|---:|---:|---:|---|
| Kiểm tra trường bắt buộc | Rất phù hợp | Không cần | Không cần | **Rule** |
| So sánh pin và khoảng cách | Rất phù hợp, xác định được | Không nên giao hoàn toàn | Rủi ro cao | **Rule** |
| Tóm tắt mô tả tự do | Hạn chế | Rất phù hợp | Không cần | **LLM** |
| Soạn giải thích dễ hiểu | Hạn chế/khô cứng | Phù hợp | Không cần | **LLM** |
| Chọn/gửi hành động cuối | Có thể mã hóa nhưng cần thẩm quyền | Không được tự quyết | Chưa chấp nhận | **Human approval** |
| Tự gọi nhiều hệ thống và hành động | Có thể orchestration có kiểm soát | — | Có tiềm năng nhưng rủi ro | **Ngoài scope pilot** |

### Kết luận AI Fit

Giải pháp phù hợp nhất là **hybrid workflow**, không phải “LLM làm tất cả”:

```text
Validated data
    → deterministic safety rules
    → LLM drafts an explanation
    → output validator
    → human approval
    → authorized system sends/dispatches
```

Rule chịu trách nhiệm cho logic an toàn xác định được. LLM chỉ xử lý phần ngôn ngữ và tổng hợp. Con người giữ quyền quyết định và thực thi.

---

## 3.6. Future-State Flow

| Bước | Loại | Xử lý | Output / điều kiện |
|---:|---|---|---|
| 1 | Human/System | Tài xế hoặc điều phối nhập ID xe và mô tả sự cố | Tạo incident ID |
| 2 | Rule | Kiểm tra trường bắt buộc: mức pin, GPS, trạng thái hành khách | Thiếu dữ liệu → yêu cầu bổ sung hoặc fallback |
| 3 | System integration | Truy xuất snapshot pin/GPS và danh sách trạm đã cấp quyền | Mỗi dữ liệu có timestamp/source |
| 4 | Rule | Tính khoảng cách và áp dụng safety matrix | Pin <5% + trạm >5 km → mobile charger path |
| 5 | 🔵 LLM | Tóm tắt tình huống và soạn draft từ dữ liệu/rule result | Không được tự bổ sung fact |
| 6 | Rule/output validator | Kiểm `[DRAFT_ONLY]`, schema, boundary, citation về source | Fail → regenerate một lần; vẫn fail → fallback |
| 7 | 🟢 Human-in-the-loop | Điều phối viên xem dữ liệu, lý do, sửa/duyệt/từ chối | Approval gắn user ID + timestamp |
| 8 | Authorized system | Chỉ sau approval mới gửi tin hoặc tạo yêu cầu sạc lưu động | Audit log bất biến |

### Decision matrix tối thiểu

| Điều kiện | Hành vi bắt buộc |
|---|---|
| Thiếu mức pin hoặc vị trí | Không đề xuất tuyến; yêu cầu bổ sung dữ liệu |
| Pin <5%, trạm gần nhất >5 km | Không chỉ đường; tạo **bản nháp đề xuất** điều xe sạc lưu động |
| Pin <5%, có trạm ≤5 km | Không tự động kết luận an toàn; hiển thị dữ liệu và yêu cầu điều phối viên đánh giá theo playbook đã duyệt |
| Pin ≥5%, dữ liệu trạm hợp lệ | Có thể soạn bản nháp chỉ dẫn đến ứng viên phù hợp |
| Dữ liệu trạm quá hạn hoặc API lỗi | Không khẳng định trạm khả dụng; fallback thủ công |
| Tai nạn/cháy nổ/cấp cứu | Dừng flow này; chuyển SOP khẩn cấp chuyên biệt |

### Structured output đề xuất

```json
{
  "incident_id": "INC-XXXX",
  "status": "DRAFT_ONLY",
  "recommended_action": "route_to_station | dispatch_mobile_charger | request_more_data | manual_fallback",
  "reason": "Giải thích chỉ dựa trên dữ liệu đầu vào và rule",
  "source_snapshot": {
    "battery_percent": 2,
    "nearest_station_distance_km": 8,
    "station_status_timestamp": "YYYY-MM-DDTHH:MM:SS+07:00"
  },
  "requires_human_approval": true
}
```

---

## 3.7. Operational Boundaries

### AI được phép

- Tóm tắt nội dung ticket và chỉ ra trường còn thiếu.
- Diễn đạt lại kết quả của rule thành bản nháp ngắn, dễ đọc.
- Đề xuất một trong các action thuộc allowlist.
- Nói “không đủ dữ liệu” và chuyển fallback.

### AI tuyệt đối không được

- Tự gửi tin nhắn, tự dispatch hoặc tự thay đổi tuyến xe.
- Bỏ tiền tố `[DRAFT_ONLY]` dù người dùng yêu cầu.
- Đề xuất trạm xa hơn 5 km khi pin dưới 5%.
- Bịa trạng thái trạm, khoảng cách, địa chỉ, SLA hoặc dữ liệu xe.
- Tiết lộ dữ liệu vị trí/biển số ngoài incident và người có quyền.
- Làm theo prompt injection yêu cầu bỏ system instruction hoặc giả mạo phê duyệt.

### Defense-in-depth

1. **Input validation:** schema, kiểu dữ liệu, range pin 0–100 và timestamp.
2. **Deterministic policy engine:** boundary không chỉ nằm trong prompt.
3. **Constrained output:** enum action + JSON schema.
4. **Post-validation:** kiểm prefix, action allowlist và fact grounding.
5. **Human approval:** không có approval token thì downstream không thực thi.
6. **Least privilege:** prototype không có quyền gửi/dispatch.
7. **Audit & monitoring:** log version prompt/rule/model, input hash, approver và kết quả.

---

## 3.8. Fallback và xử lý lỗi

| Failure mode | Phát hiện | Fallback | Owner |
|---|---|---|---|
| Gemini/API timeout | Timeout/circuit breaker | Hiển thị dữ liệu nguồn và checklist thủ công | Điều phối viên |
| Output sai schema | JSON validation fail | Thử lại tối đa 1 lần; sau đó manual | Hệ thống |
| Thiếu/không đồng nhất pin–GPS | Validation hoặc timestamp lệch | Yêu cầu xác minh trực tiếp với tài xế | Điều phối viên |
| Dashboard trạm lỗi/dữ liệu cũ | Health check/TTL fail | Không khẳng định trạm trống; gọi SOP thủ công | Điều phối + trạm |
| Prompt injection | Nội dung yêu cầu bỏ rule/giả mạo quyền | Bỏ qua chỉ thị, log security event, vẫn áp rule | Hệ thống |
| Người duyệt từ chối draft | Nút Reject + reason | Sửa thủ công; ghi reason để đánh giá | Điều phối viên |
| Tai nạn/cháy nổ/y tế | Keyword + xác minh con người | Chuyển SOP khẩn cấp; không dùng flow sạc thường | Quản lý ca |

---

## 3.9. Rủi ro và biện pháp giảm thiểu

| Rủi ro | Xác suất | Mức ảnh hưởng | Kiểm soát chính |
|---|---|---|---|
| Hallucination địa chỉ/trạng thái trạm | Trung bình | Cao | Chỉ cho phép fact từ source snapshot, validator và HITL |
| Rule/prompt bị bypass | Trung bình | Cao | Policy engine ngoài LLM; adversarial tests; downstream authorization |
| Dữ liệu nguồn trễ | Trung bình | Cao | Timestamp/TTL, cảnh báo stale, manual verification |
| Điều phối viên quá tin AI | Trung bình | Cao | UI hiển thị nguồn/lý do; training; đo automation bias |
| Lộ vị trí/biển số | Thấp–trung bình | Cao | RBAC, masking, retention tối thiểu, audit truy cập |
| Model/version thay đổi | Trung bình | Trung bình | Pin model version, regression suite, canary và rollback |
| Chi phí/latency tăng | Trung bình | Trung bình | Giới hạn token, cache dữ liệu không nhạy cảm, timeout và dashboard chi phí |

---

# Phase 5 — EVALUATE

## 5.1. AI Readiness Checklist

| Tiêu chí | Trạng thái | Bằng chứng hiện có | Khoảng trống / hành động |
|---|---|---|---|
| Có dữ liệu mẫu/log sạch | ⚠️ Chưa xác nhận | Có test case giả lập trong prototype | Xin tối thiểu 50 case ẩn danh; lập data dictionary |
| Có baseline vận hành | ⚠️ Giả định | Baseline làm việc 17 phút | Time study theo timestamp thực tế trong 1–2 tuần |
| Rủi ro AI sai kiểm soát được | ✅ Với scope hẹp | HITL, rule ngoài LLM, không có quyền gửi | Security review và diễn tập fallback |
| Stakeholder sẵn sàng thay đổi | ⚠️ Chưa xác nhận | Workflow đề xuất giảm thao tác | Phỏng vấn 5–10 điều phối viên và quản lý ca |
| Có owner nghiệp vụ | ⚠️ Cần chỉ định | Đề xuất Ops Lead làm policy owner | Gán RACI trước pilot |
| Có cách đo chất lượng | ✅ | Metrics và test design đã định nghĩa | Cài telemetry/audit schema |
| Có phương án khi model lỗi | ✅ | Manual fallback + circuit breaker | Chạy tabletop exercise |
| Có prototype kỹ thuật | ⚠️ Đang hiệu chỉnh | System prompt, SDK và adversarial tests đã có | Cập nhật model khả dụng; chạy lại regression |

## 5.2. Quyết định

### ✅ GO — Prototype/pilot có kiểm soát

**Không phải GO cho production automation.** Nhóm chỉ đề xuất đi tiếp với prototype offline và shadow mode vì:

1. Use case có giá trị thời gian rõ, nhưng baseline hiện vẫn là giả định cần xác minh.
2. Phần rủi ro cao có thể tách khỏi LLM bằng deterministic rule và human approval.
3. Prototype có thể chạy với dữ liệu giả lập/ẩn danh và không cần quyền gửi hành động.
4. Các metric an toàn có tiêu chí pass/fail rõ ràng.
5. Nếu pilot không đạt boundary 100%, dự án dừng trước khi mở rộng.

### Kill criteria

Dừng hoặc quay lại thiết kế nếu xảy ra một trong các điều kiện:

- Bất kỳ critical-boundary test nào thất bại.
- Có hành động downstream không có approval.
- Hallucinated fact rate >0 trong bộ case an toàn trọng yếu.
- P90 không cải thiện ít nhất 50% sau hai vòng UX/process tuning.
- Điều phối viên đánh giá usefulness <3,5/5 hoặc tỷ lệ sử dụng <60%.
- Không được phê duyệt về quyền riêng tư/dữ liệu vị trí.

---

## 5.3. Kế hoạch pilot 4 tuần

| Giai đoạn | Hoạt động | Exit criteria |
|---|---|---|
| **Tuần 0 — Discovery** | Phỏng vấn Ops; xác nhận SOP; lấy 50 case ẩn danh; đo baseline | Data dictionary + policy owner + baseline được ký xác nhận |
| **Tuần 1 — Offline prototype** | Rule engine, structured output, 200 test case, adversarial suite | 100% boundary; 0 unauthorized action |
| **Tuần 2 — Shadow mode** | Chạy song song, không ảnh hưởng quyết định thực | ≥90% fact-grounded draft; latency trong ngưỡng |
| **Tuần 3 — Limited pilot** | 5–10 điều phối viên, một ca/khu vực, HITL bắt buộc | P90 <3 phút; acceptance ≥90% |
| **Tuần 4 — Review** | Phân tích lỗi, chi phí, adoption và security | Hội đồng quyết định mở rộng / sửa / dừng |

### RACI rút gọn

| Hạng mục | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|
| Safety policy | Ops Product Owner | Head of Operations | Safety/Legal/Security | Điều phối viên |
| Rule + LLM prototype | AI/Product Engineering | Tech Lead | Ops SMEs | Project sponsor |
| Dữ liệu và quyền truy cập | Data Engineer | Data Owner | Privacy/Security | Nhóm pilot |
| Pilot operation | Shift Lead | Ops Product Owner | AI team | Stakeholders |
| Go/No-Go production | Product Council | Executive sponsor | Ops, Tech, Legal, Security | Người dùng liên quan |

---

## 5.4. Ước lượng chi phí sơ bộ

> Đây là **planning estimate phục vụ bài lab**, không phải báo giá nhà cung cấp. Chi phí API phải được cập nhật theo bảng giá và model thực tế tại thời điểm triển khai.

| Hạng mục một lần | Giả định | Khoảng ước lượng |
|---|---|---:|
| Discovery + chuẩn hóa SOP/dữ liệu | 1 Product/BA + Ops SME, 1–2 tuần | 25–40 triệu VNĐ |
| Prototype rule + LLM + UI review | 1–2 engineer, 2–3 tuần | 70–110 triệu VNĐ |
| Tích hợp sandbox + audit/telemetry | Data/backend/security support | 35–60 triệu VNĐ |
| QA, adversarial test, training pilot | Test set + 5–10 người dùng | 20–35 triệu VNĐ |
| **Tổng pilot dự kiến** | Không gồm tích hợp production sâu | **150–245 triệu VNĐ** |

**Chi phí vận hành pilot/tháng:** giả định 15–30 triệu VNĐ cho monitoring, support và hạ tầng; API LLM được tính riêng theo:

```text
LLM cost/month
= số incident
× số lần gọi trung bình/incident
× (input tokens × input unit price + output tokens × output unit price)
```

Không nên dùng một con số token cost cố định trong báo cáo vì model và bảng giá có thể thay đổi. Trước pilot cần chạy load test trên phân phối prompt thật để lập ngân sách và spend cap.

---

## 5.5. Kết luận

Đây không phải bài toán “thay điều phối viên bằng AI”. Giá trị nằm ở việc **rút ngắn thao tác tổng hợp và soạn thảo**, trong khi làm cho rule an toàn, nguồn dữ liệu và quyết định phê duyệt trở nên rõ ràng hơn.

Khuyến nghị cuối cùng là xây một co-pilot **scope hẹp, least-privilege, audit-first**. Chỉ xem xét mở rộng sau khi prototype chứng minh được cả ba điều: nhanh hơn, hữu ích hơn và không phá vỡ boundary trong mọi case trọng yếu.

