---
title : "Chuẩn bị EC2 và vai trò IAM"
weight : 1
chapter : false
pre : " <b> 5.7.1 </b> "
---

#### Tạo vai trò IAM cho dịch vụ backend

1. Truy cập **AWS Management Console**.
2. Tìm và mở dịch vụ **IAM**.
3. Trong thanh điều hướng bên trái, chọn **Roles**.
4. Chọn **Create role**.
5. Tại **Trusted entity type**, chọn **AWS service**.
6. Tại **Use case**, chọn **EC2**.
7. Chọn **Next**.

<!-- Bổ sung ảnh: Màn hình chọn trusted entity EC2 -->

Gắn policy `AmazonSSMManagedInstanceCore` để có thể quản trị EC2 bằng AWS Systems Manager Session Manager.

Tiếp theo, tạo policy runtime cho Backend:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SubscriberTable",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:Scan"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:ACCOUNT_ID:table/local-aqi-subscribers-dev"
    },
    {
      "Sid": "AlertTopic",
      "Effect": "Allow",
      "Action": [
        "sns:GetSubscriptionAttributes",
        "sns:ListSubscriptionsByTopic",
        "sns:Publish",
        "sns:Subscribe"
      ],
      "Resource": "arn:aws:sns:ap-southeast-1:ACCOUNT_ID:local-aqi-alerts-dev"
    },
    {
      "Sid": "ForecastEndpoint",
      "Effect": "Allow",
      "Action": "sagemaker:InvokeEndpoint",
      "Resource": "arn:aws:sagemaker:ap-southeast-1:ACCOUNT_ID:endpoint/ENDPOINT_NAME"
    }
  ]
}
```

Thay `ACCOUNT_ID` và `ENDPOINT_NAME` bằng giá trị của môi trường triển khai.

Đặt tên role:

```text
local-aqi-backend-ec2-role
```

Chọn **Create role** để hoàn tất.

<!-- Bổ sung ảnh: IAM role sau khi tạo thành công -->

#### Tạo nhóm bảo mật

1. Mở dịch vụ **Amazon EC2**.
2. Trong thanh điều hướng bên trái, chọn **Security Groups**.
3. Chọn **Create security group**.
4. Nhập tên:

```text
local-aqi-backend-sg
```

5. Chọn VPC dùng cho dự án.
6. Tại **Inbound rules**, thêm rule:

| Type | Protocol | Port range | Source |
| --- | --- | --- | --- |
| Custom TCP | TCP | `8000` | My IP hoặc CIDR được phép demo |

Không mở cổng SSH `22`. Việc quản trị máy chủ được thực hiện qua Session Manager.

<!-- Bổ sung ảnh: Inbound rule TCP 8000 -->

#### Khởi tạo EC2

1. Trong giao diện **Amazon EC2**, chọn **Instances**.
2. Chọn **Launch instances**.
3. Nhập tên:

```text
local-aqi-dev-ec2-backend
```

4. Chọn Amazon Linux làm AMI.
5. Chọn instance type phù hợp với môi trường thử nghiệm, ví dụ `t3.micro`.
6. Tại **Key pair**, chọn **Proceed without a key pair**.
7. Chọn Security Group `local-aqi-backend-sg`.
8. Tại **Advanced details**, chọn IAM instance profile tương ứng với role `local-aqi-backend-ec2-role`.
9. Bật yêu cầu sử dụng **IMDSv2**.
10. Cấu hình EBS `gp3`, bật mã hóa và chọn xóa volume khi instance bị terminate.

Thêm các tag:

```text
Project=local-aqi-forecasting
Environment=dev
Owner=quang-tuan
Module=backend
```

Chọn **Launch instance**.

<!-- Bổ sung ảnh: EC2 ở trạng thái Running -->

Đợi instance chuyển sang trạng thái `Running` và xuất hiện trong Systems Manager trước khi chuyển sang phần tiếp theo.
