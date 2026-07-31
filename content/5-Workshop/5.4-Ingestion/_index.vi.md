---
title: "Thu nhận dữ liệu: Simulator → IoT Core → Firehose → S3 Raw"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.4. </b> "
---

#### Mục tiêu của vai trò

Nhóm Thu nhận dữ liệu chịu trách nhiệm chuyển telemetry từ Simulator vào AWS và chứng minh rằng dữ liệu thực tế đã xuất hiện trong S3 Raw.

Đây là cột mốc quan trọng nhất của MVP vì các phần Xử lý dữ liệu, Học máy và Backend đều cần dữ liệu thô thực tế làm đầu vào.

#### Luồng cần đạt

```text
Simulator
-> AWS IoT Core
-> IoT Rule
-> Kinesis Data Firehose
-> S3 Raw
```

## Triển khai từng bước: thu thập và định tuyến dữ liệu

### Bước 1: Kiểm tra tài nguyên trên AWS

Xác nhận delivery stream của **Amazon Data Firehose** đã được tạo và đang hoạt động.

Firehose phải được cấu hình để:

- Nhận dữ liệu theo lô.
- Ghi dữ liệu an toàn vào Amazon S3.
- Lưu dữ liệu tại tiền tố:

```text
raw/telemetry/
```

![Firehose stream của dự án đang ở trạng thái Active](/images/5-Workshop/5.4-Ingestion/firehose-stream-active.png)

### Bước 2: Cấu hình bảo mật AWS IoT Core

Truy cập:

```text
AWS IoT Core
-> Security
-> Policies
```

Tạo IoT Policy giới hạn quyền, chỉ cho phép:

```text
iot:Connect
iot:Publish
```

Quyền gửi dữ liệu phải được giới hạn đúng topic:

```text
telemetry/aqi/dev
```

#### Gắn thẻ cho tài nguyên

Áp dụng bộ thẻ thống nhất của dự án:

```text
Project: local-aqi-forecasting
Environment: dev
Owner: [Thành viên phụ trách]
Module: ingestion
```

Tiếp tục truy cập:

```text
AWS IoT Core
-> Security
-> Certificates
```

Tạo chứng chỉ mới và tải xuống:

- Root CA.
- Private Key.
- Device Certificate.

Sau khi tạo, gắn IoT Policy vào chứng chỉ tương ứng.

### Bước 3: Định tuyến bản tin từ IoT Rule sang Firehose

Truy cập:

```text
AWS IoT Core
-> Message routing
-> Rules
```

Tạo IoT Rule, ví dụ:

```text
Route_To_Firehose
```

#### Câu lệnh SQL

Sử dụng câu lệnh sau để nhận toàn bộ telemetry gửi vào topic:

```sql
SELECT * FROM 'telemetry/aqi/dev'
```

#### Cấu hình hành động của Rule

Chọn hành động:

```text
Send a message to a Data Firehose stream
```

Sau đó chọn delivery stream đã tạo ở bước trước.

#### Cấu hình IAM Role

Chọn IAM Role hiện có hoặc yêu cầu quản trị viên tạo role:

```text
local-aqi-dev-iot-to-firehose
```

Role này phải cho phép AWS IoT Core thực hiện:

```text
firehose:PutRecord
```

trên Firehose delivery stream của dự án.

#### Quan hệ tin cậy

IAM Role phải có Trust Policy cho phép service principal sau đảm nhận role:

```text
iot.amazonaws.com
```

Ví dụ:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "iot.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

### Bước 4: Chuẩn bị môi trường làm việc cục bộ

Tải mã nguồn dự án về máy:

```bash
git clone <repository-url>
```

Đặt các chứng chỉ AWS đã tải xuống vào thư mục:

```text
certs/
```

Thêm thư mục này vào `.gitignore` để tránh làm lộ thông tin xác thực:

```gitignore
certs/
*.pem
*.key
*.crt
```

Đặt bộ dữ liệu lịch sử đã làm sạch vào:

```text
data/
```

Tạo môi trường Python riêng:

```bash
python -m venv .venv
```

Kích hoạt trên Windows:

```bash
.venv\Scripts\activate
```

Kích hoạt trên Linux hoặc WSL:

```bash
source .venv/bin/activate
```

Cài đặt các thư viện cần thiết:

```bash
pip install paho-mqtt pandas
```

### Bước 5: Chạy Simulator và kiểm tra hệ thống

Chạy chương trình mô phỏng để bắt đầu gửi dữ liệu:

```bash
python simulator.py
```

![Simulator gửi dữ liệu từ nhiều trạm](/images/5-Workshop/5.4-Ingestion/simulator-records-sent.png)

![Cấu trúc bản tin nhận được qua AWS IoT Core](/images/5-Workshop/5.4-Ingestion/message-schema.png)

#### Kiểm tra số liệu giám sát

Truy cập:

```text
AWS Console
-> Amazon Data Firehose
-> Chọn delivery stream
-> Monitoring
```

Hoặc:

```text
AWS Console
-> CloudWatch
-> Metrics
-> Firehose
```

Kiểm tra hai metric:

```text
IncomingRecords
DeliveryToS3.Success
```

Chọn phép thống kê `Sum`. Các metric phải ghi nhận dữ liệu đang đi vào hệ thống và được chuyển thành công tới S3.

![Metric IncomingRecords của Firehose](/images/5-Workshop/5.4-Ingestion/firehose-incoming-records.png)

![Metric xác nhận Firehose chuyển dữ liệu thành công tới S3](/images/5-Workshop/5.4-Ingestion/firehose-delivery-metrics.png)

#### Kiểm tra dữ liệu trên S3

Mở bucket đích và kiểm tra Firehose đã ghi các đối tượng JSON theo lô vào cấu trúc phân vùng:

```text
raw/
└── year=2026/
    └── month=07/
        └── day=31/
            └── hour=10/
```

Firehose có thể chờ đủ kích thước bộ đệm hoặc thời gian bộ đệm trước khi ghi dữ liệu. Vì vậy, đối tượng có thể chưa xuất hiện ngay sau khi Simulator gửi bản tin.

Kết quả kiểm thử thực tế:

```text
Firehose nhận 37 bản ghi và chuyển thành công một đối tượng đã gom lô vào Amazon S3.
```

#### Kết quả cần chứng minh

- Gửi thành công tới đúng MQTT topic.
- IoT Rule định tuyến bản ghi sang Firehose.
- Firehose nhận dữ liệu và ghi đối tượng vào S3 Raw.
- Payload trong S3 khớp với đầu ra của Simulator.
- Hệ thống xử lý được nhiều bản ghi, không chỉ một bản tin đơn lẻ.

## Kết quả đạt được

Sau khi hoàn thành, nhóm đạt cột mốc đầu vào quan trọng nhất của MVP:

```text
Simulator
-> AWS IoT Core
-> Firehose
-> S3 Raw
```

Luồng này tạo dữ liệu thô thực tế để các nhóm Xử lý dữ liệu, Học máy và Backend tiếp tục triển khai.
