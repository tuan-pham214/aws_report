---
title: "Worklog Tuần 2"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần
  - Công việc theo ngày/thời gian
  - Kết quả đạt được
reportType: worklog
---

### Mục tiêu tuần

- Tìm hiểu các dịch vụ AWS phù hợp với luồng thu thập, lưu trữ, dự báo và cảnh báo AQI.
- Xây dựng kiến trúc logic, xác định hợp đồng giao tiếp và trách nhiệm của từng thành phần.
- Đánh giá các lựa chọn triển khai theo tiêu chí bảo mật, khả năng mở rộng và chi phí của môi trường thực tập.

### Công việc theo ngày/thời gian

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Kết quả |
| --- | --- | --- | --- | --- |
| 2 | Tìm hiểu OpenAQ, telemetry simulator bằng Python, giao thức MQTT, cấu trúc topic và cơ chế Quality of Service. So sánh AWS IoT Core với Mosquitto chạy trên EC2 cho môi trường thử nghiệm. | 23/06/2025 | 23/06/2025 | Xác định được hai phương án tiếp nhận dữ liệu và các tiêu chí lựa chọn: công sức vận hành, bảo mật kết nối, chi phí và khả năng tích hợp AWS. |
| 3 | Tìm hiểu Kinesis Data Firehose, cơ chế buffer, retry và đích Amazon S3. Thiết kế cách tách vùng dữ liệu `raw` và `processed` theo trạm và thời gian. | 24/06/2025 | 24/06/2025 | Hoàn thành sơ đồ đường đi của bản ghi từ MQTT đến S3 và quy ước prefix phục vụ truy vết. |
| 4 | Tìm hiểu SageMaker Processing, DeepAR và SageMaker Endpoint. Trao đổi với nhóm Data/ML về dữ liệu đầu vào, khoảng dự báo 24–48 giờ và định dạng kết quả mà API cần sử dụng. | 25/06/2025 | 25/06/2025 | Hoàn thành bản nháp hợp đồng request/response giữa backend và endpoint dự báo, không gắn chặt API với chi tiết huấn luyện mô hình. |
| 5 | Thiết kế lớp backend FastAPI trên EC2, các API `health`, `forecast`, `subscribe` và `alert`; tìm hiểu Amazon SNS cho xác nhận đăng ký và gửi email cảnh báo theo ngưỡng. | 26/06/2025 | 26/06/2025 | Xác định trách nhiệm chính của backend và luồng nghiệp vụ từ yêu cầu người dùng đến SNS. |
| 6 | Rà soát kiến trúc dưới góc nhìn IAM, VPC/Security Group, CloudWatch, CloudTrail và AWS Budgets. Xác định cách xử lý mất kết nối MQTT, Firehose retry, SageMaker timeout và trạng thái đăng ký SNS `pending`. | 27/06/2025 | 27/06/2025 | Hoàn thành kiến trúc logic kèm các điểm kiểm soát bảo mật, giám sát, lỗi và chi phí. |

### Kết quả đạt được

- Mô tả được kiến trúc đầu cuối: OpenAQ/Python simulator → MQTT → IoT Core hoặc Mosquitto → Firehose → S3 → SageMaker → FastAPI → SNS.
- Phân biệt được vai trò của vùng dữ liệu thô, dữ liệu đã xử lý và endpoint phục vụ dự báo.
- Xây dựng bản nháp hợp đồng dữ liệu cho telemetry và dự báo để các nhóm có thể phát triển độc lập.
- Xác định các API cốt lõi và các trạng thái lỗi cần trả về rõ ràng cho phía sử dụng.
- Ghi nhận phương án vận hành tối thiểu theo nguyên tắc quyền tối thiểu, chỉ mở cổng cần thiết và theo dõi ngân sách ngay từ đầu.
