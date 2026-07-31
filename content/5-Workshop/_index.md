---
title: "Workshop"
date: 2026-07-31
weight: 5
chapter: false
pre: " <b> 5. </b> "
---

{{% notice info %}}
This section is the **main technical project** of the report. It is organized by team role so it matches both the real implementation split and the final end-to-end demo flow.
{{% /notice %}}

#### Overview

This workshop is written around **one complete end-to-end flow** instead of isolated AWS services. Each role explains:

+ what steps were implemented,
+ what component was created,
+ and what outcome was achieved in the overall system.

Main demo flow:

```text
Simulator
-> AWS IoT Core
-> IoT Rule
-> Kinesis Data Firehose
-> S3 Raw
-> Data Processing
-> S3 Processed
-> SageMaker Training / Forecast Result
-> FastAPI
-> SNS Email
```

#### Role-based breakdown

+ `5.3 DevOps`: problem definition, architecture, naming/tagging, IAM account setup, and access control.
+ `5.4 Ingestion`: sending simulator data through IoT Core, Firehose, and into S3 Raw.
+ `5.5 Data Preparation`: reading raw data, cleaning it, and producing ML-ready datasets.
+ `5.6 Machine Learning`: training the PM2.5 forecasting model and producing forecast results.
+ `5.7 Backend`: exposing the API and sending SNS alerts.

#### Content

1. [Workshop overview](5.1-Workshop-overview/)
2. [Prerequisites](5.2-Prerequiste/)
3. [DevOps: architecture, IAM, and account governance](5.3-DevOps/)
4. [Ingestion: Simulator -> IoT Core -> Firehose -> S3 Raw](5.4-Ingestion/)
5. [Data Preparation: data processing and normalization](5.5-Data%20Preprocessing/)
6. [Machine Learning: training and forecast generation](5.6-Machine%20learning/)
7. [Deploy the FastAPI backend](5.7-Backend-deployment/)
8. [Access policies and IAM authorization](5.8-Policy/)
9. [Resource cleanup](5.9-Cleanup/)
