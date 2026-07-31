---
title: "Building the Data Pipeline"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5.5. </b> "
---

In this module, we will build a complete **Data Pipeline** on AWS.

Environmental data (PM2.5, temperature, and humidity) generated from a simulated IoT device will be collected and stored in an Amazon S3 Data Lake. The data will then go through **validation** and **preprocessing** stages to prepare it for Machine Learning models in the following modules.

**This hands-on lab includes:**

* **5.5.1:** Create the central Amazon S3 Data Lake.
* **5.5.2:** Configure Amazon Kinesis Data Firehose for batch data ingestion.
* **5.5.3:** Develop a validation script to verify raw data quality.
* **5.5.4:** Preprocess the data, resample it to a 1-hour frequency, and export it in Parquet format.
