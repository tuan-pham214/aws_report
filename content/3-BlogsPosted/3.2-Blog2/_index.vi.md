---
title: "Reserved Concurrency vs Provisioned Concurrency trong AWS Lambda"
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
includeInReport: false
---

**Liên kết bài viết đã đăng:** https://www.facebook.com/share/p/1KpgVzwNtB/?

## Mở đầu

Reserved Concurrency và Provisioned Concurrency có tên khá giống nhau nên dễ bị hiểu là hai cách khác nhau để “đặt trước Lambda”. Thực tế, chúng giải quyết hai vấn đề riêng:

- **Reserved Concurrency** dùng để giữ một phần concurrency cho function và đồng thời giới hạn mức scale tối đa của function đó.
- **Provisioned Concurrency** khởi tạo sẵn execution environment để giảm độ trễ cold start.

Một cơ chế tập trung vào **năng lực xử lý đồng thời và giới hạn tài nguyên**; cơ chế còn lại tập trung vào **độ trễ khi bắt đầu xử lý request**.

## Concurrency trong Lambda là gì?

Concurrency là số request mà một Lambda function đang xử lý cùng lúc. Nếu một request cần trung bình một giây và có 20 request đến trong cùng thời điểm, function có thể cần khoảng 20 execution environment để xử lý song song.

Khi chưa có execution environment sẵn sàng, Lambda phải khởi tạo runtime, tải code và chạy phần initialization trước khi gọi handler. Khoảng thời gian này thường được gọi là **cold start**.

Hai câu hỏi cần tách riêng là:

1. Function được phép hoặc được bảo đảm xử lý tối đa bao nhiêu request đồng thời?
2. Các execution environment có được khởi tạo sẵn để phản hồi nhanh hay không?

Reserved Concurrency trả lời câu hỏi thứ nhất. Provisioned Concurrency chủ yếu giải quyết câu hỏi thứ hai.

## Reserved Concurrency là gì?

Khi đặt Reserved Concurrency cho một function, số concurrency đó được dành riêng cho function và không được function khác trong cùng Region sử dụng. Đồng thời, đây cũng là giới hạn concurrency tối đa của function.

Ví dụ:

```text
forecast-api: Reserved Concurrency = 20
```

Kết quả:

- `forecast-api` có tối đa 20 invocation chạy đồng thời.
- Phần concurrency này được giữ riêng, giúp function quan trọng không bị function khác dùng hết pool.
- Khi invocation thứ 21 đến trong lúc cả 20 slot đều bận, request mới có thể bị throttled.

Reserved Concurrency phù hợp với hai mục tiêu tưởng như trái ngược:

- **Bảo đảm năng lực:** giữ một phần concurrency cho function quan trọng.
- **Đặt giới hạn:** không cho function scale quá mức và làm quá tải database hoặc dịch vụ downstream.

Việc cấu hình Reserved Concurrency không có phụ phí riêng. Tuy nhiên, nó không khởi tạo sẵn execution environment, vì vậy cold start vẫn có thể xảy ra.

### Khi nào nên dùng Reserved Concurrency?

- Function quan trọng cần có phần concurrency riêng.
- Function kết nối database chỉ cho phép số connection giới hạn.
- Consumer có thể đẩy quá nhiều request tới API bên ngoài.
- Cần tạm dừng function bằng cách đặt Reserved Concurrency về `0`.
- Muốn cô lập một workload có nguy cơ chiếm hết concurrency dùng chung.

### Điểm cần lưu ý

- Đặt giá trị quá thấp sẽ làm function bị throttled dù tài khoản vẫn còn concurrency.
- Phần concurrency đã reserve không còn nằm trong pool dùng chung cho các function khác.
- Reserved Concurrency không phải cơ chế giảm cold start.
- Khi giới hạn downstream, vẫn cần cấu hình retry, timeout và xử lý lỗi phù hợp.

## Provisioned Concurrency là gì?

Provisioned Concurrency yêu cầu Lambda khởi tạo trước một số execution environment. Khi request đến trong phạm vi số lượng đã provision, môi trường đã sẵn sàng để xử lý nên độ trễ cold start được giảm đáng kể.

Ví dụ:

```text
forecast-api: Provisioned Concurrency = 5
```

Lambda duy trì năm execution environment đã được khởi tạo. Năm request đồng thời đầu tiên có thể sử dụng các môi trường này. Nếu lưu lượng vượt quá năm và function vẫn còn concurrency khả dụng, phần request vượt mức có thể chạy trên môi trường on-demand và vẫn có khả năng gặp cold start.

Provisioned Concurrency được cấu hình cho **published version hoặc alias**, không cấu hình trực tiếp cho `$LATEST`. Request cũng phải gọi đúng version hoặc alias đã được provision thì mới sử dụng các môi trường được khởi tạo sẵn.

### Khi nào nên dùng Provisioned Concurrency?

- API đồng bộ cần phản hồi ổn định.
- Ứng dụng web hoặc mobile nhạy cảm với độ trễ.
- Runtime, dependency hoặc bước initialization làm cold start kéo dài.
- Lưu lượng có khung giờ cao điểm tương đối dự đoán được.
- Function có yêu cầu latency rõ ràng trong production.

### Chi phí và vận hành

Provisioned Concurrency phát sinh chi phí trong thời gian được cấu hình, kể cả khi execution environment chưa xử lý request. Vì vậy, không nên đặt một con số lớn chỉ để đề phòng.

Có thể dùng Application Auto Scaling để thay đổi Provisioned Concurrency theo lịch hoặc theo mức sử dụng. Khi theo dõi, nên quan sát các metric như:

- `ProvisionedConcurrentExecutions`
- `ProvisionedConcurrencyUtilization`
- `ProvisionedConcurrencySpilloverInvocations`
- `ConcurrentExecutions`
- `Throttles`

## Bảng so sánh nhanh

| Tiêu chí | Reserved Concurrency | Provisioned Concurrency |
| --- | --- | --- |
| Mục tiêu chính | Giữ và giới hạn concurrency | Khởi tạo sẵn môi trường thực thi |
| Đặt giới hạn tối đa | Có | Không tự đặt trần nếu không kết hợp Reserved Concurrency |
| Giảm cold start | Không | Có, với request nằm trong phần đã provision |
| Phạm vi cấu hình | Function | Published version hoặc alias |
| Phụ phí cấu hình | Không | Có |
| Khi vượt mức | Bị throttled ở giới hạn reserved | Có thể dùng concurrency on-demand nếu còn khả dụng |
| Tình huống phù hợp | Bảo vệ function và downstream | API cần latency ổn định |

Có thể ghi nhớ ngắn gọn:

```text
Reserved Concurrency  = reserve + limit
Provisioned Concurrency = pre-warm
```

## Có thể dùng cả hai cùng lúc không?

Có. Một function có thể sử dụng cả Reserved Concurrency và Provisioned Concurrency.

Ví dụ:

```text
Reserved Concurrency = 20
Provisioned Concurrency cho alias prod = 5
```

Trong cấu hình này:

- Function có trần 20 invocation đồng thời.
- Năm execution environment được khởi tạo sẵn cho alias `prod`.
- Lưu lượng vượt quá năm có thể dùng thêm concurrency on-demand cho đến giới hạn 20.
- Lượng Provisioned Concurrency không được lớn hơn Reserved Concurrency của function.

Kết hợp hai cơ chế giúp vừa kiểm soát tác động lên downstream, vừa cải thiện latency cho lượng request quan trọng.

## Ví dụ trong hệ thống dự báo AQI

Giả sử hệ thống có hai Lambda function:

```text
forecast-api
    Nhận yêu cầu dự báo từ người dùng
    Gọi model endpoint và trả kết quả

save-sensor-data
    Xử lý telemetry nền
    Ghi dữ liệu cảm biến vào kho lưu trữ
```

`forecast-api` là API đồng bộ nên người dùng cảm nhận trực tiếp thời gian phản hồi. Có thể cấu hình Provisioned Concurrency cho alias `prod` vào giờ cao điểm để giảm cold start.

Trong khi đó, cả hai function đều có thể cần Reserved Concurrency:

- Giữ đủ năng lực cho `forecast-api`, tránh bị workload nền dùng hết concurrency chung.
- Giới hạn `save-sensor-data` để không tạo quá nhiều kết nối hoặc request tới downstream.

Không nhất thiết phải dùng Provisioned Concurrency cho workload nền nếu vài trăm mili giây cold start không ảnh hưởng đến yêu cầu nghiệp vụ.

## Các hiểu nhầm thường gặp

### Reserved Concurrency sẽ loại bỏ cold start

Không đúng. Reserved Concurrency giữ capacity nhưng execution environment vẫn được tạo theo nhu cầu.

### Provisioned Concurrency là giới hạn tối đa

Không hẳn. Khi phần provisioned đã được dùng hết, function có thể tiếp tục scale bằng concurrency on-demand nếu không bị Reserved Concurrency hoặc quota tài khoản giới hạn.

### Cấu hình cho alias nhưng gọi `$LATEST` vẫn được hưởng lợi

Không đúng. Invocation cần nhắm đúng version hoặc alias có Provisioned Concurrency.

### Càng provision nhiều càng tốt

Không đúng. Provisioned Concurrency phát sinh chi phí khi được duy trì. Giá trị nên dựa trên metric thực tế và yêu cầu latency.

### Reserved và Provisioned là hai lựa chọn loại trừ nhau

Không đúng. Chúng có thể kết hợp vì giải quyết hai lớp vấn đề khác nhau.

## Kết luận

Reserved Concurrency và Provisioned Concurrency không thể thay thế cho nhau:

- Dùng **Reserved Concurrency** khi cần giữ capacity cho function quan trọng hoặc đặt trần để bảo vệ downstream.
- Dùng **Provisioned Concurrency** khi cần giảm cold start và giữ latency ổn định.
- Dùng **cả hai** khi API vừa cần phản hồi nhanh vừa cần giới hạn scale rõ ràng.

Trước khi cấu hình, nên xác định vấn đề thực tế là thiếu concurrency, quá tải downstream hay cold start. Chọn đúng cơ chế sẽ giúp hệ thống ổn định mà không phát sinh chi phí không cần thiết.

## Tài liệu AWS chính thức

<https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html>

<https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html>

<https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html>

<https://aws.amazon.com/lambda/pricing/>
