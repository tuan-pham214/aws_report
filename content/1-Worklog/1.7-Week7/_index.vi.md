---
title: "Worklog Tuần 7"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
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

- Kiểm thử các luồng chính, trường hợp biên và lỗi tích hợp của backend.
- Hoàn thiện giám sát, audit và các biện pháp bảo mật cho môi trường AWS.
- Rà soát chi phí, loại bỏ cấu hình dư thừa và chốt checklist vận hành trước khi hoàn thiện báo cáo.

### Công việc theo ngày/thời gian

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Kết quả |
| --- | --- | --- | --- | --- |
| 2 | Mở rộng unit test cho công thức quy đổi PM2.5–AQI, các biên ngưỡng, dữ liệu không hợp lệ và response theo OpenAPI. Chạy integration test cho đăng ký trùng, trạng thái SNS, lỗi DynamoDB, SageMaker timeout/sai schema và cooldown cảnh báo. | 28/07/2025 | 28/07/2025 | Bộ kiểm thử bao phủ luồng chính, các trường hợp biên và lỗi từ dịch vụ phụ thuộc; hệ thống tránh gửi cảnh báo trùng trong các request cạnh tranh. |
| 3 | Chuẩn hóa log ứng dụng và CloudWatch Logs; cấu hình metric/alarm cho API 5xx, độ trễ, Firehose delivery failure, SageMaker invocation error và SNS publish failure. Rà soát IAM theo quyền tối thiểu, Security Group, quản trị EC2 qua SSM, mã hóa S3 và CloudTrail. | 29/07/2025 | 29/07/2025 | Hoàn thành dashboard giám sát, cảnh báo, checklist bảo mật/audit và hướng xử lý cho các tín hiệu vận hành quan trọng. |
| 4 | Rà soát AWS Budgets, tag chi phí, kích thước EC2, buffer Firehose, retention CloudWatch và lifecycle S3. Chạy kiểm tra hồi quy sau điều chỉnh và chốt checklist vận hành phục vụ demo, tài liệu và bàn giao. | 30/07/2025 | 30/07/2025 | Hoàn thành phương án tối ưu chi phí cho quy mô MVP, xác nhận hệ thống ổn định và sẵn sàng cho bước tổng hợp báo cáo. |

### Kết quả đạt được

- Bộ kiểm thử bao phủ luồng health, forecast, subscribe, alert và các trường hợp biên quan trọng.
- Các lỗi từ DynamoDB, SageMaker và SNS được ánh xạ thành phản hồi an toàn, nhất quán.
- Hoàn thiện giám sát bằng CloudWatch và audit bằng CloudTrail cho các điểm có rủi ro cao.
- Rà soát quyền IAM, Security Group, quản lý EC2 và xác nhận log không chứa dữ liệu nhạy cảm.
- Hoàn thiện danh sách tối ưu chi phí gồm right-sizing, log retention, S3 lifecycle, tag và cảnh báo AWS Budgets.
