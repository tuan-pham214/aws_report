---
title: "Kiểm định dữ liệu thô"
date: 2026-07-31
weight: 3
chapter: false
pre: " <b> 5.5.3. </b> "
---

Dữ liệu được gửi từ AWS IoT Core thông qua Amazon Kinesis Data Firehose sẽ được gom thành các lô (batch) trước khi lưu vào Amazon S3 nhằm tối ưu hiệu năng và chi phí lưu trữ. Tuy nhiên, dữ liệu đầu ra của Firehose thường gặp hiện tượng **Concatenated JSON**, trong đó nhiều đối tượng JSON được nối liền nhau (`}{`) mà không có ký tự xuống dòng.

Trong phần này, chúng ta sẽ xây dựng một chương trình Python để xử lý định dạng này và thực hiện kiểm định chất lượng dữ liệu thô trước khi chuyển sang bước tiền xử lý.

## 1. Chuẩn bị môi trường làm việc (WSL)

Mở **WSL (Ubuntu)** và tạo môi trường Python:

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install pandas
```

---

## 2. Viết chương trình kiểm định dữ liệu

Tạo file **raw_validation.py** và thêm đoạn mã dưới đây.

Chương trình sẽ thực hiện:

- Đọc dữ liệu thô từ S3.
- Chuyển đổi Concatenated JSON thành JSON Lines.
- Kiểm tra Schema.
- Kiểm tra kiểu dữ liệu.
- Kiểm tra giá trị bị thiếu.
- Kiểm tra Timestamp UTC.
- Kiểm tra Device ID.
- Kiểm tra quy tắc nghiệp vụ (PM2.5 không âm).
- Phát hiện dữ liệu trùng lặp.
```python
import pandas as pd
import json
import io

def validate_data(file_path):
    print("--- RAW DATA VALIDATION REPORT ---")

    try:
        # Read the entire raw file
        with open(file_path, 'r', encoding='utf-8') as f:
            raw_content = f.read()

        # Fix Firehose concatenated JSON
        formatted_content = raw_content.replace('}{', '}\n{')

        # Read as JSON Lines
        df = pd.read_json(io.StringIO(formatted_content), lines=True)

        print("=> Firehose formatting fixed successfully. Starting validation...\n")

    except Exception as e:
        print(f"File read error: {e}")
        return

    # 1. Schema validation
    required_fields = [
        'device_id',
        'timestamp',
        'pm2_5',
        'pm10',
        'temperature',
        'humidity'
    ]

    missing_cols = [
        col for col in required_fields
        if col not in df.columns
    ]

    print(f"[Schema] Missing columns: {missing_cols if missing_cols else 'None'}")

    # 2. Datatype validation
    print("\n[Datatype]")
    print(df.dtypes)

    # 3. Missing values
    print("\n[Missing Values]")
    print(df[required_fields].isnull().sum())

    # 4. Timestamp validation
    try:
        df['timestamp'] = pd.to_datetime(df['timestamp'], utc=True)
        print("\n[Timestamp] UTC format: Passed")
    except Exception as e:
        print(f"\n[Timestamp] Invalid: {e}")

    # 5. Device ID validation
    invalid_stations = df[
        df['device_id'].isnull() |
        (df['device_id'] == '')
    ]

    print(f"\n[Station] Invalid Device IDs: {len(invalid_stations)}")

    # 6. Business Rule
    negative_pm25 = df[df['pm2_5'] < 0]

    print(f"\n[Business Rule] Negative PM2.5 records: {len(negative_pm25)}")

    # 7. Duplicate detection
    duplicates = df[
        df.duplicated(
            subset=['device_id', 'timestamp'],
            keep=False
        )
    ]

    print(f"\n[Duplicate] Duplicate records: {len(duplicates)}")

    # Export sample records
    if len(negative_pm25) > 0:
        negative_pm25.head(5).to_json(
            "sample_invalid_records.json",
            orient="records",
            lines=True
        )

    valid_records = df[df['pm2_5'] >= 0].head(5)

    valid_records.to_json(
        "sample_valid_records.json",
        orient="records",
        lines=True
    )


if __name__ == "__main__":
    validate_data("sample_raw_data.json")
```
`

---

## 3. Tải dữ liệu mẫu từ Amazon S3

Sau khi Kinesis Data Firehose ghi dữ liệu xuống Amazon S3, truy cập bucket đã tạo và điều hướng đến thư mục:

```text
raw/year=2026/month=07/day=26/hour=14/
```

Tại đây bạn sẽ thấy các file dữ liệu được Firehose tự động tạo.

![Các file dữ liệu trong Amazon S3](/images/5-Workshop/5.3-data/5.3.3.1.png)

Chọn một file bất kỳ và nhấn **Download** để tải về máy tính.

Đổi tên file vừa tải thành:

```text
sample_raw_data.json
```

Sau đó đặt file này cùng thư mục với file **raw_validation.py**.

---

## 4. Chạy chương trình kiểm định

Mở Terminal WSL và di chuyển vào thư mục chứa script:

```bash
cd data
```

Sau đó thực thi chương trình:

```bash
python raw_validation.py
```

Nếu chương trình chạy thành công, Terminal sẽ hiển thị báo cáo kiểm định dữ liệu tương tự như hình dưới đây.

![Kết quả chạy chương trình kiểm định dữ liệu](/images/5-Workshop/5.3-data/5.3.3.2.png)

Báo cáo sẽ bao gồm:

- Kiểm tra Schema và các trường bắt buộc.
- Kiểm tra kiểu dữ liệu (Data Types).
- Kiểm tra giá trị bị thiếu (Missing Values).
- Kiểm tra Timestamp theo chuẩn UTC.
- Kiểm tra Device ID hợp lệ.
- Kiểm tra quy tắc nghiệp vụ (PM2.5 không được âm).
- Phát hiện các bản ghi trùng lặp (Duplicate Records).

Nếu phát hiện dữ liệu không hợp lệ, chương trình sẽ tạo các tệp mẫu chứa bản ghi lỗi để phục vụ cho bước làm sạch dữ liệu ở phần tiếp theo.

Sau khi hoàn thành bước kiểm định, dữ liệu đã sẵn sàng để chuyển sang **5.5.4 - Tiền xử lý và chuẩn hóa dữ liệu**.
