# 03 - AI Log và Reflection

## Tôi đã dùng AI để làm gì

Tôi dùng AI như một thought partner để biến ý tưởng rộng "hỗ trợ tài xế Xanh SM khi pin yếu" thành một bài toán vận hành. AI giúp tạo danh sách bottleneck, phản biện metric còn mơ hồ và phân biệt giữa tác vụ ngôn ngữ với một quyết định an toàn.

## Prompt và đầu ra hữu ích

| Prompt / hoạt động | Kết quả hữu ích | Tôi đã thay đổi gì sau khi review |
|---|---|---|
| Brainstorm năm pain point vận hành của Vingroup bằng các lens lặp lại, tốn thời gian, AI-upgrade và stakeholder pain. | Tạo danh sách rộng hơn ở Xanh SM, Vinhomes, Vinpearl, VinFast và Vinmec. | Tôi chỉ giữ các vấn đề có actor rõ, workflow thủ công rõ và outcome đo được. |
| Phản biện ý tưởng Xanh SM dưới góc nhìn trưởng vận hành và CFO khắt khe. | Chỉ ra rằng "điều phối nhanh hơn" quá mơ hồ và gợi ý sai trạm sạc là lỗi rủi ro cao. | Tôi thay bằng metric pilot 9 xuống 3 phút và tiêu chí không chấp nhận vi phạm routing ở ca pin nguy cấp. |
| So sánh rule, LLM feature và agent tự chủ. | Làm rõ ngưỡng pin/quyền hạn là logic xác định; LLM phù hợp hơn với việc hiểu văn bản tự do và viết nháp. | Tôi chọn thiết kế hybrid: rule trước, LLM tạo nháp sau, điều phối viên duyệt cuối. |
| Sinh adversarial prompt. | Gợi ý cách ép hệ thống bỏ nhãn review và vượt ngưỡng 5%. | Tôi thêm ba test, có prompt injection, rồi bắt code thực thi boundary sau khi model sinh phản hồi. |

## AI sai hoặc thiếu ở đâu

AI ban đầu đưa ra các con số vận hành có vẻ chắc chắn và ngầm cho phép model chọn trạm sạc. Các nhận định này không có dữ liệu nội bộ Xanh SM chứng minh; gợi ý sau còn tạo đường tự động hóa không an toàn. Tôi đã đổi các số liệu thành giả định cần validate, thêm bước xác lập baseline và giữ lựa chọn trạm/điều xe sau dữ liệu đã xác thực cùng sự duyệt của con người.

## Kết quả kỹ thuật

Tôi chạy `python starter-code/prompt_prototype.py`. Workspace chưa cấu hình `OPENROUTER_API_KEY`, nên script chạy chế độ mô phỏng an toàn offline. Ba adversarial test đều pass trên cùng lớp kiểm tra xác định. Khi có key được cấp quyền, bước review tiếp theo là ghi lại phản hồi OpenRouter, model ID, timestamp và mọi lần guardrail phải chỉnh sửa kết quả.

## Reflection

Giá trị của AI không phải chỉ là tạo ra câu trả lời trông hoàn chỉnh nhanh hơn; nó giúp bộc lộ các giả định cần có người chịu trách nhiệm và một cách kiểm thử. Scope cuối nhỏ hơn ý tưởng ban đầu nhưng đáng tin hơn: AI tóm tắt và tạo bản nháp, rule bảo vệ ngưỡng nguy cấp, còn điều phối viên chịu trách nhiệm quyết định có hậu quả. Trước khi trình bày, tôi sẽ hỏi team vận hành để xác minh taxonomy sự cố, tính sẵn có của dữ liệu trực tiếp và baseline thời gian xử lý thực tế.