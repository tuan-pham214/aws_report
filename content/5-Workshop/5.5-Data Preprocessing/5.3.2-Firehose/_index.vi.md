---
title: "Cấu hình Amazon Data Firehose"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

AWS Kinesis Data Firehose giúp luân chuyển dữ liệu streaming từ thiết bị IoT và ghi thành các file (batch) xuống Data Lake một cách hoàn toàn tự động.

#### Các bước khởi tạo Firehose:

1. Truy cập dịch vụ **Kinesis** trên AWS Console.
2. Tại bảng điều khiển, cuộn xuống phần **Kinesis Data Firehose** và nhấn **Create delivery stream**.
3. Tại phần **Choose source and destination**:
   * **Source:** Chọn `Direct PUT` (Dữ liệu từ IoT Rule sẽ được đẩy thẳng vào nguồn này).
   * **Destination:** Chọn `Amazon S3`.

   ![Chọn Source và Destination](/images/5-Workshop/5.3-data/5.3.2.1.png)

4. Đặt tên cho luồng tại mục **Firehose stream name**, ví dụ: `local-aqi-dev-firehose-telemetry`.

   ![Đặt tên Delivery Stream](/images/5-Workshop/5.3-data/5.3.2.2.png)

#### Cấu hình đích đến

1. Kéo xuống phần **Destination settings**, tại mục **S3 bucket**, nhấn Browse và chọn bucket S3 bạn vừa tạo ở bài 5.5.1 (ví dụ: `s3://local-aqi-dev-s3-raw`).

   ![Chọn đích đến S3](/images/5-Workshop/5.3-data/5.3.2.3.png)

2. **S3 bucket prefix:** Copy và dán chính xác chuỗi sau. Thao tác này giúp tự động phân mảnh (partition) dữ liệu theo chuẩn Hive-style, tối ưu hóa tốc độ và chi phí khi truy vấn bằng Athena sau này:
   ```text
   raw/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/hour=!{timestamp:HH}/
   ```

   ![Cấu hình prefix dữ liệu và lỗi trên S3](/images/5-Workshop/5.3-data/5.3.2.4.png)

#### Cấu hình quyền truy cập và thẻ tài nguyên

1. Tại phần **Service access**, giữ nguyên tùy chọn **Create or update IAM role**. Hệ thống AWS sẽ tự động tạo một Role mới (ví dụ: `KinesisFirehoseServiceRole-...`) với các quyền Least Privilege cần thiết để ghi dữ liệu vào S3.

   ![Cấu hình IAM Role](/images/5-Workshop/5.3-data/5.3.2.5.png)

2. Cuộn xuống phần **Tags - optional** (Gắn thẻ tài nguyên). Tương tự như S3, hãy thêm các thẻ để quản lý dự án:
   * **Key:** `Project` | **Value:** `local-aqi-forecasting`
   * **Key:** `Environment` | **Value:** `dev`
   * **Key:** `Owner` | **Value:** `quynh-tam`
   * **Key:** `Module` | **Value:** `data`

   ![Gắn thẻ tài nguyên Firehose](/images/5-Workshop/5.3-data/5.3.2.6.png)

3. Kéo xuống dưới cùng và nhấn nút **Create Firehose stream** màu cam. 

Quá trình khởi tạo có thể mất vài phút. Khi bạn quay lại màn hình danh sách Firehose streams và thấy trạng thái chuyển sang **Active** (màu xanh lá), luồng dữ liệu của bạn đã sẵn sàng để đón nhận thông điệp từ IoT Core!

   ![Trạng thái Active](/images/5-Workshop/5.3-data/5.3.2.7.png)
