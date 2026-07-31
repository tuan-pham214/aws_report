---
title: "Nền tảng và Quản trị"
date: 2026-07-31
weight: 1
chapter: false
pre: "<b>1. </b>"
---

## 1. Mục tiêu

Thiết lập một môi trường AWS thống nhất cho toàn nhóm, kiểm soát chi phí phát sinh và bảo đảm mọi thành viên triển khai tài nguyên theo cùng kiến trúc, Region và quy ước quản lý.

## 2. Chuẩn hóa môi trường AWS

Nhóm thống nhất triển khai toàn bộ tài nguyên tại:

```text
ap-southeast-1 - Asia Pacific (Singapore)
```

Các yêu cầu chung:

- Kiểm tra đúng tài khoản AWS và Region trước khi tạo tài nguyên.
- Không tự ý triển khai tại Region khác khi chưa thống nhất với nhóm.
- Báo lại tên tài nguyên, dịch vụ và mục đích sử dụng sau khi tạo.
- Sử dụng chung quy ước đặt tên và gắn thẻ.
- Dùng đúng hồ sơ AWS CLI hoặc tài khoản AWS Console được cấp.

## 3. Ngân sách và giám sát chi phí

AWS Budgets được thiết lập để theo dõi đồng thời **chi phí thực tế** và **mức sử dụng tài nguyên** trong quá trình triển khai dự án.

Nhóm sử dụng hai ngân sách chính:

- `My Monthly Cost Budget`: theo dõi tổng chi phí AWS theo tháng với hạn mức `100 USD`.
- `Daily usage`: theo dõi thời gian sử dụng tài nguyên với hạn mức `0.2 giờ`.

Tại thời điểm kiểm tra, chi phí tháng vẫn thấp hơn nhiều so với hạn mức. Ngân sách ở trạng thái `Healthy` và chưa vượt ngưỡng cảnh báo.

![Tổng quan AWS Budgets của dự án](/images/5-Workshop/5.3-DevOps/aws-budget-overview.png)

### Các ngưỡng cảnh báo chi phí

```text
12.5%  -> Cảnh báo khi chi phí thực tế vượt 12.50 USD
25%    -> Cảnh báo khi chi phí thực tế vượt 25.00 USD
50%    -> Cảnh báo khi chi phí thực tế vượt 50.00 USD
85%    -> Cảnh báo khi chi phí thực tế vượt 85.00 USD
90%    -> Cảnh báo khi chi phí thực tế vượt 90.00 USD
100%   -> Cảnh báo khi chi phí thực tế vượt 100.00 USD
100%   -> Cảnh báo khi chi phí dự báo vượt 100.00 USD
```

![Chi tiết ngân sách và cấu hình cảnh báo chi phí](/images/5-Workshop/5.3-DevOps/aws-budget-alerts.png)

### Quy ước kiểm soát nội bộ

```text
Từ 12.5 USD -> Kiểm tra dịch vụ đang phát sinh chi phí
Từ 25 USD   -> Rà soát tài nguyên đang chạy và người phụ trách
Từ 50 USD   -> Hạn chế tạo thêm tài nguyên không cần thiết
Từ 85 USD   -> Tạm dừng các tài nguyên thử nghiệm
Từ 90 USD   -> Chỉ duy trì tài nguyên cần thiết cho MVP
Từ 100 USD  -> Dừng và kiểm tra toàn bộ tài nguyên tính phí
```

### Các dịch vụ cần theo dõi sát

Các dịch vụ cần theo dõi sát gồm Amazon EC2, SageMaker Processing, SageMaker Training, SageMaker Endpoint, CloudWatch Logs và truyền dữ liệu.

Trước khi tạo tài nguyên có khả năng phát sinh chi phí, thành viên cần thông báo:

```text
Dịch vụ:
Tên tài nguyên:
Loại máy hoặc cấu hình:
Mục đích:
Thời gian dự kiến chạy:
Người phụ trách:
```

## 4. Hạn mức dịch vụ

Bên cạnh ngân sách, nhóm theo dõi AWS Service Quotas để tránh bị chặn khi triển khai MVP, đặc biệt với SageMaker và các loại máy có hạn mức riêng.

Việc kiểm tra sớm giúp nhóm biết tài nguyên nào có thể tạo, tránh phụ thuộc hoàn toàn vào trình diễn trực tiếp và chuẩn bị phương án dự phòng khi cần.

![Trạng thái hạn mức dịch vụ của SageMaker](/images/5-Workshop/5.3-DevOps/sagemaker-service-quota.png)

## 5. Kiến trúc, quy ước đặt tên và gắn thẻ

Luồng kiến trúc tổng thể:

```text
Simulator
-> AWS IoT Core
-> IoT Rule
-> Kinesis Data Firehose
-> S3 Raw
-> Data Processing
-> S3 Processed
-> SageMaker
-> FastAPI
-> SNS
```

Quy ước đặt tên:

```text
local-aqi-{environment}-{resource-purpose}
```

Ví dụ:

```text
local-aqi-dev-s3-raw
local-aqi-dev-s3-processed
local-aqi-dev-firehose-to-s3
local-aqi-dev-sagemaker-execution-role
```

Các thẻ bắt buộc:

```text
Project=local-aqi
Environment=dev
Owner=<member-name>
ManagedBy=manual
CostCenter=student-project
```

![Các thẻ quản lý tài nguyên AWS của dự án](/images/5-Workshop/5.3-DevOps/aws-resource-tags.png)
