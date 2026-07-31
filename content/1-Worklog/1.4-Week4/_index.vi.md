---
title: "Worklog Tuần 4"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
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

- Tổ chức cấu trúc Data Lake trên Amazon S3 để tách dữ liệu thô và dữ liệu đã xử lý.
- Khởi tạo backend FastAPI theo cấu trúc dễ kiểm thử, cấu hình và mở rộng.
- Chuẩn bị môi trường EC2 an toàn cho việc triển khai API và tích hợp các dịch vụ AWS.

### Công việc theo ngày/thời gian

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Kết quả |
| --- | --- | --- | --- | --- |
| 2 | Phối hợp thiết kế bucket/prefix cho vùng `raw` và `processed`, partition theo trạm và thời gian, quy tắc đặt tên object và metadata truy vết. Xem xét versioning, encryption và lifecycle cho dữ liệu thử nghiệm. | 07/07/2025 | 07/07/2025 | Hoàn thành cấu trúc Data Lake thống nhất, hỗ trợ xử lý theo lô và tránh trộn dữ liệu gốc với dữ liệu đã làm sạch. |
| 3 | Rà soát quyền truy cập S3 cho Firehose, SageMaker Processing và backend. Tách quyền ghi/đọc theo vai trò, tránh dùng policy phạm vi toàn bộ tài khoản. | 08/07/2025 | 08/07/2025 | Hoàn thành ma trận truy cập ở mức thành phần và bộ policy giới hạn theo tài nguyên. |
| 4 | Khởi tạo ứng dụng FastAPI theo các module cấu hình, schema, router và service. Xây dựng endpoint kiểm tra sức khỏe, cấu hình qua biến môi trường và tài liệu API tự động. | 09/07/2025 | 09/07/2025 | Backend skeleton khởi động thành công, có `/health` và cấu trúc cho các chức năng dự báo/cảnh báo. |
| 5 | Định nghĩa schema request/response cho `forecast`, `subscribe` và `alert`; bổ sung validation cho `station_id`, email, ngưỡng AQI và dữ liệu PM2.5. Chuẩn hóa lỗi để không lộ chi tiết nội bộ. | 10/07/2025 | 10/07/2025 | Hoàn thành hợp đồng OpenAPI ban đầu và quy tắc trả lỗi 4xx/5xx nhất quán. |
| 6 | Triển khai FastAPI trên EC2 bằng tiến trình dịch vụ, IAM instance role và cấu hình Security Group tối thiểu. Kiểm tra log khởi động và endpoint sức khỏe sau triển khai. | 11/07/2025 | 11/07/2025 | Backend hoạt động trên EC2, kèm quy trình triển khai lặp lại được và checklist kiểm tra sau cập nhật. |

### Kết quả đạt được

- Hoàn thành thiết kế logic cho vùng S3 `raw`/`processed`, bao gồm partition, mã hóa và vòng đời dữ liệu.
- Xây dựng backend FastAPI dạng module với cấu hình tách khỏi mã nguồn và endpoint kiểm tra sức khỏe.
- Chuẩn hóa schema và cách phản hồi lỗi cho các API cốt lõi.
- Thiết lập nguyên tắc dùng IAM role cho EC2, không nhúng khóa AWS vào ứng dụng.
- Chuẩn bị nền tảng để tích hợp kết quả xử lý dữ liệu và chức năng đăng ký cảnh báo trong tuần tiếp theo.
