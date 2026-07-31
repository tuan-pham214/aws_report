---
title: "Giám sát và Đảm bảo Chất lượng"
date: 2026-07-31
weight: 3
chapter: false
pre: "<b>3. </b>"
---

## 1. Mục tiêu

Theo dõi trạng thái hoạt động của các dịch vụ, thu thập log cần thiết và kiểm thử toàn bộ luồng trước khi nghiệm thu.

## 2. Giám sát và ghi log

Các thành phần cần theo dõi:

- AWS IoT Core và IoT Rule.
- Amazon Data Firehose.
- Amazon S3 Raw và S3 Processed.
- SageMaker Processing hoặc Training Job.
- API backend.
- Amazon SNS.

Thông tin cần kiểm tra:

```text
Số bản ghi đầu vào
Số lần chuyển dữ liệu thành công
Số lần chuyển dữ liệu thất bại
Độ mới của dữ liệu
Trạng thái công việc huấn luyện
Lỗi API
Kết quả gửi thông báo SNS
```

Khi báo lỗi, thành viên phải cung cấp thời điểm, tên tài nguyên, thông báo lỗi, vị trí log và thao tác đang thực hiện.

![Metric Firehose xác nhận dữ liệu được chuyển thành công tới S3](/images/5-Workshop/5.4-Ingestion/firehose-delivery-metrics.png)

![CloudWatch Logs của SageMaker Training Job](/images/5-Workshop/5.3-DevOps/cloudwatch-log-events.png)

## 3. Kiểm thử từng mô-đun

### Thu nhận dữ liệu

- Gửi một bản tin và nhiều bản tin.
- Gửi dữ liệu từ nhiều trạm.
- Kiểm tra bản tin thiếu trường.
- Kiểm tra trường hợp gửi thất bại.

### Xử lý dữ liệu

- Đọc JSON từ S3 Raw.
- Xử lý nhiều JSON nối tiếp.
- Kiểm tra giá trị rỗng và bản ghi trùng.
- Kiểm tra giá trị âm và dấu thời gian UTC.
- Ghi và đọc lại tệp Parquet.

### Học máy

- Đọc bộ dữ liệu đã xử lý.
- Chạy huấn luyện.
- Kiểm tra tệp mô hình.
- Tạo dự báo 24 giờ.
- Ghi nhận MAE và RMSE.
- Kiểm tra trạng thái Training Job.

### Dịch vụ backend

- Kiểm tra trạng thái hoạt động.
- Dự báo với mã trạm hợp lệ.
- Xử lý trạm không tồn tại.
- Xử lý endpoint chưa sẵn sàng.
- Kiểm tra gửi SNS thành công.
- Xác nhận đăng ký email.

## 4. Kiểm thử tích hợp

Luồng nghiệm thu:

```text
Simulator
→ AWS IoT Core
→ IoT Rule
→ Firehose
→ S3 Raw
→ Data Processing
→ S3 Processed
→ ML Forecast
→ Backend
→ SNS Email
```

Kịch bản bình thường kiểm tra dữ liệu từ nhiều trạm đi qua toàn bộ pipeline và tạo được dự báo. Kịch bản vượt ngưỡng kiểm tra backend kích hoạt SNS. Kịch bản lỗi kiểm tra payload thiếu trường, sai kiểu dữ liệu, bản ghi trùng, mã trạm không hợp lệ và API nhận trạm không tồn tại.

![Minh chứng thu nhận dữ liệu: đối tượng thực tế đã được ghi vào S3 Raw](/images/5-Workshop/5.3-DevOps/ingestion-evidence.png)

![Kết quả đánh giá mô hình DeepAR trên SageMaker](/images/5-Workshop/5.6-Machine-learning/deepar_sagemaker_evaluation.png)

## 5. Biểu mẫu kết quả

```text
Ca kiểm thử:
Đầu vào:
Kết quả mong đợi:
Kết quả thực tế:
Trạng thái: Đạt / Không đạt
Minh chứng:
Người phụ trách:
```

## 6. Tiêu chí nghiệm thu

- Simulator gửi được dữ liệu.
- IoT Core nhận được bản tin.
- Firehose ghi được dữ liệu vào S3 Raw.
- Dữ liệu đã xử lý đọc được bằng Parquet.
- Mô hình tạo được dự báo.
- Backend trả đúng phản hồi.
- SNS gửi email khi vượt ngưỡng.
- Có log và minh chứng cho các bước chính.

## 7. Kết quả đạt được

- Có ca kiểm thử cho từng mô-đun.
- Có quy trình báo lỗi thống nhất.
- Có minh chứng kiểm thử tích hợp.
- Lỗi có thể được xác định theo từng chặng.
