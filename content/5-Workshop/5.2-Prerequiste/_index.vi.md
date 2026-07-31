---

title: "Điều kiện tiên quyết"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
----------------------

#### Chuẩn bị tài khoản và quyền truy cập AWS

Trước khi triển khai hệ thống, nhóm cần chuẩn bị một AWS account có khả năng sử dụng các dịch vụ chính của dự án.

Toàn bộ tài nguyên được triển khai thống nhất tại Region:

```text
Asia Pacific (Singapore)
ap-southeast-1
```

Việc sử dụng chung một Region giúp nhóm:

* Tránh tạo nhầm tài nguyên ở nhiều Region.
* Dễ quản lý chi phí và trạng thái hoạt động.
* Giảm lỗi khi liên kết IoT Core, Firehose, S3 và SageMaker.
* Thống nhất cấu hình giữa các thành viên.

#### Người dùng IAM và nguyên tắc phân quyền

Không sử dụng hoặc chia sẻ tài khoản Root cho các thành viên trong nhóm.

Mỗi thành viên sử dụng IAM user riêng để:

* Đăng nhập AWS Management Console.
* Thao tác với các service được phân công.
* Tạo và kiểm tra resource thuộc module phụ trách.
* Hạn chế ảnh hưởng đến tài nguyên của thành viên khác.

Các quyền IAM được cấp theo vai trò và phạm vi công việc, thay vì cấp toàn quyền cho mỗi thành viên.

Một số nhóm quyền cần thiết trong quá trình triển khai gồm:

```text
AWS IoT Core
Amazon Data Firehose
Amazon S3
Amazon SageMaker AI
Amazon CloudWatch
Amazon SNS
IAM PassRole
Billing read-only nếu cần kiểm tra chi phí
```

Các quyền liên quan đến IAM role chỉ được cấp trong thời gian cần thiết. Sau khi thành viên hoàn tất việc tạo role hoặc cấu hình service, quyền tạm thời cần được thu hồi.

#### Các vai trò IAM của dự án

Dự án sử dụng các IAM role riêng cho từng service.

| IAM role                                 | Trusted service      | Mục đích                                                                        |
| ---------------------------------------- | -------------------- | ------------------------------------------------------------------------------- |
| `local-aqi-dev-iot-to-firehose`          | AWS IoT Core         | Cho phép IoT Rule gửi record sang Firehose                                      |
| `local-aqi-dev-firehose-to-s3`           | Amazon Data Firehose | Cho phép Firehose ghi dữ liệu vào S3 Raw và ghi error log                       |
| `local-aqi-dev-sagemaker-execution-role` | Amazon SageMaker AI  | Cho phép SageMaker đọc dữ liệu, chạy Processing, Training và lưu model artifact |

Trust policy của mỗi role phải xác định đúng service được phép assume role.

Ví dụ, role dùng cho AWS IoT Core cần trust:

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

Role dùng cho Amazon Data Firehose cần trust:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Role dùng cho Amazon SageMaker AI cần có service principal:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "sagemaker.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Trong giai đoạn phát triển, role SageMaker có thể được cấp thêm trusted services để hỗ trợ quá trình thử nghiệm. Sau khi hệ thống hoàn tất, trust policy và permission policy cần được thu gọn theo nguyên tắc least privilege.

#### Chuẩn bị các bucket S3

Trước khi triển khai pipeline, cần tạo hai bucket chính:

```text
local-aqi-dev-s3-raw
local-aqi-dev-s3-processed
```

Trong đó:

* `local-aqi-dev-s3-raw` lưu dữ liệu telemetry được Firehose ghi trực tiếp.
* `local-aqi-dev-s3-processed` lưu dữ liệu sau khi làm sạch, chuyển đổi và chuẩn bị cho Machine Learning.

Dữ liệu Raw được tổ chức theo cấu trúc phân vùng thời gian:

```text
raw/
  year=YYYY/
    month=MM/
      day=DD/
        hour=HH/
```

Cấu trúc này giúp:

* Dễ truy xuất dữ liệu theo thời gian.
* Hạn chế phải quét toàn bộ bucket.
* Thuận tiện cho bước xử lý dữ liệu.
* Dễ kiểm tra dữ liệu mới được ingest.

#### Chuẩn bị luồng thu nhận dữ liệu

Pipeline ingestion của dự án sử dụng các thành phần:

```text
MQTT Simulator
-> AWS IoT Core
-> IoT Rule
-> Amazon Data Firehose
-> Amazon S3 Raw
```

Các resource chính cần có:

```text
IoT Thing Group:
local-aqi-dev-iot-group

IoT Policy:
local-aqi-dev-iot-policy

IoT Rule:
Rule chuyển dữ liệu từ MQTT topic sang Firehose

Firehose delivery stream:
local-aqi-dev-firehose-telemetry
```

MQTT message cần tuân theo telemetry schema thống nhất.

Ví dụ:

```json
{
  "schema_version": "1.0",
  "station_id": "station_001",
  "ts_utc": "2026-07-31T08:00:00Z",
  "pm25_ugm3": 42.5,
  "pm10_ugm3": 61.2,
  "temperature_c": 30.5,
  "humidity_pct": 72,
  "source": "simulator"
}
```

Các field bắt buộc gồm:

```text
schema_version
station_id
ts_utc
pm25_ugm3
pm10_ugm3
temperature_c
humidity_pct
source
```

#### Chuẩn bị môi trường học máy

Mô hình dự báo được huấn luyện bằng Amazon SageMaker AI.

Dữ liệu sử dụng để training gồm:

* Dữ liệu lịch sử PM2.5 đã được thu thập và xử lý.
* Dữ liệu được chuẩn hóa trong S3 Processed.
* Các chuỗi thời gian được phân chia theo station và timestamp.

Dữ liệu realtime từ MQTT Simulator chủ yếu dùng để kiểm thử pipeline ingestion và bổ sung dữ liệu mới trong quá trình vận hành. Nhóm không cần chờ simulator thu thập đủ dữ liệu mới bắt đầu training.

Luồng Machine Learning dự kiến:

```text
S3 Raw
-> Data Processing
-> S3 Processed
-> SageMaker Training Job
-> Model Artifact
-> SageMaker Endpoint
```

Execution role của SageMaker cần tối thiểu các quyền:

```text
s3:ListBucket
s3:GetObject
s3:PutObject
s3:DeleteObject nếu pipeline cần ghi đè dữ liệu processed
```

Quyền phải được giới hạn vào đúng hai bucket của dự án thay vì cấp `s3:*` trên toàn bộ account.

#### Chuẩn bị giám sát và cảnh báo

Các dịch vụ cần được theo dõi bằng Amazon CloudWatch:

```text
Amazon Data Firehose
SageMaker Training Jobs
SageMaker Endpoint
Backend API
SNS publish result
```

Firehose cần bật:

```text
Destination error logs
Amazon CloudWatch error logging: Enabled
```

Các log group dự kiến gồm:

```text
/aws/kinesisfirehose/local-aqi-dev-firehose-telemetry
/aws/sagemaker/TrainingJobs
/aws/sagemaker/Endpoints/aqi-endpoint-test
```

Amazon SNS được sử dụng để gửi email khi giá trị PM2.5 hoặc kết quả forecast vượt ngưỡng cảnh báo.

Email subscription phải có trạng thái:

```text
Confirmed
```

trước khi thực hiện kiểm thử gửi thông báo.

#### Kiểm tra điều kiện trước khi triển khai

Trước khi chuyển sang các bước triển khai tiếp theo, cần xác nhận:

```text
[ ] Đang sử dụng Region ap-southeast-1
[ ] IAM user đăng nhập được AWS Console
[ ] IAM role cho IoT Core đã được tạo
[ ] IAM role cho Firehose đã được tạo
[ ] IAM role cho SageMaker đã được tạo
[ ] Bucket S3 Raw đã được tạo
[ ] Bucket S3 Processed đã được tạo
[ ] Firehose delivery stream đã được tạo
[ ] CloudWatch error logging đã được bật
[ ] SageMaker có thể truy cập dữ liệu trong S3
[ ] SNS email subscription đã được confirmed
[ ] AWS Budget đã được thiết lập
```

Sau khi hoàn tất các điều kiện trên, nhóm có thể tiếp tục triển khai:

```text
IoT ingestion
Data processing
Machine Learning
Backend API
SNS Alert
Integration Testing
```
