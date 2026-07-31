---
title: "Worklog Tuần 6"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
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

- Hoàn thành tích hợp FastAPI với SageMaker Endpoint theo hợp đồng đã thống nhất cùng nhóm Data/ML.
- Hoàn thiện chu trình lấy dự báo, quy đổi AQI, đánh giá ngưỡng và gửi cảnh báo bằng Amazon SNS.
- Kiểm tra đường đi end-to-end và xử lý đầy đủ lỗi cấu hình, timeout hoặc response sai schema.

### Công việc theo ngày/thời gian

| Thứ | Công việc | Ngày bắt đầu | Ngày hoàn thành | Kết quả |
| --- | --- | --- | --- | --- |
| 2 | Xây dựng lớp đọc quan sát mới nhất theo `station_id` và ánh xạ sang payload của SageMaker. Kiểm tra timezone, đơn vị PM2.5, trường nhiệt độ/độ ẩm và horizon trước khi gửi request. | 21/07/2025 | 21/07/2025 | Payload suy luận có schema xác định và tách biệt khỏi model nội bộ của FastAPI. |
| 3 | Xây dựng client gọi SageMaker Runtime bằng IAM role của EC2, giới hạn thời gian chờ và retry cho lỗi tạm thời. Phối hợp với nhóm ML đối chiếu tên trường, schema response và kết nối endpoint đã triển khai. | 22/07/2025 | 22/07/2025 | FastAPI gọi thành công SageMaker Endpoint bằng IAM role và nhận kết quả đúng hợp đồng. |
| 4 | Hoàn thiện `GET /forecast/{station_id}`; chuẩn hóa kết quả gồm thời điểm quan sát, thời điểm dự báo, horizon, PM2.5 dự báo và AQI. Ánh xạ lỗi thành 503 khi thiếu cấu hình, 504 khi timeout và 502 khi response không hợp lệ. | 23/07/2025 | 23/07/2025 | API trả kết quả dự báo đúng schema và phản hồi lỗi nhất quán trong các trường hợp kiểm thử. |
| 5 | Hoàn thiện chu trình forecast–alert: quy đổi PM2.5 sang AQI, tìm người đăng ký đã xác nhận đúng trạm/ngưỡng và publish một thông báo qua SNS. Bổ sung khóa cooldown để hạn chế cảnh báo trùng. | 24/07/2025 | 24/07/2025 | Luồng cảnh báo gửi đúng người nhận và không lặp lại khi cùng một dự báo được xử lý nhiều lần. |
| 6 | Phối hợp chạy kiểm thử end-to-end từ dữ liệu trạm, SageMaker Endpoint, FastAPI đến email cảnh báo SNS; rà soát log theo `station_id`, trạng thái chu trình và thời gian xử lý. | 25/07/2025 | 25/07/2025 | Xác nhận thành công toàn bộ chu trình dự báo và cảnh báo trong môi trường triển khai của nhóm. |

### Kết quả đạt được

- Hoàn thành API dự báo theo hợp đồng rõ ràng và có kiểm tra schema đầu ra.
- Tích hợp thành công SageMaker Runtime bằng IAM role, có timeout và retry giới hạn.
- Hoàn thiện luồng đánh giá ngưỡng và gửi cảnh báo Amazon SNS có cơ chế cooldown.
- Phối hợp với nhóm Data/ML xác nhận endpoint, schema mô hình và kết quả dự báo.
- Kiểm thử thành công chu trình end-to-end từ dữ liệu trạm đến dự báo và email cảnh báo.
