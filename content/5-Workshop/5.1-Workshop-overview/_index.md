---
title : "Introduction"
date : 2026-07-31
weight : 1
chapter : false
pre : " <b> 5.1. </b> "
---

#### Workshop objective

This workshop explains how the team built a local air-quality forecasting and alerting system on AWS in a way that can be demonstrated through one clear end-to-end flow.

Instead of writing about isolated AWS services, the section is organized by the **actual team roles**:

+ DevOps
+ Ingestion
+ Data Preparation
+ Machine Learning
+ Backend

#### Problem statement

The system must receive data from multiple simulated stations, store raw data in S3, process it into ML-ready datasets, forecast PM2.5 for the next 24 hours, expose the result through FastAPI, and send SNS alerts when thresholds are exceeded.

The current MVP prioritizes this flow:

```text
Simulator
-> AWS IoT Core
-> IoT Rule
-> Firehose
-> S3 Raw
```

This milestone is only considered complete when real objects appear in `local-aqi-dev-s3-raw`.

#### Architecture overview

Overall project architecture:

```text
MQTT Simulator
    -> AWS IoT Core
    -> IoT Rule
    -> Amazon Kinesis Data Firehose
    -> Amazon S3 Raw
    -> Data Processing
    -> Amazon S3 Processed
    -> Amazon SageMaker Processing / Training
    -> Forecast Model / Forecast Result
    -> FastAPI on EC2
    -> Amazon SNS Email Alert
```

![Overall project architecture](/images/5-Workshop/5.1-Workshop-overview/5.3-devops-local-aqi-final-architecture.png)


#### How to read this workshop

Each role follows the same structure:

1. Objective of the role.
2. Steps that were implemented.
3. AWS resources, code, or configuration that were created.
4. Outcomes achieved.
5. Evidence and demo approach.

This keeps the report consistent and makes the final demo easier to present as one continuous story.
