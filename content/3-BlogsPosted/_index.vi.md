---
title: "Các bài blog đã đăng"
weight: 3
chapter: false
pre: " <b> 3. </b> "
includeInReport: false
---

Trong thời gian thực tập, em đã hoàn thành ba bài viết chia sẻ những kinh nghiệm thực tế khi học và làm việc với AWS. Nội dung tập trung vào quản lý chi phí, khả năng mở rộng của AWS Lambda và cách sử dụng DynamoDB TTL đúng với bản chất của dịch vụ.

### [Blog 1 - 3 thói quen giúp người mới học AWS tránh phát sinh chi phí ngoài ý muốn](3.1-Blog1/)

Bài viết trình bày ba thói quen nên duy trì trong mỗi buổi thực hành AWS: thống nhất Region, gắn tag cho tài nguyên và kiểm tra, dọn dẹp sau khi hoàn thành. Đây là những bước đơn giản nhưng giúp người mới quản lý tài nguyên rõ ràng hơn và hạn chế chi phí ngoài dự kiến.

### [Blog 2 - Reserved Concurrency vs Provisioned Concurrency trong AWS Lambda](3.2-Blog2/)

Bài viết phân biệt hai cơ chế thường bị nhầm lẫn của AWS Lambda: Reserved Concurrency dùng để giữ và giới hạn năng lực xử lý đồng thời, còn Provisioned Concurrency khởi tạo sẵn execution environment để giảm độ trễ cold start.

### [Blog 3 - DynamoDB TTL không phải đồng hồ hẹn giờ](3.3-Blog3/)

Bài viết giải thích vì sao item đã hết hạn vẫn có thể xuất hiện trong DynamoDB, cách ứng dụng lọc dữ liệu hết hạn và lý do không nên dùng TTL cho những tác vụ cần chạy đúng thời điểm.
