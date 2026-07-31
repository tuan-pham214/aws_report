---
title: "Tiền xử lý và chuẩn hóa dữ liệu"
date: 2026-07-31
weight: 4
chapter: false
pre: " <b> 5.5.4. </b> "
---

Sau khi kiểm định dữ liệu thô ở bước trước, chúng ta tiến hành làm sạch dữ liệu, loại bỏ các bản ghi trùng lặp, chuẩn hóa mốc thời gian, thực hiện **Resample theo chu kỳ 1 giờ (1H)** và nội suy (Interpolation) các giá trị còn thiếu. Bộ dữ liệu sau xử lý sẽ được xuất ra định dạng **Apache Parquet**, sẵn sàng phục vụ cho quá trình huấn luyện mô hình Machine Learning.

## 1. Chuẩn bị môi trường (WSL)

Mở Terminal WSL và đảm bảo môi trường ảo của dự án đã được kích hoạt. Tiếp theo, cài đặt các thư viện Python cần thiết:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pandas pyarrow
```

---

## 2. Tạo chương trình  xử lý dữ liệu

Tạo một tệp có tên `data_processing.py` trong thư mục `data` và dán đoạn mã sau vào.

```python
import pandas as pd
import io
import os

def process_raw_to_ml_dataset(input_path, output_path):
    print("--- BẮT ĐẦU XỬ LÝ DỮ LIỆU TỪ RAW SANG PROCESSED ---")
    
    # 1. Đọc dữ liệu thô và xử lý lỗi Concatenated JSON từ Firehose
    with open(input_path, 'r', encoding='utf-8') as f:
        raw_content = f.read()

    formatted_content = raw_content.replace('}{', '}\n{')
    df = pd.read_json(io.StringIO(formatted_content), lines=True)
    print(f"[*] Đã đọc {len(df)} dòng dữ liệu thô.")

    # 2. Chuẩn hóa Schema và Timestamp
    df['timestamp'] = pd.to_datetime(df['timestamp'], utc=True)
    df = df[['device_id', 'timestamp', 'pm2_5', 'pm10', 'temperature', 'humidity']]

    # 3. Loại bỏ dữ liệu trùng lặp
    df = df.drop_duplicates(subset=['device_id', 'timestamp'], keep='last')
    print(f"[*] Đã xóa các dòng trùng lặp. Dữ liệu còn: {len(df)} dòng.")

    # 4. Resample dữ liệu theo chu kỳ 1 giờ
    print("[*] Đang Resample dữ liệu theo tần suất 1 giờ (1H)...")
    processed_dfs = []

    for device, group in df.groupby('device_id'):
        group = group.set_index('timestamp')
        resampled = group.resample('1h').mean(numeric_only=True)
        resampled = resampled.interpolate(method='linear')

        resampled['device_id'] = device
        processed_dfs.append(resampled.reset_index())

    final_df = pd.concat(processed_dfs, ignore_index=True)
    final_df = final_df.sort_values(by=['timestamp', 'device_id']).reset_index(drop=True)

    # 5. Xuất dữ liệu sang định dạng Parquet
    final_df.to_parquet(output_path, engine='pyarrow', index=False)

    print(f"[✓] Đã xuất thành công bộ dữ liệu: {output_path}")

    print("\n--- DATA QUALITY SUMMARY ---")
    print(final_df.info())

    print("\n[Sample Data]")
    print(final_df.head(5))

if __name__ == "__main__":
    process_raw_to_ml_dataset(
        'sample_raw_data.json',
        'sample_processed_dataset.parquet'
    )
```

---

## 3. Thực thi chương trình trên WSL

Di chuyển vào thư mục chứa mã nguồn và chạy chương trình:

```bash
cd data
python data_processing.py
```

Chương trình sẽ tự động thực hiện các công việc sau:

- Đọc dữ liệu JSON thô.
- Khắc phục lỗi **Concatenated JSON** do Kinesis Data Firehose tạo ra.
- Loại bỏ các bản ghi trùng lặp.
- Chuẩn hóa trường thời gian sang định dạng UTC.
- Resample dữ liệu theo chu kỳ 1 giờ (1H).
- Nội suy các giá trị còn thiếu bằng phương pháp **Linear Interpolation**.
- Xuất bộ dữ liệu đã xử lý sang định dạng **Apache Parquet**.

Sau khi thực thi thành công, Terminal sẽ hiển thị báo cáo tổng quan về chất lượng dữ liệu như hình dưới đây.

![Kết quả chạy data_processing.py trên WSL](/images/5-Workshop/5.3-data/5.3.4.1.png)

---

## 4. Tải dữ liệu đã xử lý lên Amazon S3

Sau khi xác nhận tệp Parquet đã được tạo thành công trên máy cục bộ:

1. Quay trở lại **Amazon S3 Console**.
2. Mở bucket của dự án.
3. Điều hướng đến thư mục:

```text
processed/ml-ready/
```

4. Nhấn **Upload**.
5. Chọn và tải lên tệp:

```text
sample_processed_dataset.parquet
```

Sau khi hoàn tất, tệp dữ liệu đã xử lý sẽ xuất hiện trong thư mục **ml-ready** của S3.

![Tệp Parquet đã được tải lên Amazon S3](/images/5-Workshop/5.3-data/5.3.4.2.png)

---

## Kết quả

Đến bước này, toàn bộ quy trình xử lý dữ liệu của Data Pipeline đã được hoàn thành:

- Thu thập dữ liệu telemetry từ các thiết bị IoT.
- Kiểm định chất lượng và cấu trúc dữ liệu.
- Loại bỏ các bản ghi trùng lặp.
- Chuẩn hóa dữ liệu chuỗi thời gian theo chu kỳ 1 giờ.
- Nội suy các giá trị còn thiếu.
- Xuất dữ liệu sang định dạng **Apache Parquet** tối ưu cho phân tích và Machine Learning.
- Lưu trữ bộ dữ liệu đã xử lý trong **Amazon S3 Data Lake**, sẵn sàng cho bước huấn luyện mô hình Machine Learning ở các phần tiếp theo.
