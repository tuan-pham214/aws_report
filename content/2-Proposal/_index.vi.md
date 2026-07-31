---
title: "Bản đề xuất"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
{{% notice warning %}}
⚠️ **Lưu ý:** Các thông tin dưới đây trình bày phạm vi và kế hoạch dự kiến của dự án. Kiến trúc, chi phí và các thông số kỹ thuật có thể tiếp tục được điều chỉnh trong quá trình triển khai và kiểm thử.
{{% /notice %}}

Phần này tóm tắt đề tài nhóm dự kiến triển khai trong thời gian thực tập, bao gồm bối cảnh bài toán, kiến trúc hệ thống, lộ trình thực hiện, ngân sách và rủi ro chính.

# Local AQI Forecasting & Alert System
## Hệ thống dự báo và cảnh báo ô nhiễm không khí cục bộ trên AWS

### 1. Tóm tắt điều hành

Local AQI Forecasting & Alert System là hệ thống thu thập, lưu trữ, xử lý và dự báo chất lượng không khí theo từng trạm quan trắc. Dự án tập trung vào chỉ số PM2.5 trong giai đoạn MVP, đồng thời hướng tới khả năng mở rộng thêm PM10 và các yếu tố môi trường khác trong tương lai.

Trong phiên bản đầu tiên, nhóm sử dụng dữ liệu lịch sử từ OpenAQ để xây dựng chương trình mô phỏng nhiều trạm quan trắc. Dữ liệu telemetry được gửi qua MQTT đến AWS IoT Core, đi qua Amazon Data Firehose và được lưu trữ trên Amazon S3 theo mô hình data lake.

Dữ liệu sau đó được làm sạch và chuẩn hóa bằng Amazon SageMaker Processing. Mô hình dự báo chuỗi thời gian được huấn luyện và triển khai trên Amazon SageMaker. Một backend FastAPI chạy trên Amazon EC2 cung cấp API truy vấn kết quả dự báo và kích hoạt Amazon SNS để gửi email cảnh báo khi giá trị PM2.5 dự báo vượt ngưỡng an toàn.

Phiên bản MVP dự kiến hỗ trợ 3 trạm mô phỏng, dự báo PM2.5 trong 24 giờ tiếp theo và gửi cảnh báo qua email. Kiến trúc được thiết kế theo hướng có thể mở rộng thêm số lượng trạm, chỉ số môi trường và kênh thông báo ở các giai đoạn sau.

### 2. Tuyên bố vấn đề

#### Vấn đề hiện tại

Chất lượng không khí tại các đô thị lớn có thể thay đổi rõ rệt theo khu vực và theo thời điểm. Một chỉ số AQI tổng quát cho toàn thành phố thường không phản ánh chính xác mức độ ô nhiễm tại từng quận, khu dân cư hoặc cơ sở giáo dục.

Những nhóm nhạy cảm như người cao tuổi, trẻ em, người mắc bệnh hô hấp hoặc người thường xuyên hoạt động ngoài trời thường không có đủ thông tin dự báo sớm để chủ động phòng tránh trước khi ô nhiễm tăng cao.

Phần lớn các hệ thống hiện có chủ yếu cung cấp dữ liệu hiện tại hoặc dữ liệu lịch sử. Khả năng dự báo ngắn hạn và cảnh báo theo từng trạm quan trắc cụ thể vẫn còn hạn chế.

#### Giải pháp đề xuất

Hệ thống xây dựng một pipeline dữ liệu hoàn chỉnh theo luồng:

```text
Telemetry Simulator
-> AWS IoT Core
-> IoT Rule
-> Amazon Data Firehose
-> Amazon S3 Raw
-> SageMaker Processing
-> Amazon S3 Processed
-> SageMaker Training
-> SageMaker Endpoint
-> FastAPI on EC2
-> Amazon SNS Email
```

Các chức năng chính của hệ thống gồm:

* Mô phỏng dữ liệu từ nhiều trạm quan trắc.
* Tiếp nhận dữ liệu thời gian thực bằng MQTT.
* Lưu trữ dữ liệu thô và dữ liệu đã xử lý trên Amazon S3.
* Làm sạch và chuẩn hóa dữ liệu phục vụ mô hình Machine Learning.
* Dự báo PM2.5 trong 24 giờ tiếp theo cho từng trạm.
* Cung cấp kết quả dự báo qua REST API.
* Gửi email cảnh báo khi dự báo vượt ngưỡng an toàn.
* Theo dõi quá trình ingestion, processing và alerting bằng Amazon CloudWatch.

#### Tính ứng dụng thực tế

Các trường học, cơ sở y tế hoặc đơn vị quản lý địa phương có thể đăng ký nhận cảnh báo từ hệ thống. Khi hệ thống dự báo PM2.5 tại một khu vực sẽ vượt ngưỡng trong vài giờ tiếp theo, người dùng có thể chủ động:

* Hạn chế hoạt động ngoài trời.
* Điều chỉnh lịch học hoặc lịch sinh hoạt ngoài trời.
* Đóng cửa sổ hoặc bật thiết bị lọc không khí.
* Chuẩn bị biện pháp bảo vệ cho người thuộc nhóm nhạy cảm.
* Theo dõi sớm nguy cơ ô nhiễm thay vì chỉ phản ứng khi ô nhiễm đã xảy ra.

#### Lợi ích dự kiến

* Cung cấp cảnh báo sớm thay vì chỉ hiển thị dữ liệu hiện tại.
* Tạo pipeline dữ liệu tập trung, có thể kiểm chứng và tái sử dụng.
* Giảm thao tác thu thập và tổng hợp dữ liệu thủ công.
* Tạo nền tảng cho các nghiên cứu tiếp theo về dự báo chất lượng không khí.
* Có thể mở rộng sang PM10, nhiệt độ, độ ẩm, gió và các chỉ số môi trường khác.
* Có thể tích hợp thêm dashboard, web app hoặc push notification trong các phiên bản sau.

### 3. Kiến trúc giải pháp

Hệ thống được thiết kế theo hướng event-driven kết hợp data pipeline. Mỗi thành phần đảm nhiệm một trách nhiệm riêng, giúp việc triển khai, kiểm thử và mở rộng rõ ràng hơn.

#### Luồng xử lý tổng thể

1. Python Simulator đọc dữ liệu lịch sử từ OpenAQ và mô phỏng dữ liệu của nhiều trạm.
2. Simulator gửi telemetry message đến AWS IoT Core qua MQTT.
3. AWS IoT Rule tiếp nhận message theo topic đã cấu hình.
4. IoT Rule chuyển message sang Amazon Data Firehose.
5. Firehose gom record và ghi dữ liệu xuống Amazon S3 Raw.
6. SageMaker Processing đọc dữ liệu raw, kiểm tra schema, làm sạch và tạo dữ liệu processed.
7. SageMaker Training sử dụng dữ liệu processed để huấn luyện mô hình dự báo chuỗi thời gian.
8. Mô hình được triển khai thành SageMaker Endpoint.
9. FastAPI trên Amazon EC2 gọi endpoint để lấy kết quả dự báo.
10. Khi giá trị dự báo vượt ngưỡng an toàn, backend gửi thông báo qua Amazon SNS.
11. Amazon CloudWatch thu thập metrics và logs để phục vụ giám sát và xử lý lỗi.

#### Dịch vụ AWS sử dụng

* **AWS IoT Core**: Tiếp nhận dữ liệu telemetry từ các trạm mô phỏng qua MQTT.
* **AWS IoT Rules Engine**: Lọc và định tuyến message từ IoT Core sang Firehose.
* **Amazon Data Firehose**: Gom cụm dữ liệu và chuyển dữ liệu liên tục vào Amazon S3.
* **Amazon S3 Raw**: Lưu dữ liệu telemetry nguyên bản.
* **Amazon S3 Processed**: Lưu dữ liệu đã làm sạch và chuẩn hóa.
* **Amazon SageMaker Processing**: Làm sạch, kiểm tra và chuyển đổi dữ liệu.
* **Amazon SageMaker Training**: Huấn luyện mô hình dự báo chuỗi thời gian.
* **Amazon SageMaker Endpoint**: Cung cấp khả năng suy luận mô hình qua API.
* **Amazon EC2**: Chạy backend FastAPI và logic cảnh báo.
* **Amazon SNS**: Gửi email cảnh báo tới người dùng đăng ký.
* **Amazon CloudWatch**: Theo dõi metrics, logs và lỗi vận hành.
* **AWS IAM**: Quản lý quyền truy cập giữa người dùng và các dịch vụ AWS.
* **AWS Budgets**: Theo dõi chi phí và gửi cảnh báo khi mức sử dụng vượt ngưỡng.

#### Thiết kế thành phần

##### Nguồn dữ liệu và Simulator

Nguồn dữ liệu chính là OpenAQ Dataset. Dữ liệu lịch sử được làm sạch và đưa vào Python Simulator để mô phỏng hoạt động của các trạm quan trắc.

Phiên bản MVP sử dụng 3 trạm mô phỏng. Mỗi message dự kiến chứa các trường chính:

```json
{
  "schema_version": "1.0",
  "station_id": "station-01",
  "ts_utc": "2026-07-31T00:00:00Z",
  "pm25_ugm3": 28.5,
  "pm10_ugm3": 42.1,
  "temperature_c": 31.2,
  "humidity_pct": 72.5,
  "source": "simulator"
}
```

Các trường mở rộng có thể bao gồm:

```text
wind_speed_mps
wind_dir_deg
latitude
longitude
```

##### Tiếp nhận và định tuyến dữ liệu

Simulator kết nối AWS IoT Core bằng certificate và publish message lên MQTT topic của môi trường phát triển.

Topic convention dự kiến:

```text
telemetry/aqi/dev/{station_id}
```

IoT Rule sử dụng câu lệnh:

```sql
SELECT * FROM 'telemetry/aqi/dev/+'
```

Sau khi nhận message, IoT Rule sử dụng IAM service role để gửi record sang Firehose.

##### Lưu trữ dữ liệu

Hệ thống sử dụng hai S3 bucket chính:

```text
local-aqi-dev-s3-raw
local-aqi-dev-s3-processed
```

Dữ liệu raw được lưu theo prefix:

```text
raw/telemetry/
```

Dữ liệu lỗi có thể được lưu tại:

```text
raw/errors/
```

Dữ liệu đã xử lý được lưu trong bucket processed theo cấu trúc thời gian hoặc station để thuận tiện cho huấn luyện và truy vấn.

##### Machine Learning

SageMaker Processing thực hiện:

* Đọc dữ liệu từ S3 Raw.
* Kiểm tra schema.
* Xử lý giá trị thiếu.
* Loại bỏ hoặc đánh dấu dữ liệu bất thường.
* Chuẩn hóa timestamp.
* Sắp xếp dữ liệu theo station và thời gian.
* Tạo dữ liệu train, validation và test.
* Ghi dữ liệu đã xử lý xuống S3 Processed.

Mô hình dự kiến sử dụng DeepAR hoặc một mô hình dự báo chuỗi thời gian phù hợp khác trên SageMaker.

Đầu ra chính của mô hình gồm:

* Dự báo PM2.5 trong 24 giờ tiếp theo.
* Giá trị dự báo theo từng timestamp.
* Khoảng bất định hoặc quantile dự báo nếu mô hình hỗ trợ.
* Trạng thái cảnh báo theo ngưỡng đã cấu hình.

##### Backend và cảnh báo

FastAPI được triển khai trên Amazon EC2 và cung cấp các endpoint dự kiến:

```text
GET /health
GET /stations
GET /forecast/{station_id}
POST /subscriptions
```

Backend gọi SageMaker Endpoint hoặc đọc kết quả forecast đã lưu. Khi giá trị dự báo vượt ngưỡng, backend kích hoạt Amazon SNS để gửi email cho người dùng đã đăng ký.

### 4. Triển khai kỹ thuật

#### Các giai đoạn triển khai

Dự án được triển khai trong 4 tuần với các giai đoạn chính sau:

##### Giai đoạn 1 - Thiết kế và chuẩn hóa

* Chốt phạm vi MVP.
* Chọn PM2.5 làm chỉ số dự báo chính.
* Chọn thời gian dự báo 24 giờ.
* Chọn 3 trạm cho phiên bản đầu tiên.
* Chuẩn hóa telemetry schema.
* Chốt naming convention và tagging convention.
* Thiết kế kiến trúc AWS.
* Tạo IAM users, IAM roles và budget alerts.

##### Giai đoạn 2 - Data Ingestion

* Chuẩn bị dữ liệu OpenAQ.
* Xây dựng Python Simulator.
* Tạo certificate và IoT Policy.
* Cấu hình MQTT topic.
* Tạo IoT Rule.
* Kết nối IoT Core với Firehose.
* Cấu hình Firehose ghi dữ liệu vào S3 Raw.
* Kiểm tra nhiều message và nhiều station.
* Thu thập evidence cho toàn bộ ingestion pipeline.

Milestone của giai đoạn:

```text
Simulator -> IoT Core -> Firehose -> S3 Raw
```

Dữ liệu JSON thực tế phải xuất hiện trong bucket raw.

##### Giai đoạn 3 - Data Processing và Machine Learning

* Xây dựng processing script.
* Làm sạch và chuẩn hóa dữ liệu.
* Tạo dữ liệu train, validation và test.
* Huấn luyện mô hình dự báo.
* Đánh giá kết quả bằng các chỉ số như MAE và RMSE.
* So sánh với baseline đơn giản.
* Lưu model artifact.
* Triển khai SageMaker Endpoint.
* Kiểm thử inference theo từng station.

##### Giai đoạn 4 - API, cảnh báo và kiểm thử end-to-end

* Xây dựng FastAPI backend.
* Triển khai backend trên EC2.
* Kết nối backend với SageMaker Endpoint.
* Tạo SNS topic và email subscription.
* Xây dựng logic so sánh forecast với ngưỡng cảnh báo.
* Kiểm thử API.
* Kiểm thử gửi email.
* Kiểm thử toàn bộ hệ thống từ simulator đến người nhận cảnh báo.
* Hoàn thiện tài liệu và evidence triển khai.

#### Yêu cầu kỹ thuật

##### Môi trường phát triển

* Python 3.10 hoặc phiên bản tương thích.
* Git và GitHub.
* AWS CLI.
* Visual Studio Code.
* Python virtual environment.
* Paho MQTT.
* Pandas.
* Boto3.
* FastAPI.
* Uvicorn.
* SageMaker Python SDK.

Ví dụ dependency cơ bản:

```bash
pip install paho-mqtt pandas boto3 fastapi uvicorn sagemaker
```

##### Cấu hình bảo mật

* Không sử dụng tài khoản root cho công việc hằng ngày.
* Mỗi thành viên sử dụng IAM user riêng.
* Bật MFA cho tài khoản có quyền quản trị.
* Áp dụng nguyên tắc least privilege.
* Private key và certificate không được commit lên Git.
* Thư mục `certs/` và file `.env` phải nằm trong `.gitignore`.
* IAM role của IoT Core chỉ được gửi dữ liệu vào Firehose stream được chỉ định.
* IAM role của Firehose chỉ được ghi vào S3 bucket và prefix cần thiết.
* SageMaker execution role chỉ được truy cập các bucket phục vụ processing và training.

##### Naming convention

Tên tài nguyên sử dụng cấu trúc:

```text
local-aqi-{environment}-{service-or-purpose}
```

Ví dụ:

```text
local-aqi-dev-s3-raw
local-aqi-dev-s3-processed
local-aqi-dev-firehose-telemetry
local-aqi-dev-iot-to-firehose-role
local-aqi-dev-firehose-to-s3-role
local-aqi-dev-sagemaker-execution-role
```

##### Tagging convention

Các tài nguyên hỗ trợ tagging sử dụng các tag:

```text
Project     = local-aqi-forecasting
Environment = dev
Owner       = team-member-name
Module      = ingestion | data | ml | backend | devops
```

### 5. Lộ trình và mốc triển khai

Dự án dự kiến triển khai trong khoảng 4 tuần.

#### Tuần 1 - Kiến trúc và Ingestion

* Chốt kiến trúc phiên bản 1.
* Chốt schema telemetry.
* Tạo S3 Raw và S3 Processed.
* Xây dựng simulator.
* Cấu hình AWS IoT Core.
* Cấu hình Firehose.
* Hoàn thành IoT Rule -> Firehose -> S3 Raw.

**Milestone 1**

```text
Dữ liệu thực từ simulator xuất hiện trong S3 Raw.
```

#### Tuần 2 - Data Processing

* Kiểm tra chất lượng dữ liệu.
* Xây dựng processing pipeline.
* Chuẩn hóa dữ liệu theo từng station.
* Tạo tập train, validation và test.
* Lưu dữ liệu sạch vào S3 Processed.

**Milestone 2**

```text
Dữ liệu processed có thể sử dụng để huấn luyện mô hình.
```

#### Tuần 3 - Machine Learning

* Xây dựng baseline.
* Huấn luyện mô hình DeepAR hoặc mô hình phù hợp khác.
* Đánh giá MAE và RMSE.
* Kiểm tra dự báo theo từng station.
* Triển khai SageMaker Endpoint.

**Milestone 3**

```text
SageMaker Endpoint trả về dự báo PM2.5 trong 24 giờ.
```

#### Tuần 4 - Backend và cảnh báo

* Xây dựng FastAPI.
* Triển khai trên EC2.
* Kết nối API với SageMaker Endpoint.
* Cấu hình SNS.
* Kiểm thử email cảnh báo.
* Kiểm thử end-to-end.
* Hoàn thiện tài liệu, hình ảnh và báo cáo.

**Milestone 4**

```text
Người dùng nhận email khi dự báo PM2.5 vượt ngưỡng.
```

#### Hướng phát triển sau MVP

* Mở rộng dự báo từ 24 giờ lên 48 giờ.
* Tăng số lượng trạm.
* Bổ sung PM10, nhiệt độ, độ ẩm và gió.
* Xây dựng dashboard trực quan.
* Thêm push notification hoặc SMS.
* Tự động hóa retraining.
* Theo dõi model drift và data drift.
* Tích hợp nguồn dữ liệu quan trắc thực tế.

### 6. Ước tính ngân sách

Chi phí chính của dự án đến từ:

* Amazon SageMaker Processing.
* SageMaker Training.
* SageMaker Endpoint.
* Amazon EC2.
* Amazon S3.
* Amazon Data Firehose.
* AWS IoT Core.
* Amazon SNS.
* Amazon CloudWatch.

Do khối lượng dữ liệu của MVP tương đối nhỏ, chi phí của IoT Core, Firehose, S3 và SNS dự kiến không lớn. Thành phần cần kiểm soát chặt nhất là SageMaker Endpoint, SageMaker Processing, Training Jobs và EC2 vì các tài nguyên này có thể tiếp tục phát sinh chi phí nếu không được dừng đúng lúc.

Chi phí chính xác cần được tính lại bằng AWS Pricing Calculator dựa trên:

* Region `ap-southeast-1`.
* Số lượng message mỗi tháng.
* Kích thước mỗi telemetry record.
* Dung lượng lưu trữ S3.
* Số lần chạy Processing Job.
* Thời gian Training Job.
* Loại instance SageMaker.
* Thời gian hoạt động của SageMaker Endpoint.
* Loại EC2 instance và thời gian chạy.
* Số lượng email gửi qua SNS.

#### Nguyên tắc kiểm soát ngân sách

Nhóm sử dụng ngân sách AWS tối đa 100 USD cho giai đoạn MVP và áp dụng các ngưỡng kiểm soát:

* **10 USD hoặc 5% ngân sách**: Kiểm tra dịch vụ đang phát sinh chi phí.
* **25 USD hoặc 15% ngân sách**: Đánh giá lại tài nguyên và thời gian chạy.
* **50 USD hoặc 30% ngân sách**: Tạm dừng các tài nguyên không thiết yếu.
* **100 USD hoặc 50% credit khả dụng**: Đóng băng phạm vi, chỉ giữ các thành phần MVP.

#### Biện pháp tối ưu chi phí

* Chạy SageMaker Processing và Training theo job, không duy trì liên tục.
* Xóa hoặc dừng SageMaker Endpoint sau khi kiểm thử.
* Dừng EC2 khi không sử dụng.
* Sử dụng instance nhỏ cho backend demo.
* Giới hạn CloudWatch log retention.
* Xóa dữ liệu test không cần thiết.
* Sử dụng S3 lifecycle policy khi dữ liệu tăng.
* Theo dõi AWS Cost Explorer và AWS Budgets hằng ngày.
* Gắn tag đầy đủ để xác định chi phí theo module và owner.

> Chi phí cụ thể sẽ được cập nhật sau khi nhóm hoàn thành bản AWS Pricing Calculator theo cấu hình triển khai thực tế.

### 7. Đánh giá rủi ro

#### Ma trận rủi ro

| Rủi ro | Ảnh hưởng | Khả năng xảy ra | Biện pháp xử lý |
| --- | ---: | ---: | --- |
| Simulator không kết nối được AWS IoT Core | Cao | Trung bình | Kiểm tra endpoint, certificate, policy và MQTT topic |
| IoT Rule không chuyển dữ liệu sang Firehose | Cao | Trung bình | Kiểm tra SQL, IAM role, rule action và CloudWatch Logs |
| Firehose không ghi được xuống S3 | Cao | Trung bình | Kiểm tra bucket policy, execution role, prefix và error output |
| Message không đúng schema | Cao | Trung bình | Validate schema trước khi publish và trong Processing Job |
| Dữ liệu nhiều trạm bị trộn sai | Cao | Trung bình | Bắt buộc có `station_id`, kiểm tra partition và timestamp |
| Dữ liệu thiếu hoặc không liên tục | Cao | Cao | Resampling, đánh dấu missing values và áp dụng quy tắc xử lý |
| Mô hình dự báo không tốt hơn baseline | Cao | Trung bình | So sánh baseline, tuning tham số và điều chỉnh feature |
| Data leakage khi chia tập dữ liệu | Cao | Trung bình | Chia dữ liệu theo thời gian, giữ test set hoàn toàn chưa thấy |
| SageMaker Endpoint phát sinh chi phí liên tục | Cao | Trung bình | Chỉ bật khi demo hoặc kiểm thử, xóa endpoint sau khi sử dụng |
| EC2 không được tắt sau khi demo | Trung bình | Trung bình | Thiết lập checklist shutdown và theo dõi Cost Explorer |
| Không gửi được SNS Email | Trung bình | Thấp | Xác nhận subscription, kiểm tra topic policy và logs |
| Vượt ngân sách AWS | Cao | Trung bình | AWS Budgets, tagging, kiểm tra chi phí định kỳ |
| Rò rỉ certificate hoặc private key | Rất cao | Thấp | `.gitignore`, secret scanning và thu hồi certificate khi cần |
| Không hoàn thành trong 4 tuần | Cao | Trung bình | Giữ phạm vi MVP, ưu tiên pipeline end-to-end trước tính năng mở rộng |

#### Chiến lược giảm thiểu

##### Ingestion

* Test từng tầng riêng biệt trước khi kiểm thử end-to-end.
* Dùng MQTT Test Client để xác nhận IoT Core nhận message.
* Theo dõi `IncomingRecords` và `DeliveryToS3.Success`.
* Cấu hình error action hoặc logging cho IoT Rule.
* Kiểm tra object thực trong S3 thay vì chỉ dựa vào metrics.

##### Data và Machine Learning

* Lưu dữ liệu raw nguyên bản để có thể xử lý lại.
* Không sửa trực tiếp dữ liệu raw.
* Version hóa schema.
* Tách rõ train, validation và test theo thời gian.
* Luôn so sánh mô hình với baseline.
* Không triển khai endpoint nếu mô hình chưa đạt tiêu chí tối thiểu.

##### Bảo mật và chi phí

* Không chia sẻ root account.
* Áp dụng least privilege.
* Bật MFA.
* Không commit secrets.
* Dừng hoặc xóa tài nguyên sau khi kiểm thử.
* Theo dõi ngân sách và tag tài nguyên đầy đủ.

#### Kế hoạch dự phòng

* Nếu IoT Core chưa hoạt động, simulator có thể ghi dữ liệu mẫu cục bộ để tiếp tục phát triển processing pipeline.
* Nếu Firehose gặp lỗi, có thể upload file telemetry mẫu trực tiếp vào S3 Raw để kiểm thử phần Data và ML.
* Nếu mô hình DeepAR không đạt kết quả phù hợp, có thể dùng baseline hoặc thuật toán dự báo khác.
* Nếu chưa thể duy trì SageMaker Endpoint, backend có thể đọc kết quả batch forecast đã lưu trên S3.
* Nếu chưa hoàn thành push notification, phiên bản MVP vẫn giữ Amazon SNS Email.
* Nếu vượt thời gian, ưu tiên luồng end-to-end với một trạm trước, sau đó mở rộng lên 3 trạm.

### 8. Kết quả kỳ vọng

#### Kết quả kỹ thuật

Sau khi hoàn thành, hệ thống dự kiến đạt được:

* Python Simulator phát dữ liệu từ ít nhất 3 trạm.
* AWS IoT Core tiếp nhận message MQTT đúng topic.
* IoT Rule chuyển dữ liệu thành công sang Firehose.
* Firehose ghi dữ liệu vào S3 Raw.
* Dữ liệu được làm sạch và lưu trong S3 Processed.
* Mô hình dự báo PM2.5 được huấn luyện trên SageMaker.
* SageMaker Endpoint trả về kết quả dự báo 24 giờ.
* FastAPI cung cấp endpoint truy vấn forecast theo station.
* Amazon SNS gửi email khi dự báo vượt ngưỡng.
* CloudWatch cung cấp metrics và logs để kiểm tra hệ thống.
* Toàn bộ pipeline có evidence triển khai và kiểm thử.

#### Tiêu chí hoàn thành MVP

```text
1. Có dữ liệu telemetry thực trong S3 Raw.
2. Có dữ liệu sạch trong S3 Processed.
3. Có kết quả đánh giá mô hình trên test set.
4. Có dự báo PM2.5 cho từng station trong 24 giờ.
5. API trả về kết quả dự báo hợp lệ.
6. Email cảnh báo được gửi khi vượt ngưỡng.
7. Có kiểm thử end-to-end và tài liệu triển khai.
8. Chi phí nằm trong giới hạn ngân sách đã đặt.
```

#### Giá trị thực tế

Dự án thể hiện khả năng kết hợp IoT, data engineering, machine learning và cloud application thành một hệ thống hoàn chỉnh.

Thay vì chỉ hiển thị chất lượng không khí tại thời điểm hiện tại, hệ thống giúp người dùng chủ động hơn nhờ khả năng dự báo và cảnh báo sớm. Kiến trúc này cũng tạo nền tảng để tiếp tục tích hợp dữ liệu từ cảm biến thực, mở rộng phạm vi quan trắc và phát triển thành một ứng dụng hỗ trợ sức khỏe cộng đồng trong tương lai.
