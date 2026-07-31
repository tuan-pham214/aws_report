---
title: "Worklog Tuần 5"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
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

- Tìm hiểu quy trình làm sạch dữ liệu và baseline model để thống nhất đầu vào/đầu ra dự báo với nhóm Data/ML.
- Xây dựng chức năng đăng ký nhận cảnh báo theo trạm và ngưỡng AQI.
- Bổ sung kiểm thử cho validation, tính idempotent và các trạng thái đăng ký qua Amazon SNS.

### Công việc theo ngày/thời gian

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Kết quả |
| --- | --- | --- | --- | --- |
| 2 | Phối hợp với nhóm Data/ML loại bản ghi lỗi, xử lý thiếu dữ liệu, resample theo thời gian và tạo tập `processed`. Rà soát các đặc trưng PM2.5, nhiệt độ, độ ẩm và lịch sử theo trạm. | 14/07/2025 | 14/07/2025 | Hoàn thành tập dữ liệu đã xử lý và thống nhất các điều kiện đầu vào cho bước dự báo. |
| 3 | Phối hợp đánh giá baseline theo giá trị gần nhất/trung bình trượt và so sánh với DeepAR bằng MAE/RMSE. Thống nhất horizon, timestamp nguồn, timestamp dự báo và các giá trị bắt buộc trong response. | 15/07/2025 | 15/07/2025 | Hoàn thành đánh giá baseline, lựa chọn mô hình DeepAR và chốt hợp đồng dự báo giữa nhóm ML với backend. |
| 4 | Xây dựng `POST /subscribe` để nhận email, `station_id` và `threshold_aqi`. Thiết kế bản ghi người đăng ký có định danh ổn định, lưu trạng thái trong DynamoDB và xử lý request lặp lại an toàn. | 16/07/2025 | 16/07/2025 | API kiểm tra đầu vào và không tạo nhiều bản ghi khi người dùng gửi lại cùng một yêu cầu. |
| 5 | Tích hợp Amazon SNS để gửi email xác nhận đăng ký; phân biệt các trạng thái `provisioning`, `pending`, `confirmed` và `error`. Thiết kế filter theo trạm/ngưỡng trong giới hạn của MVP. | 17/07/2025 | 17/07/2025 | Hoàn thành luồng đăng ký, xác nhận email và lọc người nhận theo trạm/ngưỡng. |
| 6 | Viết unit/integration test với dịch vụ AWS được mock cho email sai định dạng, ngưỡng ngoài miền, đăng ký trùng, lỗi DynamoDB/SNS và request cạnh tranh. Cập nhật OpenAPI cho endpoint. | 18/07/2025 | 18/07/2025 | Bộ kiểm thử bao phủ luồng chính và các trường hợp lỗi quan trọng của chức năng đăng ký. |

### Kết quả đạt được

- Phối hợp hoàn thành tập dữ liệu đã xử lý và đánh giá baseline trước khi huấn luyện DeepAR.
- Thống nhất hợp đồng dự báo có timestamp nguồn, horizon và giá trị PM2.5/AQI phù hợp giữa nhóm Data/ML và backend.
- Hoàn thiện API đăng ký nhận cảnh báo với validation và cơ chế idempotent.
- Quản lý trạng thái đăng ký bằng DynamoDB và Amazon SNS thay vì lưu tạm trong bộ nhớ ứng dụng.
- Bổ sung kiểm thử để xác minh lỗi được xử lý an toàn, không trả thông tin nhạy cảm từ dịch vụ AWS.
