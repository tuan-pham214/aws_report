---
title: "Dọn dẹp tài nguyên"
date: 2026-07-31
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Mục tiêu

Xóa hoặc dừng các tài nguyên không còn sử dụng sau khi hoàn thành Workshop để tránh phát sinh chi phí ngoài dự kiến.

{{% notice warning %}}
Chỉ xóa tài nguyên sau khi đã sao lưu dữ liệu, kết quả dự báo, model artifact và toàn bộ ảnh minh chứng cần dùng cho báo cáo.
{{% /notice %}}

## 1. Xóa SageMaker Endpoint

Endpoint có thể tiếp tục phát sinh chi phí khi vẫn ở trạng thái `InService`. Xóa theo thứ tự:

1. SageMaker Endpoint.
2. Endpoint Configuration.
3. SageMaker Model không còn sử dụng.

```bash
aws sagemaker delete-endpoint --endpoint-name <ENDPOINT_NAME>
aws sagemaker delete-endpoint-config --endpoint-config-name <ENDPOINT_CONFIG_NAME>
aws sagemaker delete-model --model-name <MODEL_NAME>
```

## 2. Dừng EC2 và kiểm tra tài nguyên mạng

- Dừng hoặc chấm dứt EC2 backend.
- Giải phóng Elastic IP không còn gắn với tài nguyên.
- Kiểm tra Security Group, NAT Gateway và VPC Endpoint.
- Chỉ xóa VPC khi không còn tài nguyên phụ thuộc.

![Kiểm tra và xóa VPC không còn sử dụng](/images/5-Workshop/5.6-Cleanup/vpc.png)

## 3. Xóa Firehose, IoT Rule và chứng chỉ

1. Xóa IoT Rule định tuyến dữ liệu sang Firehose.
2. Vô hiệu hóa rồi xóa IoT Certificate.
3. Xóa IoT Policy không còn được gắn.
4. Xóa Firehose delivery stream.

Kiểm tra kỹ tên và Region trước khi xóa để không ảnh hưởng tài nguyên của nhóm khác.

## 4. Làm rỗng và xóa S3 Bucket

Sao lưu dữ liệu cần thiết trước khi thực hiện:

```bash
aws s3 sync s3://local-aqi-dev-s3-raw ./backup/raw
aws s3 sync s3://local-aqi-dev-s3-processed ./backup/processed
```

Sau đó xóa toàn bộ đối tượng và xóa bucket:

```bash
aws s3 rm s3://local-aqi-dev-s3-raw --recursive
aws s3 rb s3://local-aqi-dev-s3-raw
```

![Xóa dữ liệu trong S3 Bucket](/images/5-Workshop/5.6-Cleanup/delete-s3.png)

## 5. Xóa CloudFormation Stack

Nếu tài nguyên được tạo bằng CloudFormation, xóa stack sau khi đã xử lý các bucket hoặc tài nguyên giữ dữ liệu.

![Xóa CloudFormation Stack](/images/5-Workshop/5.6-Cleanup/delete-stack.png)

## 6. Xóa Hosted Zone không còn sử dụng

Kiểm tra toàn bộ record trước khi xóa Route 53 Hosted Zone.

![Xóa Route 53 Hosted Zone](/images/5-Workshop/5.6-Cleanup/delete-zone.png)

## 7. Xóa SNS và tài nguyên giám sát

- Xóa SNS Subscription và SNS Topic của dự án.
- Xóa CloudWatch Log Group không còn cần lưu.
- Xóa alarm và dashboard thử nghiệm.
- Kiểm tra thời gian lưu log để tránh chi phí lâu dài.

## 8. Xóa IAM Role và Policy

Chỉ xóa role sau khi các dịch vụ đang sử dụng role đã được xóa. Gỡ inline policy, managed policy và instance profile trước khi xóa role.

## 9. Kiểm tra cuối cùng

- [ ] SageMaker Endpoint đã được xóa.
- [ ] EC2 đã dừng hoặc chấm dứt.
- [ ] Firehose và IoT Rule đã được xóa.
- [ ] S3 đã được sao lưu và xóa theo kế hoạch.
- [ ] SNS, CloudWatch và IAM đã được rà soát.
- [ ] Không còn Elastic IP, NAT Gateway hoặc tài nguyên tính phí ngoài dự kiến.
- [ ] AWS Budgets không ghi nhận chi phí bất thường.

Sau bước này, môi trường Workshop không còn tài nguyên chạy liên tục gây phát sinh chi phí.
