---
title: "Configure Amazon Kinesis Data Firehose"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.5.2. </b> "
---

Amazon Kinesis Data Firehose enables you to automatically collect streaming data from IoT devices and deliver it to your Data Lake in Amazon S3 as batched files.

#### Creating a Firehose Delivery Stream

1. Open the **Amazon Kinesis** service from the AWS Management Console.

2. On the Kinesis dashboard, scroll down to the **Amazon Data Firehose** section and choose **Create Firehose stream**.

3. Under **Choose source and destination**:

   * **Source:** Select **Direct PUT** (Telemetry data from AWS IoT Core Rules will be sent directly to this delivery stream.)
   * **Destination:** Select **Amazon S3**.

   ![Choose Source and Destination](/images/5-Workshop/5.3-data/5.3.2.1.png)

4. Enter a name for the delivery stream in **Firehose stream name**, for example:

   `local-aqi-dev-firehose-telemetry`

   ![Firehose stream name](/images/5-Workshop/5.3-data/5.3.2.2.png)

---

#### Configuring the Destination

1. Scroll down to **Destination settings**.

   Under **S3 bucket**, choose **Browse** and select the S3 bucket created in **Section 5.5.1** (for example: `local-aqi-dev-s3-raw`).

   ![Select Amazon S3 destination](/images/5-Workshop/5.3-data/5.3.2.3.png)

2. In the **S3 bucket prefix** field, copy and paste the following prefix exactly as shown.

   This configuration automatically partitions incoming data using the **Hive-style** directory structure, improving query performance and reducing costs when using Amazon Athena.

```text
raw/year=!{timestamp:yyyy}/month=!{timestamp:MM}/day=!{timestamp:dd}/hour=!{timestamp:HH}/
```

![Configure S3 data and error output prefixes](/images/5-Workshop/5.3-data/5.3.2.4.png)

---

#### Configuring Service Access and Tags

1. Under **Service access**, keep the default option **Create or update IAM role**.

   AWS will automatically create an IAM service role (for example, `KinesisFirehoseServiceRole-...`) with the minimum permissions required to write data to Amazon S3.

   ![Configure IAM Role](/images/5-Workshop/5.3-data/5.3.2.5.png)

2. Scroll down to **Tags (optional)** and add the following resource tags:

   * **Key:** `Project` | **Value:** `local-aqi-forecasting`
   * **Key:** `Environment` | **Value:** `dev`
   * **Key:** `Owner` | **Value:** `quynh-tam`
   * **Key:** `Module` | **Value:** `data`

   ![Firehose resource tags](/images/5-Workshop/5.3-data/5.3.2.6.png)

3. Scroll to the bottom of the page and choose **Create Firehose stream**.

The provisioning process may take a few minutes. Once the delivery stream status changes to **Active** (green) in the Firehose console, your data pipeline is ready to receive telemetry messages from AWS IoT Core.

![Firehose Active status](/images/5-Workshop/5.3-data/5.3.2.7.png)
