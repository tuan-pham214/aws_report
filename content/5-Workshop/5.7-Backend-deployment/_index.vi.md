---
title : "Triển khai dịch vụ FastAPI"
weight : 7
chapter : false
pre : " <b> 5.7. </b> "
---

#### Tổng quan

Trong phần này, bạn sẽ triển khai dịch vụ backend của **Local AQI Forecasting & Alert System** lên Amazon EC2.

Dịch vụ được xây dựng bằng FastAPI và thực hiện các chức năng:

- Cung cấp API kiểm tra trạng thái hệ thống.
- Nhận yêu cầu đăng ký ngưỡng cảnh báo AQI.
- Lưu thông tin đăng ký và trạng thái cooldown trong Amazon DynamoDB.
- Gọi SageMaker Endpoint để lấy kết quả dự báo.
- Gửi email cảnh báo thông qua Amazon SNS.

Luồng hoạt động của Backend:

```text
Người dùng
    → FastAPI trên Amazon EC2
        → Amazon DynamoDB
        → Amazon SageMaker Endpoint
        → Amazon SNS
```

Backend sử dụng IAM role của EC2 để truy cập các dịch vụ AWS. Ứng dụng được chạy bằng `systemd` để tự khởi động lại khi EC2 được bật hoặc khi tiến trình gặp lỗi.

<!-- Bổ sung ảnh: Sơ đồ kiến trúc triển khai Backend -->

#### Nội dung

- [Chuẩn bị EC2 và IAM role](5.7.1-prepare/)
- [Cài đặt và cấu hình Backend](5.7.2-install/)
- [Khởi chạy Backend bằng systemd](5.7.3-systemd/)
- [Kiểm tra API và chu kỳ cảnh báo](5.7.4-test/)
