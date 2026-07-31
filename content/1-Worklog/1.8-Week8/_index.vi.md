---
title: "Worklog Tuần 8"
weight: 8
chapter: false
pre: " <b> 1.8. </b> "
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

- Hoàn thiện tài liệu kỹ thuật, hướng dẫn triển khai và bàn giao backend.
- Thực hiện kịch bản demo thể hiện được luồng dữ liệu, dự báo và cảnh báo.
- Tổng hợp kết quả dự án và hoàn thiện báo cáo thực tập trước hạn nộp ngày 31/07/2025.

### Công việc theo ngày/thời gian

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Kết quả |
| --- | --- | --- | --- | --- |
| Buổi sáng | Rà soát OpenAPI, mô tả endpoint, ví dụ request/response, mã lỗi và trạng thái nghiệp vụ. Hoàn thiện hướng dẫn triển khai FastAPI lên EC2, cấu hình IAM role, kiểm tra service/health, rollback và chẩn đoán qua CloudWatch hoặc SSM. | 30/07/2025 | 30/07/2025 | Hoàn thành tài liệu API và runbook triển khai để các thành viên có thể chạy thử, tích hợp và bàn giao hệ thống. |
| Buổi chiều | Chạy kịch bản demo: phát telemetry, kiểm tra object tại S3, gọi API dự báo, đăng ký người nhận và gửi cảnh báo theo ngưỡng. Kiểm tra lại log, quyền truy cập, cảnh báo ngân sách và kết quả tích hợp của các thành phần ingestion, Data/ML và backend. | 30/07/2025 | 30/07/2025 | Demo thành công toàn bộ luồng từ dữ liệu telemetry đến dự báo AQI và email cảnh báo; xác nhận các thành phần hoạt động đúng kiến trúc. |
| Cuối ngày | Tổng hợp kiến trúc, công việc, kết quả và bài học vào báo cáo Hugo; rà soát liên kết Tuần 1–8, thuật ngữ tiếng Việt, bảng nhật ký và tính nhất quán của nội dung trước khi nộp. | 30/07/2025 | 30/07/2025 | Hoàn thiện Worklog tiếng Việt, bộ tài liệu bàn giao, nội dung demo và báo cáo thực tập trước hạn ngày 31/07/2025. |

### Kết quả đạt được

- Hoàn thiện tài liệu OpenAPI, hướng dẫn triển khai, rollback, chẩn đoán và bàn giao backend.
- Trình diễn thành công luồng telemetry → S3 → SageMaker → FastAPI → Amazon SNS.
- Xác nhận các thành phần ingestion, Data Lake, mô hình dự báo, backend và cảnh báo đã tích hợp hoàn chỉnh.
- Hoàn thành rà soát giám sát, bảo mật và kiểm soát chi phí trước khi bàn giao.
- Hoàn thành báo cáo Worklog tiếng Việt trước ngày 31/07/2025, tập trung vào đóng góp Backend/API Engineer và sự phối hợp với các nhóm liên quan.
