---
title: "3 thói quen giúp người mới học AWS tránh phát sinh chi phí ngoài ý muốn"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
includeInReport: false
---

**Liên kết bài viết đã đăng:** https://www.facebook.com/share/p/1BxtXE5ien/?

## Mở đầu

Khi mới học AWS, chúng ta thường tập trung vào việc tạo tài nguyên để bài thực hành chạy được: khởi tạo EC2, tạo bucket S3, triển khai database hoặc thử một SageMaker endpoint. Sau khi thấy kết quả, nhiều người đóng trình duyệt và cho rằng buổi lab đã kết thúc.

Tuy nhiên, tài nguyên trên cloud vẫn tiếp tục tồn tại cho đến khi được dừng hoặc xóa đúng cách. Một EC2 còn chạy, một EBS volume bị bỏ quên, một public IPv4 address hoặc một endpoint vẫn hoạt động đều có thể tiếp tục phát sinh chi phí. Chỉ cần làm nhiều bài lab ở nhiều Region khác nhau, việc kiểm soát tài nguyên sẽ nhanh chóng trở nên khó khăn.

Ba thói quen dưới đây giúp mình quản lý môi trường thực hành rõ ràng hơn và giảm nguy cơ nhận một hóa đơn ngoài dự kiến.

## Thói quen 1: Thống nhất Region trước khi tạo tài nguyên

Region là một trong những thông tin đầu tiên cần kiểm tra khi bắt đầu buổi lab. Nhiều tài nguyên AWS chỉ hiển thị trong Region đã tạo. Nếu hôm nay tạo EC2 ở Singapore nhưng ngày mai mở Console tại Tokyo, chúng ta có thể tưởng instance đã biến mất và vô tình tạo thêm một instance khác.

Với nhóm thực hiện dự án Local AQI Forecasting & Alert System, nhóm thống nhất sử dụng:

```text
ap-southeast-1 (Asia Pacific - Singapore)
```

Trước khi thao tác, mình kiểm tra Region ở góc trên bên phải AWS Management Console. Nếu dùng AWS CLI, cần kiểm tra cả profile và Region mặc định:

```bash
aws configure list
```

Với câu lệnh quan trọng, có thể chỉ rõ Region để tránh chạy nhầm:

```bash
aws ec2 describe-instances --region ap-southeast-1
```

Thói quen này mang lại ba lợi ích:

- Dễ tìm lại tài nguyên sau mỗi buổi thực hành.
- Tránh tạo trùng dịch vụ ở nhiều Region.
- Giúp cả nhóm thống nhất endpoint, log và cấu hình triển khai.

Nếu dự án thật sự cần nhiều Region, danh sách Region sử dụng nên được ghi rõ trong tài liệu kiến trúc. Không nên chuyển Region tùy ý chỉ vì Console đang hiển thị một lựa chọn khác.

## Thói quen 2: Gắn tag ngay khi tạo tài nguyên

Tên tài nguyên cho biết một phần mục đích, nhưng chưa đủ để trả lời các câu hỏi như: tài nguyên thuộc dự án nào, môi trường nào, ai đang phụ trách và dùng cho module nào.

Nhóm mình sử dụng bộ tag cơ bản:

```text
Project=local-aqi-forecasting
Environment=dev
Owner=<tên thành viên>
Module=<backend|data|ml|ingestion|devops>
```

Ví dụ, một EC2 chạy FastAPI có thể được gắn:

```text
Project=local-aqi-forecasting
Environment=dev
Owner=backend-team
Module=backend
```

Tag giúp việc vận hành thuận tiện hơn:

- Tìm và lọc tài nguyên liên quan đến cùng một dự án.
- Xác định người hoặc nhóm chịu trách nhiệm.
- Phân biệt môi trường `dev`, `staging` và `production`.
- Theo dõi chi phí theo dự án hoặc module sau khi kích hoạt cost allocation tag.
- Hạn chế xóa nhầm tài nguyên của người khác.

Tag nên được tạo cùng lúc với tài nguyên thay vì chờ đến cuối tuần mới bổ sung. Khi số lượng tài nguyên tăng, việc nhớ chính xác mục đích của từng security group, volume hoặc bucket sẽ rất khó.

Không nên ghi mật khẩu, access key, email cá nhân hoặc dữ liệu nhạy cảm vào tag. Tag là metadata quản trị, không phải nơi lưu secret.

## Thói quen 3: Kiểm tra và dọn dẹp sau mỗi buổi lab

Hoàn thành bài thực hành chưa có nghĩa là vòng đời tài nguyên đã kết thúc. Trước khi rời khỏi AWS Console, mình dành vài phút kiểm tra các dịch vụ vừa sử dụng.

Danh sách cần rà soát thường gồm:

- EC2 instance còn ở trạng thái `running`.
- EBS volume không còn gắn với instance.
- Elastic IP hoặc public IPv4 address không còn sử dụng.
- Load balancer, NAT Gateway hoặc VPC endpoint được tạo để thử nghiệm.
- SageMaker notebook, endpoint hoặc processing resource vẫn hoạt động.
- Snapshot, log group và dữ liệu thử không còn cần thiết.
- Security group tạm thời mở cổng rộng hơn yêu cầu.

Một điểm dễ nhầm là **dừng EC2 không đồng nghĩa với mọi chi phí liên quan đều dừng**. Khi instance ở trạng thái `stopped`, phần compute không tiếp tục tính như lúc chạy, nhưng EBS volume và một số tài nguyên đi kèm vẫn có thể phát sinh chi phí.

Vì vậy, với mỗi tài nguyên mình tự hỏi:

1. Tài nguyên này có cần dùng lại trong buổi sau không?
2. Nếu cần, có thể dừng hoặc giảm cấu hình tạm thời không?
3. Nếu không cần, đã xóa cả tài nguyên phụ thuộc hay chưa?
4. Dữ liệu nào cần sao lưu trước khi xóa?

Nếu dùng hạ tầng dưới dạng mã, việc tạo và hủy một môi trường lab có thể được chuẩn hóa bằng CloudFormation, AWS CDK hoặc Terraform. Dù dùng cách nào, vẫn nên mở Console kiểm tra lần cuối vì có thể tồn tại tài nguyên được tạo thủ công ngoài template.

## AWS Budgets là lớp cảnh báo bổ sung

Bên cạnh ba thói quen trên, tài khoản học tập nên có AWS Budgets để cảnh báo khi chi phí thực tế hoặc dự báo vượt ngưỡng.

Ví dụ, có thể cấu hình các mốc cảnh báo nhỏ phù hợp với ngân sách cá nhân và gửi thông báo qua email. Cảnh báo giúp phát hiện bất thường sớm, nhưng không thay thế việc kiểm tra tài nguyên vì dữ liệu chi phí có thể cập nhật trễ.

AWS Budgets cũng không tự động bảo đảm mọi dịch vụ sẽ dừng ngay khi chạm ngưỡng. Vì vậy, cách an toàn nhất vẫn là kết hợp:

```text
Region thống nhất
    + Tag rõ ràng
    + Dọn dẹp sau mỗi buổi lab
    + Cảnh báo ngân sách
```

## Bài học rút ra

Cloud cho phép tạo tài nguyên trong vài phút, nhưng sự thuận tiện đó đi kèm trách nhiệm quản lý toàn bộ vòng đời tài nguyên. Đối với người mới học AWS, ba thói quen hiệu quả nhất là:

1. Chọn đúng Region và giữ thống nhất trong suốt dự án.
2. Gắn tag ngay từ khi tạo tài nguyên.
3. Kiểm tra, dừng hoặc xóa tài nguyên sau mỗi buổi thực hành.

Đây không chỉ là cách tránh chi phí ngoài ý muốn. Những thói quen này còn tạo nền tảng tốt cho quản trị, bảo mật và vận hành hệ thống sau này.

## Tài liệu AWS chính thức

<https://docs.aws.amazon.com/whitepapers/latest/tagging-best-practices/tags-for-cost-allocation-and-financial-management.html>

<https://docs.aws.amazon.com/cost-management/latest/userguide/create-cost-budget.html>

<https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ec2-instance-lifecycle.html>
