---
title: "Hướng dẫn thực hành"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

{{% notice info %}}
Đây là **dự án kỹ thuật chính** của báo cáo. Nội dung được tổ chức theo vai trò của từng thành viên để phản ánh đúng cách phân chia công việc thực tế và luồng demo end-to-end cuối cùng.
{{% /notice %}}

#### Tổng quan

Phần thực hành trình bày **một luồng đầu cuối hoàn chỉnh** thay vì mô tả riêng lẻ từng dịch vụ AWS. Mỗi vai trò giải thích:

+ các bước đã triển khai,
+ thành phần đã xây dựng,
+ và kết quả đạt được trong toàn bộ hệ thống.

Luồng demo chính:

```text
Simulator
-> AWS IoT Core
-> IoT Rule
-> Kinesis Data Firehose
-> S3 Raw
-> Data Processing
-> S3 Processed
-> SageMaker Training / Forecast Result
-> FastAPI
-> SNS Email
```

#### Phân chia theo vai trò

+ `5.3 DevOps`: xác định bài toán, kiến trúc, quy tắc đặt tên/gắn thẻ, thiết lập tài khoản IAM và kiểm soát truy cập.
+ `5.4 Ingestion`: gửi dữ liệu mô phỏng qua IoT Core, Firehose và lưu vào S3 Raw.
+ `5.5 Data Preparation`: đọc dữ liệu thô, làm sạch và tạo bộ dữ liệu sẵn sàng cho Machine Learning.
+ `5.6 Machine Learning`: huấn luyện mô hình dự báo PM2.5 và tạo kết quả dự báo.
+ `5.7 Backend`: cung cấp API và gửi cảnh báo qua SNS.

#### Nội dung

1. [Tổng quan phần thực hành](5.1-Workshop-overview/)
2. [Điều kiện tiên quyết](5.2-Prerequiste/)
3. [DevOps: kiến trúc, IAM và quản trị tài khoản](5.3-DevOps/)
4. [Thu nhận dữ liệu: Simulator -> IoT Core -> Firehose -> S3 Raw](5.4-Ingestion/)
5. [Chuẩn bị dữ liệu: xử lý và chuẩn hóa](5.5-Data%20Preprocessing/)
6. [Học máy: huấn luyện và tạo kết quả dự báo](5.6-Machine%20learning/)
7. [Triển khai dịch vụ FastAPI](5.7-Backend-deployment/)
8. [Chính sách và kiểm soát truy cập](5.8-Policy/)
9. [Dọn dẹp tài nguyên](5.9-Cleanup/)
