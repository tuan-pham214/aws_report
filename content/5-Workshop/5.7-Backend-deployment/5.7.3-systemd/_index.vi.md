---
title : "Khởi chạy dịch vụ bằng systemd"
weight : 3
chapter : false
pre : " <b> 5.7.3 </b> "
---

#### Kiểm tra dịch vụ

Script bootstrap đã tạo file:

```text
/etc/systemd/system/local-aqi-backend.service
```

Kiểm tra nội dung:

```bash
sudo systemctl cat local-aqi-backend
```

Service sử dụng cấu hình:

```ini
[Unit]
Description=Local AQI FastAPI backend
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
User=ec2-user
Group=ec2-user
WorkingDirectory=/opt/local-aqi-backend
EnvironmentFile=-/opt/local-aqi-backend/.env
ExecStart=/opt/local-aqi-backend/.venv/bin/uvicorn app.main:app --host 0.0.0.0 --port 8000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

#### Khởi động dịch vụ backend

Nạp lại cấu hình `systemd`:

```bash
sudo systemctl daemon-reload
```

Bật service tự khởi động cùng EC2:

```bash
sudo systemctl enable local-aqi-backend
```

Khởi động service:

```bash
sudo systemctl start local-aqi-backend
```

Kiểm tra trạng thái:

```bash
sudo systemctl status local-aqi-backend --no-pager
```

Kết quả mong đợi:

```text
Active: active (running)
```

<!-- Bổ sung ảnh: Service ở trạng thái active (running) -->

#### Kiểm tra log

Hiển thị 100 dòng log gần nhất:

```bash
sudo journalctl -u local-aqi-backend -n 100 --no-pager
```

Theo dõi log theo thời gian thực:

```bash
sudo journalctl -u local-aqi-backend -f
```

Log không được chứa email, credential, access key hoặc payload nhạy cảm.

#### Kiểm tra endpoint trạng thái

Gọi API trực tiếp từ EC2:

```bash
curl -i http://127.0.0.1:8000/health
```

Kết quả mong đợi:

```text
HTTP/1.1 200 OK
```

```json
{"status":"ok"}
```

<!-- Bổ sung ảnh: Kết quả gọi /health trên EC2 -->

Nếu API chưa hoạt động, kiểm tra lại:

```bash
sudo systemctl status local-aqi-backend --no-pager
sudo journalctl -u local-aqi-backend -n 100 --no-pager
sudo ss -lntp | grep 8000
```
