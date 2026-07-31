---
title: "Worklog Tuần 3"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
includeInReport: true
reportTableColumns:
  - Thứ
  - Công việc
  - Ngày hoàn thành
reportHeadings:
  - Mục tiêu tuần
  - Công việc theo ngày/thời gian
  - Kết quả đạt được
reportType: worklog
---

### Mục tiêu tuần

- Làm rõ yêu cầu chức năng, yêu cầu phi chức năng và phạm vi MVP của hệ thống AQI.
- Chuẩn hóa bản ghi telemetry và kiểm chứng luồng ingestion từ simulator đến vùng dữ liệu thô.
- Chuẩn bị hợp đồng dữ liệu ổn định để backend, Data Lake và nhóm Data/ML có thể tích hợp.

### Công việc theo ngày/thời gian

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Kết quả |
| --- | --- | --- | --- | --- |
| 2 | Phân tích các ca sử dụng: nhận dữ liệu theo trạm, xem dự báo 24–48 giờ, đăng ký ngưỡng AQI và nhận cảnh báo. Bổ sung yêu cầu về độ trễ, khả năng truy vết, tính sẵn sàng và bảo vệ thông tin người đăng ký. | 30/06/2025 | 30/06/2025 | Có danh sách yêu cầu và tiêu chí chấp nhận cho MVP, đồng thời tách rõ chức năng bắt buộc và phần mở rộng. |
| 3 | Thiết kế schema telemetry gồm `station_id`, thời điểm đo theo UTC, PM2.5, nhiệt độ, độ ẩm và metadata nguồn. Quy định kiểu dữ liệu, miền giá trị, đơn vị đo và cách xử lý bản ghi thiếu hoặc trùng. | 01/07/2025 | 01/07/2025 | Hoàn thành phiên bản schema dùng chung và bộ ví dụ hợp lệ/không hợp lệ cho kiểm thử tích hợp. |
| 4 | Tìm hiểu API OpenAQ và phối hợp xây dựng Python telemetry simulator khi dữ liệu thật không liên tục. Thiết kế topic MQTT theo môi trường và trạm, đồng thời tránh đưa thông tin xác thực vào payload hoặc repository. | 02/07/2025 | 02/07/2025 | Hoàn thành nguồn phát dữ liệu thử nghiệm lặp lại được và quy ước topic phục vụ định tuyến. |
| 5 | Phối hợp kiểm tra rule từ AWS IoT Core hoặc bridge Mosquitto đến Kinesis Data Firehose; theo dõi cơ chế buffer, retry và bản ghi lỗi trước khi ghi vào S3. | 03/07/2025 | 03/07/2025 | Xác nhận thành công đường đi ingestion và hoàn thành checklist xử lý khi bản ghi không tới S3. |
| 6 | Kiểm tra object tại vùng `raw`, đối chiếu timestamp, partition, schema và khả năng truy vết về trạm nguồn. Cập nhật tài liệu giao tiếp giữa ingestion và backend/Data/ML. | 04/07/2025 | 04/07/2025 | Chốt hợp đồng đầu vào và danh sách trường hợp biên cho các bước xử lý tiếp theo. |

### Kết quả đạt được

- Hoàn thiện phạm vi MVP và tiêu chí chấp nhận cho các chức năng dự báo, đăng ký và cảnh báo.
- Chuẩn hóa schema telemetry, quy ước UTC, đơn vị đo và khóa nhận diện trạm.
- Chuẩn bị được simulator để hỗ trợ kiểm thử mà không phụ thuộc hoàn toàn vào dữ liệu OpenAQ tại một thời điểm.
- Phối hợp xác minh luồng MQTT–Firehose–S3 và xây dựng checklist chẩn đoán ingestion.
- Cung cấp hợp đồng dữ liệu rõ ràng cho phần backend; không nhận là người triển khai toàn bộ pipeline dữ liệu của nhóm.
