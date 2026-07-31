---
title: "Học máy: Dự báo PM2.5 bằng Amazon SageMaker DeepAR"
date: 2026-07-31
weight: 6
chapter: false
pre: " <b> 5.6. </b> "
---

**Dự án:** Local AQI Forecasting & Alert System  
**Vai trò:** Kỹ sư học máy  
**Người phụ trách:** `duy-tuong`  
**Mô-đun:** `ml`  
**Môi trường:** `dev`  
**Region:** `ap-southeast-1`  
**Thời gian:** 4 tuần

## 1. Tổng quan

### 1.1. Bài toán

Nồng độ bụi mịn PM2.5 thay đổi đáng kể theo vị trí và thời điểm. Người cao tuổi, trẻ em và người mắc bệnh hô hấp thường không có cảnh báo đủ sớm để chủ động phòng tránh. Mô-đun học máy sử dụng lịch sử cảm biến gần nhất của từng trạm để **dự báo PM2.5 trong 48 giờ tiếp theo**, từ đó hỗ trợ hệ thống gửi cảnh báo sớm.

### 1.2. Phương pháp

Nhóm sử dụng thuật toán **DeepAR tích hợp sẵn trong Amazon SageMaker**. Đây là mô hình học sâu có giám sát cho bài toán dự báo chuỗi thời gian xác suất, sử dụng mạng LSTM xếp chồng.

DeepAR được chọn vì:

- Huấn luyện một mô hình chung cho nhiều trạm, đồng thời vẫn giữ được đặc trưng riêng của từng trạm.
- Học được nhiều dạng mùa vụ như chu kỳ theo giờ và khác biệt giữa ngày thường với cuối tuần.
- Trả về giá trị trung bình cùng các phân vị dự báo, phù hợp với cơ chế cảnh báo theo ngưỡng của backend.

### 1.3. Vị trí trong kiến trúc

```text
IoT Stations
-> IoT Core / MQTT
-> Kinesis Data Firehose
-> S3 Raw
-> S3 Processed / ML Ready
-> SageMaker DeepAR
-> SageMaker Endpoint
-> FastAPI Backend
-> SNS Alerts
```

### 1.4. Quy ước tài nguyên

| Tài nguyên | Tên hoặc vị trí |
|---|---|
| Dữ liệu đầu vào | `s3://local-aqi-dev-s3-raw/raw/parquet/` |
| Dữ liệu DeepAR | `s3://local-aqi-dev-s3-raw/processed/deepar/` |
| Đầu ra mô hình | `s3://local-aqi-dev-s3-raw/models/deepar/` |
| Training Job | `aqi-deepar-on-demand-<timestamp>` |
| Endpoint | `aqi-endpoint-test` |
| Máy huấn luyện | `ml.m5.large` |
| Máy suy luận | `ml.t2.medium` |

Các thẻ bắt buộc:

```text
Project     = local-aqi-forecasting
Environment = dev
Owner       = duy-tuong
Module      = ml
```

## 2. Tuần 1 — Khảo sát và chuẩn bị dữ liệu

### 2.1. Mục tiêu

Thiết lập môi trường phát triển, khảo sát đặc tính thống kê và quy luật thời gian của dữ liệu, sau đó tạo bộ dữ liệu sạch theo định dạng JSON Lines mà DeepAR yêu cầu.

### 2.2. Thiết lập môi trường

Các thư viện chính:

```bash
pip install pandas numpy matplotlib seaborn pyarrow boto3 sagemaker
```

### 2.3. Bộ dữ liệu

Dữ liệu gồm nhiều trạm quan trắc với các trường chính:

| Trường | Ý nghĩa |
|---|---|
| `station_id` | Mã trạm |
| `timestamp` | Thời điểm đo |
| `pm2_5` | Nồng độ PM2.5 |
| `pm10` | Nồng độ PM10 |
| `temperature` | Nhiệt độ |
| `humidity` | Độ ẩm |

### 2.4. Quy trình làm sạch

Các bước xử lý:

1. Chuyển thời gian UTC sang `Asia/Ho_Chi_Minh` và loại bỏ thông tin múi giờ sau khi chuẩn hóa.
2. Lọc các trạm về cùng khoảng thời gian.
3. Sắp xếp dữ liệu theo trạm và thời gian.
4. Tạo trục thời gian liên tục theo giờ.
5. Nội suy các khoảng thiếu ngắn.
6. Loại bỏ bản ghi trùng và giá trị không hợp lệ.

Ví dụ:

```python
df["timestamp"] = pd.to_datetime(df["timestamp"], utc=True)
df["timestamp"] = df["timestamp"].dt.tz_convert("Asia/Ho_Chi_Minh")
df = df.sort_values(["station_id", "timestamp"])
```

### 2.5. Kết quả khảo sát

- PM2.5 và PM10 có phân phối lệch phải, xuất hiện một số thời điểm ô nhiễm tăng cao.
- Dữ liệu thể hiện chu kỳ theo giờ trong ngày.
- Các trạm có mức nền và biên độ dao động khác nhau.
- Nhiệt độ và độ ẩm có thể hỗ trợ mô hình dưới dạng đặc trưng động.

### 2.6. Chuyển sang JSON Lines của DeepAR

Mỗi dòng biểu diễn một chuỗi thời gian:

```json
{
  "start": "2026-01-01 00:00:00",
  "target": [18.2, 19.1, 20.4],
  "cat": [0],
  "dynamic_feat": [[0, 1, 2], [28.1, 27.8, 27.5]]
}
```

Kết quả của tuần 1 gồm bộ dữ liệu sạch, tệp JSONL cho DeepAR, báo cáo khảo sát và dữ liệu đã tải lên S3.

## 3. Tuần 2 — Huấn luyện mô hình cơ sở cục bộ

### 3.1. Mục tiêu

Tạo một mô hình cơ sở bằng GluonTS để kiểm tra cấu trúc dữ liệu, chiến lược chia tập và chất lượng dự báo trước khi chạy SageMaker.

### 3.2. Chia tập theo thời gian

Dữ liệu chuỗi thời gian không được chia ngẫu nhiên vì sẽ làm rò rỉ thông tin tương lai vào tập huấn luyện.

```text
Train      -> phần dữ liệu sớm nhất
Validation -> giai đoạn tiếp theo
Test       -> giai đoạn mới nhất
```

### 3.3. Cấu hình mô hình cơ sở

| Tham số | Giá trị |
|---|---|
| Tần suất | `1H` |
| Độ dài dự báo | `48` giờ |
| Số epoch | `15` |
| Ngữ cảnh | `168` giờ |
| Kiểu phân phối | Student-t |

### 3.4. Kết quả mô hình cơ sở

Mô hình cục bộ đạt sMAPE khoảng `5.87%`. Kết quả này xác nhận pipeline dữ liệu và cấu hình DeepAR hoạt động trước khi chuyển sang SageMaker.

## 4. Tuần 3 — Huấn luyện và triển khai trên SageMaker

### 4.1. Tải dữ liệu huấn luyện lên S3

```python
import boto3

s3 = boto3.client("s3")
s3.upload_file(
    "deepar_train.jsonl",
    "local-aqi-dev-s3-raw",
    "processed/deepar/deepar_train.jsonl"
)
```

### 4.2. Khởi tạo SageMaker Session

```python
import sagemaker
from sagemaker import get_execution_role

session = sagemaker.Session()
role = get_execution_role()
region = session.boto_region_name
```

### 4.3. Cấu hình siêu tham số

```python
hyperparameters = {
    "time_freq": "H",
    "prediction_length": "48",
    "context_length": "168",
    "epochs": "50",
    "mini_batch_size": "32",
    "learning_rate": "0.001",
    "num_layers": "2",
    "num_cells": "40"
}
```

### 4.4. Chạy Training Job

```python
image_uri = sagemaker.image_uris.retrieve("forecasting-deepar", region)

estimator = sagemaker.estimator.Estimator(
    image_uri=image_uri,
    role=role,
    instance_count=1,
    instance_type="ml.m5.large",
    output_path="s3://local-aqi-dev-s3-raw/models/deepar/",
    sagemaker_session=session
)

estimator.set_hyperparameters(**hyperparameters)
estimator.fit({"train": train_s3_uri})
```

### 4.5. Triển khai Endpoint và kiểm tra suy luận

```python
predictor = estimator.deploy(
    initial_instance_count=1,
    instance_type="ml.t2.medium",
    endpoint_name="aqi-endpoint-test"
)
```

Payload suy luận:

```json
{
  "instances": [
    {
      "start": "2026-07-01 00:00:00",
      "target": [18.2, 19.1, 20.4]
    }
  ],
  "configuration": {
    "num_samples": 100,
    "output_types": ["mean", "quantiles"],
    "quantiles": ["0.1", "0.5", "0.9"]
  }
}
```

### 4.6. Kết quả đánh giá cuối cùng

| Chỉ số | Giá trị | Đánh giá |
|---|---:|---|
| MAE | **0.191 µg/m³** | Sai số trung bình thấp |
| RMSE | **0.201 µg/m³** | Không có sai lệch lớn đáng kể |
| R² | **0.999** | Giải thích 99.9% biến thiên PM2.5 |
| MAPE | **1.441%** | Thấp hơn nhiều so với ngưỡng 15% |

![Biểu đồ đánh giá mô hình DeepAR trên SageMaker](/images/5-Workshop/5.6-Machine-learning/deepar_sagemaker_evaluation.png)

### 4.7. Dọn Endpoint sau khi kiểm thử

```python
predictor.delete_endpoint()
```

Endpoint phải được xóa ngay sau khi thu thập đủ minh chứng để tránh phát sinh chi phí liên tục.

## 5. Tuần 4 — Tích hợp và hoàn thiện tài liệu

### 5.1. Hợp đồng API cho backend

Backend gửi mã trạm và chuỗi dữ liệu gần nhất tới SageMaker Endpoint. Phản hồi chứa giá trị trung bình cùng các phân vị dự báo trong 48 giờ.

```json
{
  "station_id": "station-01",
  "forecast_horizon_hours": 48,
  "predictions": {
    "mean": [24.1, 25.3],
    "p10": [20.2, 21.1],
    "p50": [24.0, 25.2],
    "p90": [29.3, 30.4]
  }
}
```

Backend so sánh dự báo với ngưỡng PM2.5 của từng trạm. Nếu có giá trị vượt ngưỡng, hệ thống gửi cảnh báo qua Amazon SNS.

### 5.2. Khuyến nghị giám sát

- Theo dõi trạng thái Training Job và Endpoint trong CloudWatch.
- Ghi log thời gian suy luận và lỗi gọi Endpoint.
- Cảnh báo khi sai số đầu vào, độ trễ hoặc chi phí vượt ngưỡng.
- Gắn thẻ đầy đủ để phân bổ chi phí theo dự án và mô-đun.

### 5.3. Chi phí ước tính

Chi phí lớn nhất đến từ thời gian chạy `ml.m5.large` khi huấn luyện và `ml.t2.medium` khi Endpoint hoạt động. Vì tần suất gọi theo giờ, nhóm nên xem xét SageMaker Serverless Inference hoặc xử lý theo lô ở giai đoạn tiếp theo.

## 6. Tổng kết

| Tuần | Đầu ra chính | Cột mốc |
|---|---|---|
| **1** | Khảo sát dữ liệu, tạo JSONL và tải lên S3 | Xác nhận chất lượng dữ liệu |
| **2** | DeepAR cơ sở, 15 epoch trên CPU | sMAPE 5.87% |
| **3** | SageMaker Training 50 epoch và Endpoint | RMSE 0.201, R² 0.999 |
| **4** | Báo cáo, hợp đồng API và chuẩn bị trình diễn | Sẵn sàng tích hợp đầu cuối |

Các tệp đầu ra chính gồm notebook khảo sát, notebook huấn luyện, tệp JSONL, mô hình cục bộ, tệp metric, biểu đồ đánh giá và tài liệu tích hợp backend.

## 7. Hướng phát triển

- Bổ sung đặc trưng trễ tại 24 giờ và 168 giờ.
- Tinh chỉnh riêng cho trạm có mức ô nhiễm phức tạp.
- Chuyển sang SageMaker Serverless Inference để giảm chi phí nhàn rỗi.
- Lập lịch SageMaker Pipeline để tái huấn luyện hàng tháng bằng dữ liệu mới từ Firehose.
