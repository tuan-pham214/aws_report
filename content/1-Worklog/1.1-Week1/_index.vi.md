---
title: "Worklog Tuần 1"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
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

- Nắm được mô hình trách nhiệm chia sẻ, cấu trúc Region/Availability Zone và các nhóm dịch vụ AWS nền tảng.
- Làm quen với AWS Management Console, AWS CLI, IAM và nguyên tắc không lưu thông tin truy cập trong mã nguồn.
- Hiểu vai trò của EC2, VPC, Security Group, S3 và các dịch vụ giám sát đối với backend của hệ thống AQI.

### Công việc theo ngày/thời gian

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Kết quả |
| --- | --- | --- | --- | --- |
| 2 | Làm quen với quy trình làm việc, xác định phạm vi **Local AQI Forecasting & Alert System** và vai trò Backend/API Engineer. Phác thảo các thành phần từ nguồn telemetry đến API và cảnh báo. | 16/06/2025 | 16/06/2025 | Có bức tranh tổng thể về dự án, đầu vào/đầu ra của backend và ranh giới phối hợp với nhóm Data/ML. |
| 3 | Tìm hiểu hạ tầng toàn cầu AWS, Region, Availability Zone, mô hình trách nhiệm chia sẻ và các nhóm Compute, Storage, Networking, Security. | 17/06/2025 | 17/06/2025 | Phân biệt được trách nhiệm bảo mật của AWS và của nhóm triển khai; chọn Region nhất quán cho môi trường phát triển. |
| 4 | Thực hành điều hướng AWS Console và AWS CLI; tìm hiểu IAM user, role, policy, MFA và nguyên tắc quyền tối thiểu. Thiết lập cách dùng profile cục bộ, không đưa access key vào source code. | 18/06/2025 | 18/06/2025 | Thực hiện được các thao tác đọc thông tin tài khoản/tài nguyên bằng CLI và xây dựng quy tắc quản lý thông tin xác thực an toàn. |
| 5 | Tìm hiểu EC2, AMI, EBS, VPC, subnet, route table và Security Group. Xem xét cách FastAPI có thể chạy trên EC2 và nhận lưu lượng qua các cổng cần thiết. | 19/06/2025 | 19/06/2025 | Mô tả được đường đi của request tới backend và biết cách giới hạn inbound rule thay vì mở rộng toàn bộ cổng quản trị. |
| 6 | Tìm hiểu S3, CloudWatch, CloudTrail và AWS Budgets; lập checklist gắn tag, theo dõi log, audit thao tác và cảnh báo chi phí cho tài nguyên thử nghiệm. | 20/06/2025 | 20/06/2025 | Hoàn thành bộ nguyên tắc nền tảng dùng chung cho các tuần triển khai tiếp theo. |

### Kết quả đạt được

- Nắm được các khái niệm AWS cần thiết để đọc và trao đổi kiến trúc của dự án.
- Sử dụng được Console và CLI cho các tác vụ cơ bản, đồng thời hiểu vì sao backend trên EC2 nên dùng IAM role thay cho khóa truy cập tĩnh.
- Giải thích được mối quan hệ giữa VPC, subnet, Security Group, EC2 và S3 trong môi trường triển khai.
- Xây dựng checklist ban đầu cho tagging, log, audit và ngân sách, làm cơ sở hạn chế rủi ro bảo mật và chi phí.
- Xác định rõ phần việc cá nhân tập trung vào API/backend; các hạng mục dữ liệu và mô hình sẽ được thực hiện theo hướng tìm hiểu hợp đồng dữ liệu, phối hợp và tích hợp với nhóm phụ trách.
