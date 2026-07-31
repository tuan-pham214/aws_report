---
title: "Tạo hồ dữ liệu Amazon S3"
date: 2026-07-31
weight: 1
chapter: false
pre: " <b> 5.5.1. </b> "
---

Amazon S3 sẽ đóng vai trò là Data Lake (hồ dữ liệu) trung tâm, nơi lưu trữ toàn bộ dữ liệu ở cả dạng thô (Raw) và dạng đã qua chế biến (Processed).

#### Các bước thực hiện trên AWS Console:

1. Đăng nhập vào [Amazon VPC console](https://us-east-1.console.aws.amazon.com/vpc/home?region=us-east-1#Home:), tìm kiếm dịch vụ **S3** và chọn **Create bucket**.
2. Tại phần **General configuration**:
   * **Bucket name:** Đặt tên cho bucket. Ví dụ: `local-aqi-dev-s3-raw` (Lưu ý: Tên bucket trên S3 phải là duy nhất trên toàn cầu, vì vậy hãy thêm hậu tố tên của bạn hoặc một mã ngẫu nhiên).

   ![Cấu hình tên Bucket](/images/5-Workshop/5.3-data/5.3.1.1.png)

3. Tại phần **Block Public Access settings for this bucket**:
   * Đảm bảo mục **Block all public access** đã được tích chọn. Đây là bước cực kỳ quan trọng để bảo vệ dữ liệu nội bộ của dự án khỏi các truy cập trái phép từ internet.

   ![Chặn quyền truy cập công khai](/images/5-Workshop/5.3-data/5.3.1.2.png)

4. Cuộn xuống phần **Tags - optional** (Gắn thẻ tài nguyên). Việc gắn thẻ giúp team dễ dàng quản lý chi phí và phân loại tài nguyên dự án sau này. Nhấn **Add new tag** và thêm các thẻ giống như sau:
   * **Key:** `Project` | **Value:** `local-aqi-forecasting`
   * **Key:** `Environment` | **Value:** `dev`
   * **Key:** `Owner` | **Value:** `quynh-tam`
   * **Key:** `Module` | **Value:** `data`

   ![Gắn thẻ tài nguyên AWS](/images/5-Workshop/5.3-data/5.3.1.3.png)

5. Cuộn xuống dưới cùng và nhấn **Create bucket** sẽ tạo ra bucket.
![buket đã tạo](/images/5-Workshop/5.3-data/5.3.1.4.png)

#### Quy hoạch tiền tố thư mục

Sau khi tạo xong, hãy nhấp vào tên Bucket vừa tạo trong danh sách để đi vào bên trong và thiết lập các thư mục làm việc. Lần lượt nhấn nút **Create folder** để tạo các thư mục gốc sau:
* `ml/`: Dành cho các tác vụ Machine Learning.
* `models/`: Lưu trữ các mô hình AI sau khi huấn luyện.
* `monitoring/`: Chứa các log và dữ liệu giám sát hệ thống.
* `processed/`: Dành cho dữ liệu đã làm sạch (bên trong có thể tạo thêm `ml-ready/`).
* `raw/`: Nơi Kinesis Firehose sẽ tự động đẩy dữ liệu thô vào.

**Kết quả sau cùng:** Giao diện bên trong bucket S3 của bạn sẽ có cấu trúc các thư mục gốc giống như hình bên dưới:

![Kết quả cấu trúc thư mục S3](/images/5-Workshop/5.3-data/5.3.1.5.png)

*Lưu ý: Dữ liệu bên trong thư mục `raw/` sẽ tự động được phân mảnh (partition) theo thời gian ở bước thiết lập Firehose tiếp theo, do đó bạn không cần tạo thêm các thư mục con theo ngày tháng thủ công.*
