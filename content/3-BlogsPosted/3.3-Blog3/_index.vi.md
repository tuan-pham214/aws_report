---
title: "DynamoDB TTL không phải đồng hồ hẹn giờ"
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
includeInReport: false
---

**Liên kết bài viết đã đăng:** https://www.facebook.com/share/p/1Ehu9A9z6w/?

## Mở đầu

Khi bật Time to Live (TTL) cho Amazon DynamoDB, nhiều người kỳ vọng item sẽ biến mất đúng vào giây được ghi trong thuộc tính hết hạn. Sau đó, ứng dụng vẫn đọc thấy item đã quá hạn và cho rằng TTL không hoạt động.

Thực tế, **DynamoDB TTL không phải đồng hồ hẹn giờ xóa dữ liệu chính xác**. Khi thời điểm TTL đã qua, item chỉ trở thành đối tượng đủ điều kiện để tiến trình nền của DynamoDB xóa. AWS cho biết việc xóa thường diễn ra trong vòng vài ngày sau thời điểm hết hạn.

Vì vậy, ứng dụng không nên dựa vào sự tồn tại của item để kết luận dữ liệu còn hiệu lực.

## DynamoDB TTL hoạt động như thế nào?

TTL cho phép chọn một thuộc tính trong table làm thời điểm hết hạn. Ví dụ:

```json
{
  "stationId": "HCM-Q1",
  "alertType": "PM25_HIGH",
  "expireAt": 1767225600
}
```

Để DynamoDB nhận diện đúng:

- `expireAt` phải là thuộc tính đã được cấu hình làm TTL cho table.
- Giá trị phải có kiểu `Number`.
- Thời gian phải ở định dạng Unix epoch **tính bằng giây**.
- Tên thuộc tính phân biệt chữ hoa và chữ thường.

Khi `expireAt` nhỏ hơn thời gian hiện tại, item được xem là đã hết hạn. Một tiến trình nền sẽ xóa item sau đó mà không tiêu thụ write throughput cho thao tác TTL delete ban đầu tại Region gốc.

Điều quan trọng là giữa thời điểm hết hạn và thời điểm bị xóa thật sự có một khoảng trễ không được bảo đảm chính xác.

## Vì sao item hết hạn vẫn xuất hiện trong Query hoặc Scan?

Trong thời gian chờ tiến trình nền xóa, item vẫn tồn tại trong table. Do đó:

- `GetItem` vẫn có thể trả về item.
- `Query` và `Scan` vẫn có thể chứa item hết hạn.
- Item vẫn chiếm dung lượng lưu trữ và có thể góp phần vào chi phí đọc.
- Ứng dụng vẫn có thể cập nhật item trước khi nó bị xóa.

TTL chỉ tự động dọn dẹp dữ liệu; nó không tự động làm item trở nên “vô hình” với ứng dụng ngay khi hết hạn.

## Ứng dụng phải tự kiểm tra thời hạn

Nếu dữ liệu không còn giá trị sau một thời điểm nhất định, lớp ứng dụng phải kiểm tra `expireAt`.

Ví dụ:

```python
from time import time


def is_active(item: dict) -> bool:
    expire_at = int(item.get("expireAt", 0))
    return expire_at > int(time())
```

Khi đọc một item:

```python
response = table.get_item(
    Key={
        "stationId": "HCM-Q1",
        "alertType": "PM25_HIGH",
    }
)

item = response.get("Item")

if item and is_active(item):
    use_active_alert(item)
else:
    treat_as_expired()
```

Quy tắc nghiệp vụ nên là:

```text
Item còn hiệu lực = item tồn tại VÀ expireAt > thời gian hiện tại
```

Không nên chỉ kiểm tra `item is not None`.

## Lọc item hết hạn trong Query và Scan

DynamoDB hỗ trợ `FilterExpression` để loại item hết hạn khỏi kết quả trả về:

```python
from time import time
from boto3.dynamodb.conditions import Attr

now = int(time())

response = table.scan(
    FilterExpression=Attr("expireAt").gt(now)
)
```

Tuy nhiên, filter được áp dụng sau khi DynamoDB đã đọc dữ liệu. Điều đó có nghĩa:

- Item hết hạn có thể không xuất hiện trong kết quả cuối.
- Read capacity vẫn có thể bị tiêu thụ cho dữ liệu đã đọc trước khi lọc.
- Nếu số item hết hạn lớn, `FilterExpression` không phải giải pháp tối ưu cho mọi mô hình truy vấn.

Với hệ thống có lưu lượng cao, nên thiết kế partition key, sort key hoặc index phù hợp với truy vấn dữ liệu còn hiệu lực thay vì phụ thuộc hoàn toàn vào Scan và filter.

## TTL phù hợp cho việc gì?

TTL phù hợp với dữ liệu có thể được dọn dẹp bất đồng bộ:

- Session hết hạn.
- Mã OTP hoặc token tạm thời.
- Cache.
- Dữ liệu telemetry chỉ cần lưu trong một khoảng thời gian.
- Bản ghi cooldown chống gửi cảnh báo lặp lại.
- Dữ liệu thử nghiệm không cần giữ lâu dài.

Trong các trường hợp này, ứng dụng vẫn kiểm tra thời gian hết hạn để bảo đảm logic đúng; TTL chịu trách nhiệm dọn dữ liệu cũ về sau.

## TTL không phù hợp cho việc gì?

TTL không nên được dùng như scheduler cho tác vụ cần chạy đúng thời điểm:

- Gửi email lúc 08:00.
- Đóng đơn hàng chính xác lúc hết hạn.
- Hủy quyền truy cập tại một thời điểm bắt buộc.
- Kích hoạt workflow ngay sau 15 phút.
- Gửi cảnh báo khi một item bị xóa.

Nếu công việc phải thực thi theo lịch, có thể sử dụng Amazon EventBridge Scheduler, Step Functions hoặc một cơ chế lập lịch phù hợp. TTL chỉ nên là lớp dọn dẹp dữ liệu.

## Ví dụ thực tế: cooldown cảnh báo AQI

Giả sử hệ thống gửi cảnh báo khi chỉ số PM2.5 vượt ngưỡng. Để tránh gửi thông báo liên tục, backend tạo một item cooldown:

```json
{
  "stationId": "HCM-Q1",
  "alertType": "PM25_HIGH",
  "lastSentAt": 1767222000,
  "expireAt": 1767225600
}
```

Sai lầm thường gặp là:

```text
Nếu item tồn tại → không gửi cảnh báo
```

Sau khi `expireAt` đã qua, item có thể vẫn tồn tại thêm một khoảng thời gian. Nếu chỉ kiểm tra sự tồn tại, hệ thống tiếp tục chặn cảnh báo dù cooldown đã hết.

Logic đúng là:

```text
Nếu item tồn tại và expireAt > now
    → cooldown vẫn còn, không gửi

Nếu item không tồn tại hoặc expireAt <= now
    → cooldown đã hết, có thể gửi cảnh báo mới
```

Sau khi gửi, backend cập nhật `lastSentAt` và `expireAt` bằng một conditional write phù hợp để hạn chế hai request đồng thời cùng gửi cảnh báo.

## Có thể cập nhật item đã hết hạn không?

Có. Trước khi tiến trình TTL xóa item, ứng dụng vẫn có thể cập nhật nó, bao gồm thay `expireAt` bằng một thời điểm trong tương lai hoặc xóa thuộc tính TTL.

Tuy nhiên, tiến trình xóa có thể chạy đồng thời với request cập nhật. AWS khuyến nghị dùng condition expression để bảo đảm item chưa bị thay đổi hoặc xóa ngoài dự kiến.

Ví dụ:

```python
table.update_item(
    Key={
        "stationId": "HCM-Q1",
        "alertType": "PM25_HIGH",
    },
    UpdateExpression="SET lastSentAt = :sent, expireAt = :expires",
    ConditionExpression="attribute_exists(stationId)",
    ExpressionAttributeValues={
        ":sent": now,
        ":expires": now + 3600,
    },
)
```

Condition cụ thể cần được thiết kế theo yêu cầu đồng thời của ứng dụng, không nên dùng một công thức cho mọi trường hợp.

## TTL, DynamoDB Streams và Global Tables

Khi DynamoDB xóa item bằng TTL, sự kiện xóa có thể xuất hiện trong DynamoDB Streams dưới dạng service deletion. Điều này hữu ích cho một số quy trình hậu xử lý, nhưng không nên dùng Streams của TTL để kích hoạt tác vụ cần đúng thời điểm vì bản thân TTL đã có độ trễ.

Với Global Tables phiên bản hiện hành:

- TTL delete ban đầu tại Region nơi item hết hạn không tiêu thụ WCU.
- Thao tác xóa được sao chép sang các replica Region.
- Replicated TTL delete có thể tiêu thụ replicated write capacity và phát sinh chi phí tại các replica.
- TTL deletion chỉ được nhận diện trong DynamoDB Streams tại Region nơi việc xóa TTL diễn ra, không phải tại các Region chỉ nhận bản sao thao tác xóa.

## Những lỗi cấu hình thường gặp

### Dùng mili giây thay vì giây

Nhiều ngôn ngữ trả timestamp theo mili giây. Nếu lưu trực tiếp mà không chia cho 1.000, thời điểm hết hạn có thể bị đẩy rất xa về tương lai.

```python
expire_at = int(time()) + 3600
```

### Lưu timestamp dưới dạng String

TTL yêu cầu giá trị kiểu `Number`. Chuỗi như `"1767225600"` hoặc ngày ISO như `"2026-01-01T00:00:00Z"` không được xử lý như TTL hợp lệ.

### Sai tên thuộc tính

`expireAt`, `ExpireAt` và `expiresAt` là ba tên khác nhau. Tên cấu hình trong table phải khớp chính xác với tên được ghi vào item.

### Cho rằng bật TTL là xóa ngay

TTL cần thời gian để được bật trên toàn bộ table, và item hết hạn được xóa bất đồng bộ. Không nên dùng vài phút đầu để kết luận cấu hình thất bại.

### Quên bật lại TTL sau khi restore

Table được khôi phục từ backup cần được cấu hình TTL lại. Không nên giả định mọi thiết lập TTL đã tự động đi theo table mới.

### Dùng TTL làm lịch thực thi

TTL không cung cấp cam kết xóa tại một giây cụ thể. Tác vụ cần đúng giờ phải dùng dịch vụ lập lịch.

## Kết luận

DynamoDB TTL là cơ chế dọn dữ liệu hết hạn có chi phí thấp, không phải đồng hồ hẹn giờ. Item đã quá `expireAt` vẫn có thể tồn tại và được đọc trong một khoảng thời gian trước khi tiến trình nền xóa.

Để dùng TTL đúng cách:

1. Lưu thời gian dưới dạng Unix epoch theo giây và kiểu `Number`.
2. Kiểm tra `expireAt` trong logic ứng dụng.
3. Lọc item hết hạn khi Query hoặc Scan, đồng thời hiểu chi phí đọc vẫn có thể phát sinh.
4. Dùng conditional write khi cập nhật item có khả năng bị xóa.
5. Dùng scheduler cho công việc cần chạy đúng thời điểm.

Hiểu đúng ranh giới này giúp ứng dụng không phụ thuộc vào thời điểm xóa vật lý và vẫn hoạt động chính xác ngay cả khi item hết hạn còn nằm trong table.

## Tài liệu AWS chính thức

<https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html>

<https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/time-to-live-ttl-how-to.html>

<https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/time-to-live-ttl-before-you-start.html>

<https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ttl-expired-items.html>

<https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Streams.html>

<https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/GlobalTables.html>
