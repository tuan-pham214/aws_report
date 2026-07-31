---
title: "Điều phối Dự án"
date: 2026-07-31
weight: 2
chapter: false
pre: "<b>2. </b>"
---

## 1. Mục tiêu

Quản lý tiến độ, phân công trách nhiệm và theo dõi quan hệ phụ thuộc giữa các mô-đun để tránh tình trạng từng thành viên hoàn thành phần riêng nhưng hệ thống không thể tích hợp.

## 2. Quản lý công việc bằng GitHub

Các công việc được chia theo các nhóm:

```text
Thu nhận dữ liệu
Xử lý dữ liệu
Học máy
Dịch vụ backend
DevOps
Tích hợp
```

Mỗi công việc gồm:

- Tên công việc
- Người phụ trách
- Mục tiêu
- Mức ưu tiên
- Danh sách kiểm tra
- Điều kiện tiên quyết
- Minh chứng
- Tiêu chí hoàn thành
- Công việc tiếp theo

Trạng thái được thống nhất:

```text
Chưa thực hiện
Đang thực hiện
Bị chặn
Đang rà soát
Hoàn thành
```

![Bảng GitHub Project quản lý sprint Local AQI](/images/5-Workshop/5.3-DevOps/github-project-board.png)

Một công việc chỉ được chuyển sang trạng thái hoàn thành khi có đầu ra thực tế và minh chứng đi kèm. Nếu bị chặn, nhóm phải ghi rõ thành viên, tài nguyên hoặc đầu ra phụ thuộc đang gây trở ngại.

![Ví dụ công việc có người phụ trách, mục tiêu và danh sách kiểm tra](/images/5-Workshop/5.3-DevOps/github-task-example.png)

## 3. Điều phối phụ thuộc và tích hợp

Luồng phụ thuộc chính của dự án:

```text
Simulator
-> S3 Raw
-> S3 Processed
-> Bộ dữ liệu học máy
-> Kết quả dự báo
-> Backend
-> SNS
```

### 3.1. Thu nhận dữ liệu đến xử lý dữ liệu

Hai nhóm cần thống nhất MQTT topic, cấu trúc telemetry, phiên bản schema, múi giờ UTC, mã trạm, tên trường, kiểu dữ liệu và đơn vị đo.

### 3.2. Xử lý dữ liệu đến học máy

Hai nhóm cần thống nhất S3 URI đầu vào, cấu trúc bộ dữ liệu đã xử lý, tần suất theo giờ, quy tắc xử lý giá trị thiếu và trùng lặp, quy ước phân vùng, khoảng thời gian và danh sách trạm.

### 3.3. Học máy đến dịch vụ backend

Hai nhóm cần thống nhất đầu vào mô hình, cấu trúc kết quả dự báo, khoảng dự báo, mã trạm, phản hồi lỗi, ngưỡng cảnh báo và cách backend đọc tệp dự báo hoặc gọi mô hình.

## 4. Quy trình báo cáo tiến độ

Khi cập nhật công việc, thành viên cần trả lời:

```text
Đang làm công việc nào?
Đã hoàn thành phần nào?
Đầu ra hiện tại là gì?
Minh chứng được lưu ở đâu?
Đang bị chặn bởi ai hoặc tài nguyên nào?
Công việc tiếp theo là gì?
Có tạo thêm tài nguyên AWS không?
Có phát sinh thêm chi phí không?
```

## 5. Điều phối tài liệu

Mỗi thành viên viết tài liệu cho mô-đun mình phụ trách. Vai trò DevOps chịu trách nhiệm:

- Thống nhất định dạng chung.
- Kiểm tra tên tài nguyên và Region.
- Kiểm tra sự thống nhất giữa đầu vào và đầu ra của các nhóm.
- Tổng hợp minh chứng.
- Rà soát nội dung trước khi đưa vào báo cáo và buổi trình diễn.

Tài liệu được nhóm theo kiến trúc tổng quan, schema telemetry, định nghĩa bộ dữ liệu đã xử lý, kế hoạch kiểm thử, kịch bản trình diễn và danh sách minh chứng.

![Cấu trúc tài liệu dùng chung của dự án](/images/5-Workshop/5.3-DevOps/project-docs-structure.png)

## 6. Tiêu chí hoàn thành chung

Một công việc chỉ được nghiệm thu khi mã nguồn hoặc tài nguyên AWS hoạt động, có kiểm thử thực tế, có đầu ra mong đợi, có log hoặc ảnh chụp minh chứng, có hướng dẫn chạy lại, không để lộ thông tin xác thực và tài liệu liên quan đã được cập nhật.

## 7. Kết quả đạt được

- Công việc được phân chia theo nhóm và quan hệ phụ thuộc.
- Mỗi thành viên có trách nhiệm rõ ràng.
- Các điểm bàn giao giữa nhóm được xác định cụ thể.
- Công việc bị chặn được phát hiện sớm.
- Tài liệu và minh chứng được chuẩn hóa trước khi tổng hợp.
