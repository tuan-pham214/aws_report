---
title : "Kiểm tra API và chu kỳ cảnh báo"
weight : 4
chapter : false
pre : " <b> 5.7.4 </b> "
---

#### Kiểm tra dịch vụ backend từ máy cá nhân

Lấy public IPv4 hiện tại của EC2. Địa chỉ này có thể thay đổi khi instance được dừng và khởi động lại.

Tạo biến `BASE_URL`:

```bash
BASE_URL=http://<PUBLIC_IP>:8000
```

Kiểm tra health endpoint:

```bash
curl -i "$BASE_URL/health"
```

Mở Swagger UI:

```text
http://<PUBLIC_IP>:8000/docs
```

<!-- Bổ sung ảnh: Giao diện Swagger UI -->

#### Kiểm tra API dự báo

Gọi API:

```bash
curl -i "$BASE_URL/forecast/station-01"
```

Khi SageMaker Endpoint và dữ liệu trạm đã được cấu hình đúng, API trả về:

- Thời điểm của dữ liệu nguồn.
- Thời điểm dự báo.
- Khoảng dự báo.
- PM2.5 dự báo.
- Chỉ số AQI.

Nếu chưa cấu hình endpoint hoặc dữ liệu nguồn, API trả về HTTP `503` thay vì tạo kết quả dự báo giả.

<!-- Bổ sung ảnh: Kết quả gọi API dự báo -->

#### Kiểm tra đăng ký nhận cảnh báo

Gửi yêu cầu đăng ký:

```bash
curl -i -X POST "$BASE_URL/subscribe/" \
  -H 'Content-Type: application/json' \
  -d '{
    "email": "APPROVED_TEST_EMAIL",
    "station_id": "station-01",
    "threshold_aqi": 150
  }'
```

Amazon SNS gửi email xác nhận. Mở email và chọn **Confirm subscription**.

Chỉ sử dụng địa chỉ đã đồng ý nhận email. Không đưa email thật vào repository, log hoặc ảnh chụp màn hình.

<!-- Bổ sung ảnh: Trạng thái subscription đã che email -->

#### Kiểm tra gửi cảnh báo

Gửi giá trị PM2.5:

```bash
curl -i -X POST "$BASE_URL/alert/" \
  -H 'Content-Type: application/json' \
  -d '{
    "station_id": "station-01",
    "pm25": 55.4
  }'
```

Backend thực hiện:

1. Chuyển đổi PM2.5 sang AQI.
2. Tìm người đăng ký đã xác nhận.
3. Kiểm tra ngưỡng cảnh báo.
4. Kiểm tra thời gian cooldown.
5. Publish thông báo lên Amazon SNS khi đủ điều kiện.

<!-- Bổ sung ảnh: Email cảnh báo đã che địa chỉ người nhận -->

#### Cấu hình chu kỳ tự động

Chỉ bật timer khi SageMaker Endpoint, dữ liệu trạm và IAM permission đã được kiểm tra.

Cài đặt service và timer:

```bash
cd /opt/local-aqi-backend

sudo install -m 0644 deploy/local-aqi-forecast-cycle.service \
  /etc/systemd/system/

sudo install -m 0644 deploy/local-aqi-forecast-cycle.timer \
  /etc/systemd/system/

sudo systemctl daemon-reload
sudo systemctl enable --now local-aqi-forecast-cycle.timer
```

Kiểm tra lịch chạy:

```bash
sudo systemctl list-timers local-aqi-forecast-cycle.timer
sudo systemctl status local-aqi-forecast-cycle.timer --no-pager
```

Kiểm tra lần chạy gần nhất:

```bash
sudo systemctl status local-aqi-forecast-cycle.service --no-pager
sudo journalctl -u local-aqi-forecast-cycle.service -n 100 --no-pager
```

<!-- Bổ sung ảnh: systemd timer và lần chạy gần nhất -->

#### Hoàn thành

Bạn đã hoàn thành việc:

- Triển khai FastAPI Backend trên Amazon EC2.
- Cho phép Backend truy cập DynamoDB, SageMaker và SNS bằng IAM role.
- Khởi chạy ứng dụng bằng `systemd`.
- Kiểm tra các API health, forecast, subscribe và alert.
- Cấu hình chu kỳ dự báo và cảnh báo tự động.

Khi không cần chạy demo, dừng EC2 và SageMaker Endpoint để hạn chế chi phí phát sinh.
