---
title: "Nhật ký công việc"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

Trong kỳ thực tập, em tham gia xây dựng **Hệ thống Dự báo và Cảnh báo Ô nhiễm Không khí Cục bộ (Local AQI Forecasting & Alert System)** với vai trò chính là Backend/API Engineer. Lộ trình bắt đầu từ kiến thức nền tảng AWS, phân tích kiến trúc và yêu cầu, sau đó chuyển sang xây dựng API, tích hợp dự báo, gửi cảnh báo và hoàn thiện khả năng vận hành.

Thời gian thực tập của em kéo dài từ ngày **15/06/2025 đến 15/08/2025**. Ngày 15/06/2025 là Chủ nhật nên bảng công việc bắt đầu từ Thứ 2, ngày 16/06/2025. Do báo cáo thực tập phải hoàn thành trước ngày 31/07/2025, phần nhật ký chi tiết dưới đây ghi nhận công việc đến hết ngày **30/07/2025**; khoảng thời gian từ 31/07/2025 đến 15/08/2025 vẫn thuộc kỳ thực tập nhưng không đưa vào phần nhật ký dùng để nộp báo cáo.

Luồng hệ thống được nhóm triển khai theo chuỗi: dữ liệu OpenAQ hoặc Python telemetry simulator được gửi qua MQTT tới AWS IoT Core (hoặc Mosquitto trên EC2), chuyển tiếp bằng Kinesis Data Firehose vào vùng `raw` và `processed` trên Amazon S3, xử lý và huấn luyện dự báo bằng SageMaker Processing/DeepAR, phục vụ dự báo qua SageMaker Endpoint, cung cấp API bằng FastAPI trên EC2 và gửi cảnh báo qua Amazon SNS. Các phần IAM, VPC/Security Group, CloudWatch, CloudTrail và AWS Budgets được áp dụng xuyên suốt quá trình triển khai.

Nội dung công việc theo từng tuần:

1. [Tuần 1 - Xây dựng nền tảng AWS](1.1-week1/)
2. [Tuần 2 - Tìm hiểu dịch vụ và kiến trúc áp dụng cho hệ thống AQI](1.2-week2/)
3. [Tuần 3 - Phân tích yêu cầu và xây dựng luồng thu thập dữ liệu](1.3-week3/)
4. [Tuần 4 - Tổ chức Data Lake và khởi tạo backend FastAPI](1.4-week4/)
5. [Tuần 5 - Xử lý dữ liệu, baseline model và chức năng đăng ký cảnh báo](1.5-week5/)
6. [Tuần 6 - Tích hợp dự báo với FastAPI và Amazon SNS](1.6-week6/)
7. [Tuần 7 - Kiểm thử, giám sát, bảo mật và tối ưu chi phí](1.7-week7/)
8. [Tuần 8 - Hoàn thiện tài liệu, demo và báo cáo](1.8-week8/)
