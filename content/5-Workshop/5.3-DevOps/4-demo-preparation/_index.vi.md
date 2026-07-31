---
title: "Chuẩn bị Trình diễn"
date: 2026-07-31
weight: 4
chapter: false
pre: "<b>4. </b>"
---

## 1. Mục tiêu

Chuẩn bị một kịch bản trình diễn ngắn gọn nhưng chứng minh được hệ thống hoạt động xuyên suốt từ dữ liệu đầu vào đến dự báo và cảnh báo.

## 2. Nguyên tắc trình bày

Mỗi phần trình diễn phải có:

```text
Đầu vào
Thao tác
Đầu ra
Minh chứng
```

Không chỉ mở giao diện AWS và mô tả cấu hình.

## 3. Thứ tự trình diễn

1. Giới thiệu bài toán và phạm vi MVP.
2. Trình bày sơ đồ kiến trúc.
3. Chạy Simulator với nhiều trạm.
4. Xác minh bản tin trong MQTT Test Client.
5. Kiểm tra Firehose và S3 Raw.
6. Chạy xử lý dữ liệu và tạo tệp Parquet.
7. Trình bày SageMaker Training và kết quả dự báo.
8. Kiểm tra API backend.
9. Kích hoạt cảnh báo SNS.
10. Trình bày CloudWatch và AWS Budgets.

## 4. Phân công trình bày

```text
Kiến trúc và DevOps        → M5
Simulator và IoT Core      → Kỹ sư thu nhận dữ liệu
S3 và đường ống dữ liệu    → Kỹ sư dữ liệu
SageMaker và dự báo        → Kỹ sư học máy
Backend và SNS             → Kỹ sư backend
Kết luận                   → Trưởng nhóm
```

![Bảng phân công và trạng thái công việc trên GitHub Project](/images/5-Workshop/5.3-DevOps/github-project-board.png)

## 5. Danh sách kiểm tra trước khi quay

- [ ] Đăng nhập đúng tài khoản AWS.
- [ ] Region là `ap-southeast-1`.
- [ ] Simulator chạy thành công.
- [ ] MQTT Test Client đăng ký đúng topic.
- [ ] Firehose ở trạng thái `Active`.
- [ ] S3 Raw có đối tượng mới.
- [ ] Xử lý dữ liệu chạy thành công.
- [ ] S3 Processed có tệp Parquet.
- [ ] SageMaker Training Job ở trạng thái `Completed`.
- [ ] Kết quả dự báo đã sẵn sàng.
- [ ] API backend hoạt động.
- [ ] Đăng ký SNS đã được xác nhận.
- [ ] Email cảnh báo đã được kiểm tra.
- [ ] CloudWatch có log.
- [ ] AWS Budgets truy cập được.
- [ ] Không để lộ thông tin xác thực.

## 6. Phương án dự phòng

Chuẩn bị ảnh Training Job ở trạng thái `Completed`, tệp dự báo JSON hoặc CSV, video ngắn của luồng thu nhận dữ liệu, đối tượng S3 mẫu, phản hồi API mẫu, email SNS đã nhận, sơ đồ kiến trúc ngoại tuyến và tệp log dùng làm minh chứng.

Khi dùng minh chứng dự phòng cần nói rõ đó là kết quả của lần kiểm thử trước.

## 7. Minh chứng bắt buộc

```text
Sơ đồ kiến trúc
GitHub Project Board
Đầu ra Simulator
MQTT Test Client
Firehose Monitoring
Đối tượng S3 Raw
Tệp Parquet trong S3 Processed
SageMaker Training Job
Kết quả dự báo
Phản hồi API
Email SNS
CloudWatch Logs
AWS Budgets
```

![Sơ đồ kiến trúc tổng thể dùng cho luồng trình diễn](/images/5-Workshop/5.1-Workshop-overview/5.3-devops-local-aqi-final-architecture.png)

## 8. Kết quả mong đợi

- Dữ liệu được gửi từ nhiều trạm.
- IoT Core nhận được bản tin.
- Firehose ghi dữ liệu xuống S3.
- Pipeline tạo dữ liệu đã xử lý.
- Mô hình tạo kết quả dự báo.
- Backend cung cấp kết quả qua API.
- SNS gửi cảnh báo.
- Hệ thống có giám sát và kiểm soát chi phí.
