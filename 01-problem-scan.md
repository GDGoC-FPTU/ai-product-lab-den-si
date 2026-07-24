# 01 - Quét vấn đề và Quick Problem Cards

## Phase 1 - Quét vấn đề

| # | Công ty thành viên | Lens | Mô tả ngắn vấn đề |
|---|---|---|---|
| 1 | Xanh SM | Tốn thời gian | Tài xế Xanh SM thường đi sạc vào cùng một khung giờ, như giờ nghỉ trưa hoặc giờ giao ca. Điều này làm nhiều trạm sạc quá tải cục bộ; tài xế phải xếp hàng 1-2 giờ mới có trụ sạc, lãng phí thời gian kinh doanh. |
| 2 | Vinhomes | Nỗi đau của stakeholder | Khi có thời tiết cực đoan như bão hoặc ngập lụt, hệ thống có thể nhận đồng thời khoảng 500 yêu cầu như "thấm nước" hoặc "chập điện". Xử lý theo thứ tự ai báo trước (FIFO) có thể bỏ sót các ca có rủi ro lớn. |
| 3 | Vinpearl | AI có thể nâng cấp | Cần phát hiện sự không hài lòng của khách khi họ vẫn đang lưu trú để đội vận hành có thể xử lý ngay thay vì chỉ phản hồi sau khi khách đã rời đi. |
| 4 | VinFast | Lặp lại | Nhân viên bảo hành lặp lại việc trích xuất mẫu xe, triệu chứng, số km và lịch sử bảo dưỡng từ ticket dạng văn bản tự do. |
| 5 | Vinmec | Tốn thời gian | Nhân viên đặt lịch tự tóm tắt các yêu cầu đổi lịch không liên quan chẩn đoán trước khi xác nhận khung giờ mới. |

## Phase 2 - Đánh giá nhanh

### Quick Problem Card 1 - Giãn tải sạc theo khung giờ cho Xanh SM

| Trường | Nội dung |
|---|---|
| Bài toán | Dự báo nguy cơ quá tải tại trạm sạc và gợi ý khung giờ/trạm thay thế để tài xế không tập trung đi sạc cùng lúc. |
| Công ty thành viên | Xanh SM (GSM) |
| Ai đang đau | Tài xế phải xếp hàng 1-2 giờ; điều phối viên và đội vận hành trạm phải xử lý phản ánh khi quá tải. |
| Workflow thủ công hiện tại | 1. Tài xế tự quyết định thời điểm đi sạc. 2. Đến trạm quen thuộc. 3. Phát hiện hàng chờ. 4. Chờ hoặc tìm trạm khác. 5. Báo/than phiền khi thời gian chờ quá lâu. |
| Bước chậm/dễ lỗi nhất | Tài xế không có thông tin dự báo hàng chờ và thường đưa ra cùng một quyết định vào giờ giao ca; mất 1-2 giờ chờ trong các đợt quá tải. |
| Điểm AI hỗ trợ | Dự báo nhu cầu theo khu vực/khung giờ từ lịch sử sạc, trạng thái pin và lịch vận hành; tạo gợi ý giãn tải để tài xế và điều phối viên review. |
| Metric thành công | Giảm thời gian chờ sạc median trong giờ cao điểm ít nhất 30%; giảm tỷ lệ tài xế chờ quá 30 phút; không làm tăng tỷ lệ xe xuống dưới ngưỡng pin an toàn. |
| Kiến trúc nhanh | Forecasting/optimization + rule an toàn về ngưỡng pin và tương thích trạm. LLM chỉ dùng để giải thích/gửi bản nháp, không tự điều xe. |

### Quick Problem Card 2 - Phân loại ưu tiên sự cố thời tiết cực đoan tại Vinhomes

| Trường | Nội dung |
|---|---|
| Bài toán | Phân loại và ưu tiên hàng trăm yêu cầu đồng thời trong bão/ngập, để các ca có nguy cơ chập điện, ngập nghiêm trọng hoặc liên quan an toàn được xử lý trước FIFO. |
| Công ty thành viên | Vinhomes |
| Ai đang đau | Cư dân cần hỗ trợ khẩn; tổng đài viên, ban quản lý và đội kỹ thuật bị quá tải khi nhận khoảng 500 yêu cầu cùng lúc. |
| Workflow thủ công hiện tại | 1. Cư dân gửi yêu cầu. 2. Tổng đài tiếp nhận. 3. Nhân viên đọc từng ticket. 4. Phân loại mức độ nghiêm trọng. 5. Chuyển đội kỹ thuật/bảo vệ xử lý theo thứ tự. |
| Bước chậm/dễ lỗi nhất | Đọc và đánh giá độ nghiêm trọng từ văn bản tự do trong lúc volume tăng đột biến; FIFO có thể đẩy các ca nguy hiểm xuống sau. |
| Điểm AI hỗ trợ | Trích xuất loại sự cố, vị trí và dấu hiệu nguy hiểm; đề xuất mức ưu tiên và queue xử lý. Các ca điện, cháy, có người mắc kẹt hoặc confidence thấp bắt buộc chuyển supervisor. |
| Metric thành công | 95% ticket có dấu hiệu an toàn cao được gắn cờ trong dưới 60 giây; 100% ca nghi ngờ chập điện/cháy được human review trước khi đóng ticket. |
| Kiến trúc nhanh | Rule ưu tiên an toàn + LLM phân loại văn bản + human-in-the-loop. Không dùng agent tự xử lý hoặc tự đóng ticket. |

### Quick Problem Card 3 - Phát hiện khách không hài lòng khi đang lưu trú tại Vinpearl

| Trường | Nội dung |
|---|---|
| Bài toán | Phát hiện tín hiệu không hài lòng từ chat, cuộc gọi, khảo sát ngắn và yêu cầu dịch vụ khi khách vẫn còn lưu trú để có thể chủ động can thiệp. |
| Công ty thành viên | Vinpearl |
| Ai đang đau | Khách có trải nghiệm xấu nhưng chưa được hỗ trợ kịp thời; đội guest relations chỉ biết vấn đề sau review hoặc khiếu nại muộn. |
| Workflow thủ công hiện tại | 1. Khách gửi chat/yêu cầu dịch vụ. 2. Nhân viên đọc từng tương tác. 3. Ghi nhận hoặc bỏ sót tín hiệu tiêu cực. 4. Phân công xử lý nếu khách khiếu nại rõ. 5. Phản hồi sau đó. |
| Bước chậm/dễ lỗi nhất | Nhận diện tín hiệu tiêu cực gián tiếp, như "phòng vẫn chưa được dọn" hoặc nhiều yêu cầu lặp lại, trên nhiều kênh. |
| Điểm AI hỗ trợ | Tóm tắt tương tác theo khách/lưu trú, phân loại cảm xúc và mức độ khẩn, sau đó tạo cảnh báo để nhân viên guest relations kiểm tra. |
| Metric thành công | Phát hiện 85% khách có tín hiệu không hài lòng trên tập test đã gắn nhãn; 90% cảnh báo hợp lệ được nhân viên xem trong 15 phút; đo mức cải thiện CSAT sau pilot. |
| Kiến trúc nhanh | LLM feature cho tổng hợp/phân loại + rule cho SLA/cảnh báo + nhân viên quyết định cách liên hệ, xin lỗi hoặc bồi hoàn. |

## Lựa chọn của nhóm

Nhóm cần chọn một trong ba Quick Problem Cards trước khi cập nhật `02-deep-dive-report.md` và `starter-code/prompt_prototype.py` cho cùng một case.

Khuyến nghị chọn **Card 2 - Phân loại ưu tiên sự cố thời tiết cực đoan tại Vinhomes**: pain point rõ, có tác động an toàn, metric có thể kiểm chứng và ranh giới human review dễ bảo vệ. Card 1 phù hợp khi nhóm có dữ liệu lịch sử sạc đủ sạch; Card 3 phù hợp khi nhóm có quyền truy cập dữ liệu tương tác khách hàng theo thời gian thực.
