---

title: "Giới thiệu"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.1. </b> "
----------------------

#### Mục tiêu phần thực hành

Phần này mô tả cách nhóm xây dựng một hệ thống dự báo và cảnh báo chất lượng không khí cục bộ trên AWS theo hướng có thể trình diễn lại bằng một luồng đầu cuối rõ ràng.

Thay vì trình bày riêng lẻ theo từng dịch vụ AWS, nội dung workshop được chia theo **vai trò thực tế của các thành viên trong nhóm**:

* DevOps
* Ingestion
* Data Preparation
* Machine Learning
* Backend

#### Bài toán cần giải quyết

Hệ thống cần tiếp nhận dữ liệu từ nhiều trạm đo giả lập, lưu dữ liệu gốc lên Amazon S3, xử lý dữ liệu thành dataset phục vụ Machine Learning, dự báo nồng độ PM2.5 trong 24 giờ tiếp theo, trả kết quả thông qua FastAPI và gửi cảnh báo bằng Amazon SNS khi chỉ số vượt ngưỡng.

MVP hiện tại ưu tiên hoàn thành luồng:

```text
Simulator
-> AWS IoT Core
-> IoT Rule
-> Amazon Data Firehose
-> Amazon S3 Raw
```

Milestone này chỉ được xem là hoàn thành khi object dữ liệu thực tế xuất hiện trong bucket `local-aqi-dev-s3-raw`.

#### Tổng quan kiến trúc

Kiến trúc tổng thể của dự án:

```text
MQTT Simulator
    -> AWS IoT Core
    -> IoT Rule
    -> Amazon Data Firehose
    -> Amazon S3 Raw
    -> Data Processing
    -> Amazon S3 Processed
    -> Amazon SageMaker Processing / Training
    -> Forecast Model / Forecast Result
    -> FastAPI trên Amazon EC2
    -> Amazon SNS Email Alert
```

![Kiến trúc tổng thể của dự án](/images/5-Workshop/5.1-Workshop-overview/5.3-devops-local-aqi-final-architecture.png)

#### Cách đọc phần thực hành

Mỗi vai trò được trình bày theo một cấu trúc chung:

1. Mục tiêu của vai trò.
2. Các bước đã thực hiện.
3. Tài nguyên AWS, mã nguồn hoặc cấu hình đã tạo.
4. Kết quả đã đạt được.
5. Evidence và cách demo.

Cấu trúc này được giữ thống nhất để liên kết các phần thành một câu chuyện triển khai xuyên suốt từ đầu đến cuối.
