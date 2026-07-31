---
title: "Chia sẻ, đóng góp ý kiến"
date: 2024-01-01
weight: 7
chapter: false
pre: " <b> 7. </b> "
includeInReport: false
---

## Cảm nhận chung

Chương trình **First Cloud AI Journey** mang lại cho em một trải nghiệm thực tập có tính thực hành cao. Thay vì chỉ tìm hiểu từng dịch vụ AWS riêng lẻ, em được tham gia xây dựng một dự án có đầy đủ các bước từ phân tích bài toán, thiết kế kiến trúc, triển khai, tích hợp đến kiểm thử và trình bày kết quả. Nhờ đó, em hiểu rõ hơn sự khác biệt giữa việc học một công nghệ và việc sử dụng công nghệ đó để giải quyết một bài toán cụ thể.

Trong quá trình thực tập, em được tiếp xúc với cách làm việc theo nhóm và phải chủ động phối hợp với các phần việc có liên quan. Đối với dự án Local AQI, backend không thể hoàn thành độc lập mà cần thống nhất dữ liệu với nhóm ingestion, Data và ML. Đây là trải nghiệm có giá trị vì em học được cách trao đổi bằng schema, API contract và kết quả kiểm thử thay vì chỉ mô tả yêu cầu bằng lời nói.

## Môi trường học tập và làm việc

Môi trường của chương trình cởi mở và khuyến khích sinh viên tự tìm hiểu. Em có cơ hội thử nghiệm trên AWS, gặp lỗi thực tế và tự kiểm tra nguyên nhân trước khi trao đổi với mentor. Cách tiếp cận này giúp em nhớ kiến thức lâu hơn và hình thành thói quen tìm bằng chứng từ log, tài liệu và trạng thái tài nguyên.

Các hoạt động cộng đồng và sự kiện đi kèm cũng giúp em có góc nhìn rộng hơn về nghề nghiệp trong lĩnh vực cloud. Bên cạnh kỹ thuật, các diễn giả còn chia sẻ về thái độ học hỏi, kỹ năng giao tiếp, khả năng làm việc nhóm và trách nhiệm khi vận hành hệ thống thực tế.

## Sự hỗ trợ của mentor và đội ngũ chương trình

Mentor và đội ngũ FCAJ hỗ trợ sinh viên trong việc định hướng nội dung, giải đáp vướng mắc và theo dõi tiến độ. Điều em đánh giá cao là mentor không chỉ đưa ra đáp án, mà thường gợi ý cách tiếp cận để sinh viên tự kiểm tra và đưa ra lựa chọn. Điều này giúp em tự tin hơn khi phải xử lý một vấn đề mới.

Các tài liệu hướng dẫn và workshop cung cấp nền tảng cần thiết để bắt đầu với AWS. Khi thực hiện dự án, em có thể dựa trên kiến thức đó để tiếp tục tìm hiểu EC2, IAM, S3, SageMaker, DynamoDB, SNS và CloudWatch theo đúng nhu cầu của phần việc.

## Mức độ phù hợp với chuyên ngành và định hướng cá nhân

Nội dung thực tập phù hợp với chuyên ngành **Khoa học máy tính** và định hướng phát triển Backend/Cloud của em. Dự án yêu cầu kiến thức lập trình, API, cơ sở dữ liệu, mạng máy tính, bảo mật và triển khai hệ thống. Đồng thời, việc tích hợp với mô hình dự báo giúp em hiểu thêm cách backend làm việc với một thành phần machine learning trong sản phẩm thực tế.

Sau chương trình, em xác định rõ hơn rằng mình muốn tiếp tục phát triển theo hướng **Backend Engineer**, đồng thời bổ sung kiến thức về cloud infrastructure và vận hành hệ thống.

## Điều em hài lòng nhất

Điều em hài lòng nhất là nhóm đã hoàn thiện được một luồng sản phẩm có thể trình bày trực tiếp: dữ liệu telemetry được đưa qua IoT Core và Firehose vào S3, dữ liệu được xử lý để phục vụ dự báo, SageMaker Endpoint trả kết quả cho FastAPI, còn DynamoDB và SNS hỗ trợ chức năng đăng ký và gửi cảnh báo AQI qua email.

Quá trình này giúp em thấy rõ kết quả của phần việc cá nhân trong một hệ thống lớn hơn. Việc API chạy trên EC2 và có thể gọi trực tiếp qua Swagger cũng tạo cho em cảm giác đã chuyển từ một bài tập lập trình sang một sản phẩm có thể triển khai và kiểm thử.

## Đề xuất cho chương trình

- Có thể tổ chức một buổi thống nhất kiến trúc và hợp đồng dữ liệu ngay đầu giai đoạn làm dự án để các nhóm giảm thời gian chỉnh sửa khi tích hợp.
- Nên có thêm các buổi technical review ngắn theo từng mốc, tập trung vào kiến trúc, bảo mật, chi phí và khả năng kiểm thử thay vì chỉ kiểm tra tiến độ chung.
- Có thể cung cấp checklist demo và bàn giao sớm hơn để sinh viên chủ động chuẩn bị log, ảnh minh chứng, tài khoản thử nghiệm và kịch bản xử lý khi dịch vụ phản hồi chậm.
- Nếu điều kiện cho phép, chương trình có thể bổ sung một buổi code review hoặc vận hành sự cố mô phỏng để sinh viên hiểu rõ hơn tiêu chuẩn của môi trường production.
- Nên tiếp tục duy trì các sự kiện cộng đồng để sinh viên được trao đổi với kỹ sư đang làm việc thực tế và hiểu thêm yêu cầu tuyển dụng của doanh nghiệp.

## Mong muốn cá nhân

Em mong muốn tiếp tục tham gia các hoạt động của cộng đồng FCAJ và những chương trình chuyên sâu hơn về Backend, kiến trúc cloud và AI application. Nếu giới thiệu cho bạn bè có định hướng theo lĩnh vực cloud, em sẽ khuyến khích các bạn tham gia vì chương trình không chỉ cung cấp kiến thức AWS mà còn tạo cơ hội làm dự án nhóm, gặp gỡ cộng đồng và rèn luyện cách trình bày một sản phẩm kỹ thuật.

Nhìn chung, kỳ thực tập là một trải nghiệm có ý nghĩa đối với em. Chương trình giúp em nhận ra những kiến thức mình còn thiếu, đồng thời tạo động lực để tiếp tục học tập và xây dựng các dự án có chất lượng tốt hơn trong tương lai.
