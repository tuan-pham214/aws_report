---
title : "Cài đặt và cấu hình dịch vụ backend"
weight : 2
chapter : false
pre : " <b> 5.7.2 </b> "
---

#### Kết nối đến EC2

1. Mở dịch vụ **Amazon EC2**.
2. Chọn instance `local-aqi-dev-ec2-backend`.
3. Chọn **Connect**.
4. Chọn tab **Session Manager**.
5. Chọn **Connect**.

<!-- Bổ sung ảnh: Phiên kết nối Session Manager -->

Kiểm tra phiên bản Python:

```bash
python3 --version
```

Cài đặt các công cụ cần thiết:

```bash
sudo dnf install -y python3-pip git rsync
```

#### Tải mã nguồn

Di chuyển tới thư mục của người dùng EC2:

```bash
cd /home/ec2-user
```

Tải repository:

```bash
git clone <REPOSITORY_URL> AWS-FCJ-local_aqi_forecast
cd AWS-FCJ-local_aqi_forecast
```

Chạy script bootstrap để tạo thư mục ứng dụng và service `systemd`:

```bash
sudo bash backend/deploy/ec2-user-data.sh
```

Đồng bộ mã nguồn Backend vào `/opt/local-aqi-backend`:

```bash
sudo rsync -a --delete \
  --exclude '.env' \
  --exclude '.venv' \
  backend/ /opt/local-aqi-backend/

sudo chown -R ec2-user:ec2-user /opt/local-aqi-backend
```

<!-- Bổ sung ảnh: Cấu trúc thư mục /opt/local-aqi-backend -->

#### Cài đặt thư viện phụ thuộc

Di chuyển tới thư mục Backend:

```bash
cd /opt/local-aqi-backend
```

Tạo virtual environment:

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Cài đặt dependency:

```bash
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

#### Cấu hình biến môi trường

Tạo file `.env`:

```bash
cp .env.example .env
chmod 600 .env
```

Mở file:

```bash
nano .env
```

Cập nhật nội dung:

```dotenv
AWS_REGION=ap-southeast-1
SUBSCRIBERS_TABLE=local-aqi-subscribers-dev
ALERTS_TOPIC_ARN=arn:aws:sns:ap-southeast-1:ACCOUNT_ID:local-aqi-alerts-dev

SAGEMAKER_ENDPOINT_NAME=ENDPOINT_NAME
SAGEMAKER_CONTENT_TYPE=application/json
SAGEMAKER_TIMEOUT_SECONDS=5
SAGEMAKER_MAX_ATTEMPTS=3
FORECAST_HORIZON_HOURS=24

STATION_DATA_FILE=/opt/local-aqi-backend/runtime/stations.json
ALLOW_SAMPLE_STATION_DATA=false
ALERT_COOLDOWN_SECONDS=3600
```

Thay `ACCOUNT_ID` và `ENDPOINT_NAME` bằng giá trị thực tế.

Chỉ cấu hình `SAGEMAKER_ENDPOINT_NAME` khi endpoint ở trạng thái `InService` và định dạng request/response đã được thống nhất với nhóm ML.

Không lưu AWS Access Key, Secret Access Key hoặc email cá nhân trong file `.env`.

<!-- Bổ sung ảnh: File cấu hình đã che Account ID và ARN -->

#### Kiểm tra mã nguồn

Chạy bộ kiểm thử:

```bash
python -m unittest discover -s tests -v
```

Kiểm tra cú pháp:

```bash
python -m compileall -q app tests
```

Kiểm tra tài liệu OpenAPI:

```bash
python scripts/validate_openapi.py
```

Nếu các câu lệnh đều hoàn thành thành công, chuyển sang bước khởi chạy Backend.

<!-- Bổ sung ảnh: Kết quả chạy test và OpenAPI validation -->
