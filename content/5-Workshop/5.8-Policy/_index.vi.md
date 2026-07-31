---
title: "Chính sách truy cập và phân quyền IAM"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Tổng quan

Trong dự án **Local AQI Forecasting & Alert System**, nhiều dịch vụ AWS cần trao đổi dữ liệu với nhau, bao gồm:

* AWS IoT Core nhận dữ liệu từ thiết bị mô phỏng.
* Amazon Data Firehose chuyển dữ liệu vào Amazon S3.
* Amazon SageMaker đọc dữ liệu, xử lý dữ liệu và huấn luyện mô hình.
* Amazon EC2 chạy API phục vụ kết quả dự báo.
* Amazon SNS gửi cảnh báo chất lượng không khí qua email.

Để các thành phần này hoạt động an toàn, dự án sử dụng **AWS Identity and Access Management (IAM)** để kiểm soát người dùng, vai trò và quyền truy cập.

Nguyên tắc chính được áp dụng là **Least Privilege**, nghĩa là mỗi người dùng hoặc dịch vụ chỉ được cấp những quyền cần thiết để hoàn thành nhiệm vụ của mình.

> **Lưu ý:** Không chia sẻ tài khoản root cho thành viên nhóm. Tài khoản root chỉ được sử dụng cho các thao tác quản trị đặc biệt như quản lý thanh toán, khôi phục tài khoản hoặc cấu hình bảo mật cấp cao.

---

## 1. Quản lý tài khoản người dùng IAM

Mỗi thành viên làm việc trên AWS được cấp một IAM User riêng thay vì sử dụng chung tài khoản root.

Việc tách riêng IAM User giúp:

* Xác định thành viên thực hiện từng thao tác.
* Giảm nguy cơ lộ thông tin đăng nhập tài khoản root.
* Có thể khóa hoặc thu hồi quyền của từng thành viên khi cần.
* Kiểm soát phạm vi dịch vụ mà mỗi thành viên được sử dụng.
* Theo dõi hoạt động thông qua AWS CloudTrail nếu được cấu hình.

Tài khoản quản trị dự án được bật **Multi-Factor Authentication – MFA** để tăng mức độ bảo mật.

---

## 2. Phân quyền theo vai trò trong nhóm

Các thành viên được cấp quyền dựa trên nhiệm vụ phụ trách trong dự án.

### Quản trị và DevOps

Người phụ trách quản trị AWS thực hiện các công việc:

* Tạo và quản lý IAM User.
* Tạo IAM Role cho các dịch vụ.
* Cấu hình AWS Budgets.
* Theo dõi tài nguyên đang hoạt động.
* Hỗ trợ các thành viên khi gặp lỗi phân quyền.
* Thu hồi quyền tạm thời sau khi hoàn thành công việc.

### Kỹ thuật dữ liệu

Thành viên phụ trách dữ liệu cần quyền làm việc với:

* AWS IoT Core.
* Amazon Data Firehose.
* Amazon S3.
* Amazon CloudWatch Logs khi kiểm tra lỗi luồng ingestion.

### Học máy

Thành viên phụ trách Machine Learning cần quyền làm việc với:

* Amazon SageMaker.
* Amazon S3 Raw.
* Amazon S3 Processed.
* Các job xử lý, huấn luyện và triển khai mô hình.

### Dịch vụ backend

Thành viên phụ trách backend cần quyền làm việc với:

* Amazon EC2.
* Amazon SNS.
* Amazon S3 hoặc SageMaker Endpoint tùy theo cách backend lấy kết quả dự báo.
* CloudWatch Logs để kiểm tra lỗi ứng dụng.

Việc phân quyền theo nhiệm vụ giúp hạn chế trường hợp một tài khoản có thể thay đổi những tài nguyên không thuộc phạm vi phụ trách.

---

## 3. Chính sách AWS IoT cho thiết bị mô phỏng

Thiết bị mô phỏng gửi dữ liệu chất lượng không khí đến AWS IoT Core thông qua giao thức MQTT.

AWS IoT Policy được gắn với certificate của thiết bị mô phỏng để cho phép thiết bị:

* Kết nối đến AWS IoT Core.
* Publish dữ liệu lên topic được chỉ định.
* Không được publish tùy ý lên các topic khác trong hệ thống.

Topic được sử dụng trong môi trường phát triển:

```text
telemetry/aqi/dev
```

Ví dụ AWS IoT Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:ap-southeast-1:<AWS_ACCOUNT_ID>:client/${iot:Connection.Thing.ThingName}"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": "arn:aws:iot:ap-southeast-1:<AWS_ACCOUNT_ID>:topic/telemetry/aqi/dev"
    }
  ]
}
```

Trong policy thực tế, `<AWS_ACCOUNT_ID>` được thay bằng AWS Account ID của dự án.

Policy không sử dụng `"Resource": "*"` cho quyền publish nhằm tránh việc thiết bị gửi dữ liệu đến các topic ngoài phạm vi cho phép.

Certificate của thiết bị được gắn với policy tương ứng.

---

## 4. Vai trò IAM cho AWS IoT Rule

AWS IoT Rule nhận dữ liệu từ MQTT topic và chuyển record đến Amazon Data Firehose.

Để thực hiện thao tác này, IoT Rule cần một IAM Role có trust relationship cho phép dịch vụ AWS IoT Core sử dụng role.

Ví dụ trust policy:

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

Role chỉ được cấp quyền gửi record vào delivery stream của dự án.

Ví dụ permission policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "firehose:PutRecord",
        "firehose:PutRecordBatch"
      ],
      "Resource": "arn:aws:firehose:ap-southeast-1:<AWS_ACCOUNT_ID>:deliverystream/<FIREHOSE_STREAM_NAME>"
    }
  ]
}
```

Policy giới hạn quyền trên đúng Firehose delivery stream thay vì cho phép truy cập tất cả delivery stream trong tài khoản.

---

## 5. Vai trò IAM cho Amazon Data Firehose

Amazon Data Firehose cần IAM Role để ghi dữ liệu telemetry vào S3 Raw Bucket.

Tên role được sử dụng theo quy ước của dự án:

```text
local-aqi-dev-firehose-to-s3
```

Trust policy cho phép dịch vụ Firehose sử dụng role:

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

Permission policy giới hạn quyền trên bucket dữ liệu thô:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::local-aqi-dev-s3-raw"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:AbortMultipartUpload",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::local-aqi-dev-s3-raw/raw/telemetry/*"
    }
  ]
}
```

Firehose chỉ được ghi dữ liệu vào prefix:

```text
raw/telemetry/
```

Việc giới hạn prefix giúp giảm rủi ro Firehose ghi đè hoặc tạo dữ liệu tại các khu vực khác trong bucket.

Trong trường hợp Firehose sử dụng CloudWatch Logs, role có thể được bổ sung các quyền cần thiết như:

```json
{
  "Effect": "Allow",
  "Action": [
    "logs:PutLogEvents"
  ],
  "Resource": "arn:aws:logs:ap-southeast-1:<AWS_ACCOUNT_ID>:log-group:<LOG_GROUP_NAME>:log-stream:*"
}
```

---

## 6. Vai trò thực thi SageMaker

Amazon SageMaker cần một execution role để đọc dữ liệu đầu vào, ghi dữ liệu sau xử lý và lưu kết quả huấn luyện.

Tên role của dự án:

```text
local-aqi-dev-sagemaker-execution-role
```

Trust policy cho phép Amazon SageMaker sử dụng role:

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

Role được cấp các quyền chính:

* Đọc dữ liệu từ S3 Raw Bucket.
* Đọc dữ liệu từ S3 Processed Bucket.
* Ghi và xóa dữ liệu trong S3 Processed Bucket.
* Thực hiện các thao tác SageMaker cần thiết cho Processing Job, Training Job hoặc Endpoint.

Ví dụ policy truy cập S3:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::local-aqi-dev-s3-raw",
        "arn:aws:s3:::local-aqi-dev-s3-processed"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::local-aqi-dev-s3-raw/*",
        "arn:aws:s3:::local-aqi-dev-s3-processed/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::local-aqi-dev-s3-processed/*"
    }
  ]
}
```

SageMaker không được cấp quyền ghi hoặc xóa dữ liệu trong S3 Raw Bucket. Điều này giúp bảo vệ dữ liệu gốc được thu thập từ pipeline ingestion.

---

## 7. Quyền truy cập Amazon SNS

Amazon SNS được sử dụng để gửi cảnh báo khi chỉ số chất lượng không khí vượt ngưỡng.

Ứng dụng backend chỉ cần quyền publish đến đúng SNS Topic của dự án.

Ví dụ policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:ap-southeast-1:<AWS_ACCOUNT_ID>:<SNS_TOPIC_NAME>"
    }
  ]
}
```

Policy không cấp các quyền quản trị như:

* `sns:DeleteTopic`
* `sns:SetTopicAttributes`
* `sns:Subscribe`
* `sns:Unsubscribe`

Nhờ đó, ứng dụng có thể gửi cảnh báo nhưng không thể tự thay đổi cấu hình của SNS Topic.

---

## 8. Quyền chuyển giao vai trò

Một số thành viên cần tạo hoặc cấu hình tài nguyên sử dụng IAM Role, ví dụ:

* Gắn role cho IoT Rule.
* Gắn role cho Firehose.
* Chọn execution role cho SageMaker.

Trong các trường hợp này, IAM User cần quyền `iam:PassRole`.

Quyền này chỉ nên áp dụng với các role thuộc dự án:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": [
        "arn:aws:iam::<AWS_ACCOUNT_ID>:role/local-aqi-dev-firehose-to-s3",
        "arn:aws:iam::<AWS_ACCOUNT_ID>:role/local-aqi-dev-sagemaker-execution-role",
        "arn:aws:iam::<AWS_ACCOUNT_ID>:role/local-aqi-dev-iot-to-firehose"
      ]
    }
  ]
}
```

Không nên cấp:

```json
{
  "Action": "iam:PassRole",
  "Resource": "*"
}
```

vì người dùng có thể gắn các role có quyền cao hơn vào dịch vụ mà họ kiểm soát.

Sau khi thành viên hoàn thành việc tạo tài nguyên, các quyền IAM tạm thời như `iam:CreateRole`, `iam:AttachRolePolicy` hoặc `iam:PassRole` có thể được thu hồi.

---

## 9. Quy ước đặt tên tài nguyên

Các IAM Role và policy được đặt tên theo cấu trúc:

```text
<project>-<environment>-<service>-<purpose>
```

Ví dụ:

```text
local-aqi-dev-firehose-to-s3
local-aqi-dev-sagemaker-execution-role
local-aqi-dev-iot-to-firehose
```

Quy ước này giúp:

* Nhận biết tài nguyên thuộc dự án nào.
* Phân biệt môi trường phát triển và môi trường production.
* Xác định dịch vụ sử dụng role.
* Dễ tìm kiếm và dọn dẹp tài nguyên.

---

## 10. Gắn thẻ cho tài nguyên

Các tài nguyên hỗ trợ tagging được gắn các tag chung:

| Tag         | Giá trị mẫu                              |
| ----------- | ---------------------------------------- |
| Project     | local-aqi-forecasting                    |
| Environment | dev                                      |
| Owner       | Tên thành viên phụ trách                 |
| Module      | ingestion, data, ml, backend hoặc devops |

Ví dụ:

```text
Project     = local-aqi-forecasting
Environment = dev
Owner       = Quynh-Tam
Module      = ingestion
```

Tag được sử dụng để:

* Theo dõi tài nguyên theo dự án.
* Xác định người phụ trách.
* Phân loại chi phí.
* Hỗ trợ việc kiểm tra và dọn dẹp tài nguyên.
* Tránh xóa nhầm tài nguyên của dự án khác.

---

## 11. Kiểm tra quyền truy cập

Sau khi cấu hình IAM Role và policy, nhóm kiểm tra quyền bằng luồng thực tế.

### Kiểm tra IoT Core đến Firehose

Thiết bị mô phỏng publish message lên topic:

```text
telemetry/aqi/dev
```

IoT Rule chuyển message sang Firehose.

Nếu IAM Role thiếu quyền, AWS IoT Rule sẽ không thể gọi:

```text
firehose:PutRecord
```

### Kiểm tra Firehose đến S3

Sau khi gửi dữ liệu, nhóm kiểm tra object trong bucket:

```bash
aws s3 ls s3://local-aqi-dev-s3-raw/raw/telemetry/ --recursive
```

Nếu role Firehose hoạt động đúng, object mới sẽ xuất hiện trong S3 Raw Bucket.

### Kiểm tra SageMaker đọc dữ liệu

Trong môi trường SageMaker, nhóm kiểm tra khả năng đọc dữ liệu:

```bash
aws s3 ls s3://local-aqi-dev-s3-raw/
```

Sau đó kiểm tra khả năng ghi dữ liệu đã xử lý:

```bash
aws s3 cp processed-data.csv \
s3://local-aqi-dev-s3-processed/processed/processed-data.csv
```

SageMaker không được phép xóa hoặc thay đổi dữ liệu gốc trong S3 Raw Bucket.

### Kiểm tra SNS

Backend hoặc AWS CLI gửi thử một cảnh báo:

```bash
aws sns publish \
  --topic-arn arn:aws:sns:ap-southeast-1:<AWS_ACCOUNT_ID>:<SNS_TOPIC_NAME> \
  --subject "Local AQI Test Alert" \
  --message "Đây là cảnh báo thử nghiệm của hệ thống."
```

Nếu policy hợp lệ và subscription đã được xác nhận, email cảnh báo sẽ được gửi đến người nhận.

---

## 12. Kết quả

Sau khi hoàn thành cấu hình IAM và policy:

* Thành viên không sử dụng chung tài khoản root.
* Tài khoản quản trị được bảo vệ bằng MFA.
* Thiết bị chỉ được publish lên MQTT topic của dự án.
* IoT Rule chỉ được gửi dữ liệu vào Firehose delivery stream được chỉ định.
* Firehose chỉ được ghi dữ liệu vào S3 Raw Bucket.
* SageMaker được đọc dữ liệu gốc nhưng không được xóa dữ liệu gốc.
* Backend chỉ được publish cảnh báo đến SNS Topic của dự án.
* Các quyền IAM tạm thời được thu hồi sau khi hoàn thành cấu hình.
* Tài nguyên được đặt tên và gắn tag theo quy ước chung.

Việc áp dụng nguyên tắc Least Privilege giúp hệ thống giảm nguy cơ truy cập trái phép, hạn chế ảnh hưởng khi thông tin đăng nhập bị lộ và hỗ trợ quản lý tài nguyên AWS rõ ràng hơn.
