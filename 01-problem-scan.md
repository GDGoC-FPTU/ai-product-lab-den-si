# 01 - Quét vấn đề và Quick Problem Cards

## Phase 1 - Quét vấn đề

| #   | Công ty thành viên | Lens                    | Mô tả ngắn vấn đề                                                                                                                                                                                                                           |
| --- | ------------------ | ----------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1   | Xanh SM            | Tốn thời gian           | Tài xế Xanh SM thường có xu hướng đi sạc vào cùng một khung giờ (giờ nghỉ trưa hoặc giờ giao ca). Điều này dẫn đến việc nhiều trạm sạc bị quá tải cục bộ, tài xế phải xếp hàng 1-2 tiếng mới có trụ sạc, gây lãng phí thời gian kinh doanh. |
| 2   | Vinhomes           | Nỗi đau của stakeholder | Khi có một sự kiện thời tiết cực đoan (như bão, ngập lụt), hệ thống có thể nhận cùng lúc 500 yêu cầu "thấm nước" hoặc "chập điện". Nếu xử lý theo nguyên tắc "Ai báo trước sửa trước" (FIFO) sẽ dẫn đến rủi ro lớn.                         |
| 3   | Vinpearl           | AI có thể nâng cấp      | Nhân viên tự gom các đánh giá điểm thấp và soạn phản hồi, nên các vấn đề dịch vụ lặp lại được phát hiện muộn.                                                                                                                               |
| 4   | VinFast            | Lặp lại                 | Nhân viên bảo hành lặp lại việc trích xuất mẫu xe, triệu chứng, số km và lịch sử bảo dưỡng từ ticket dạng văn bản tự do.                                                                                                                    |
| 5   | Vinmec             | Tốn thời gian           | Nhân viên đặt lịch tự tóm tắt các yêu cầu đổi lịch không liên quan chẩn đoán trước khi xác nhận khung giờ mới.                                                                                                                              |

## Phase 2 - Đánh giá nhanh

### Quick Problem Card 1 - Phân luồng hỗ trợ pin Xanh SM

| Trường                     | Nội dung                                                                                                                                                                                   |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Bài toán                   | Hỗ trợ điều phối viên phân luồng sự cố pin yếu do tài xế báo và chuẩn bị phản hồi an toàn.                                                                                                 |
| Công ty thành viên         | Xanh SM (GSM)                                                                                                                                                                              |
| Ai đang đau                | Tài xế chờ hỗ trợ bên đường; điều phối viên xử lý đồng thời nhiều sự cố.                                                                                                                   |
| Workflow thủ công hiện tại | 1. Tài xế báo sự cố. 2. Điều phối viên kiểm tra pin và GPS. 3. Điều phối viên kiểm tra trạm sạc còn chỗ. 4. Điều phối viên chọn phương án an toàn. 5. Điều phối viên soạn và gửi phản hồi. |
| Bước chậm/dễ lỗi nhất      | Tra cứu tình trạng trạm sạc và soạn phản hồi, ước tính 6 phút mỗi sự cố.                                                                                                                   |
| Điểm AI hỗ trợ             | Tóm tắt báo cáo, phân loại mức khẩn và tạo bản nháp để duyệt. Ngưỡng pin và quyết định điều phối vẫn là rule xác định.                                                                     |
| Metric thành công          | Giảm median triage time từ 8 phút xuống dưới 2 phút, đồng thời giữ số vi phạm routing khi pin nguy cấp bằng 0.                                                                             |
| Kiến trúc nhanh            | Hybrid: Rule/state machine cho hành động an toàn + LLM feature để tóm tắt và soạn bản nháp.                                                                                                |

### Quick Problem Card 2 - Chuyển tuyến yêu cầu cư dân Vinhomes

| Trường                     | Nội dung                                                                                                                                 |
| -------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| Bài toán                   | Phân loại yêu cầu dịch vụ dạng văn bản tự do của cư dân và đề xuất đúng tuyến xử lý.                                                     |
| Công ty thành viên         | Vinhomes                                                                                                                                 |
| Ai đang đau                | Nhân viên CSKH và cư dân đang chờ xác nhận đã tiếp nhận yêu cầu.                                                                         |
| Workflow thủ công hiện tại | 1. Cư dân gửi yêu cầu. 2. Nhân viên đọc yêu cầu. 3. Xác định tòa nhà, loại việc và mức khẩn. 4. Chuyển tuyến. 5. Soạn tin nhắn xác nhận. |
| Bước chậm/dễ lỗi nhất      | Phân loại và handoff, ước tính 4 phút mỗi ticket; các yêu cầu mơ hồ thường bị chuyển lại nhiều lần.                                      |
| Điểm AI hỗ trợ             | Trích xuất vấn đề/nhóm xử lý/vị trí và đề xuất queue; nhân viên duyệt trước khi giao việc.                                               |
| Metric thành công          | Đúng tuyến ngay lần đầu 85% trên tập test tách biệt và median phản hồi đầu tiên dưới 5 phút.                                             |
| Kiến trúc nhanh            | LLM feature kết hợp bảng routing theo rule và human review.                                                                              |

### Quick Problem Card 3 - Phân luồng đánh giá điểm thấp Vinpearl

| Trường                     | Nội dung                                                                                                   |
| -------------------------- | ---------------------------------------------------------------------------------------------------------- |
| Bài toán                   | Gom các đánh giá điểm thấp theo vấn đề dịch vụ và soạn phản hồi đúng giọng thương hiệu để nhân viên duyệt. |
| Công ty thành viên         | Vinpearl                                                                                                   |
| Ai đang đau                | Nhân viên guest experience phải đọc bình luận trên nhiều kênh.                                             |
| Workflow thủ công hiện tại | 1. Xuất đánh giá. 2. Đọc từng đánh giá. 3. Gắn nhãn vấn đề. 4. Nhận diện khiếu nại khẩn. 5. Soạn phản hồi. |
| Bước chậm/dễ lỗi nhất      | Đọc và gắn nhãn theo lô, ước tính 2 phút mỗi đánh giá.                                                     |
| Điểm AI hỗ trợ             | Tóm tắt, phân loại cảm xúc/vấn đề và tạo bản nháp phản hồi; nhân viên duyệt mọi giao tiếp ra bên ngoài.    |
| Metric thành công          | Gắn nhãn 90% đánh giá trong dưới 15 giây và đưa tỷ lệ phản hồi trong ngày lên 95% sau khi có baseline.     |
| Kiến trúc nhanh            | LLM feature; không tự đăng phản hồi hoặc quyết định bồi hoàn.                                              |

## Lựa chọn của nhóm

Nhóm chọn **Quick Problem Card 1: Phân luồng hỗ trợ pin Xanh SM**. Bài toán có workflow hẹp, quan sát được, ranh giới an toàn rõ và điểm duyệt của con người ngay trước hành động thực tế. Các số liệu về thời gian và sản lượng ở trên là giả định phục vụ bài lab, không phải số liệu vận hành chính thức; nhóm cần kiểm chứng bằng log sự cố trước khi triển khai.
