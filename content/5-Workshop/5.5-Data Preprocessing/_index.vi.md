---
title: "Xây dựng đường ống dữ liệu"
date : 2026-07-31
weight : 5
chapter : false
pre : " <b> 5.5. </b> "
---

Trong mô-đun này, chúng ta sẽ thực hành xây dựng một đường ống dữ liệu hoàn chỉnh trên AWS.

Dữ liệu môi trường (PM2.5, nhiệt độ, độ ẩm) từ thiết bị IoT giả lập sẽ được thu thập, lưu trữ trong hồ dữ liệu S3, sau đó được kiểm định và tiền xử lý để sẵn sàng phục vụ mô hình học máy.

**Nội dung thực hành bao gồm:**
* **5.5.1:** Khởi tạo hồ dữ liệu trung tâm trên Amazon S3.
* **5.5.2:** Thiết lập luồng Kinesis Data Firehose để gom lô dữ liệu.
* **5.5.3:** Viết chương trình kiểm định chất lượng dữ liệu thô.
* **5.5.4:** Tiền xử lý, chuẩn hóa tần suất (1H) và xuất định dạng Parquet.
