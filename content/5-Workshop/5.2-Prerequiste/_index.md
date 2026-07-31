---
title: "Prerequisites"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 5.2. </b> "
---

#### Prepare the AWS account and access model

Before deploying the system, the team needs an AWS account that can use the core services required by the project.

All resources are standardized in the following Region:

```text
Asia Pacific (Singapore)
ap-southeast-1
```

Using one shared Region helps the team:

* avoid creating resources in multiple Regions by mistake
* manage cost and runtime status more easily
* reduce integration errors across IoT Core, Firehose, S3, and SageMaker
* keep configuration consistent across members

#### IAM users and access-control principles

The Root account must not be used or shared among team members.

Each member should use a separate IAM user to:

* sign in to the AWS Management Console
* work only with the services assigned to their role
* create and validate resources for their own module
* reduce the risk of affecting resources owned by other members

IAM permissions should be granted based on role and task scope rather than giving every member full access.

Typical service permissions needed during implementation include:

```text
AWS IoT Core
Amazon Data Firehose
Amazon S3
Amazon SageMaker AI
Amazon CloudWatch
Amazon SNS
IAM PassRole
Billing read-only when cost review is needed
```

Permissions related to IAM role creation or service-role configuration should only be granted for the required period of time. Once the setup is complete, temporary permissions should be removed.

#### Project IAM roles

The project uses separate IAM roles for different services.

| IAM role                                 | Trusted service      | Purpose |
| ---------------------------------------- | -------------------- | ------- |
| `local-aqi-dev-iot-to-firehose`          | AWS IoT Core         | Allows the IoT Rule to send records to Firehose |
| `local-aqi-dev-firehose-to-s3`           | Amazon Data Firehose | Allows Firehose to write data into S3 Raw and write error logs |
| `local-aqi-dev-sagemaker-execution-role` | Amazon SageMaker AI  | Allows SageMaker to read data, run Processing and Training, and store model artifacts |

The trust policy of each role must explicitly identify the correct service that is allowed to assume it.

For example, the role used by AWS IoT Core should trust:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "iot.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

The role used by Amazon Data Firehose should trust:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

The role used by Amazon SageMaker AI should include:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "sagemaker.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

During development, the SageMaker role may temporarily trust additional services to support experiments. After the system stabilizes, both trust and permission policies should be reduced following least-privilege principles.

#### Prepare the S3 buckets

Before the pipeline is deployed, the following two main buckets should exist:

```text
local-aqi-dev-s3-raw
local-aqi-dev-s3-processed
```

Where:

* `local-aqi-dev-s3-raw` stores telemetry written directly by Firehose
* `local-aqi-dev-s3-processed` stores cleaned and transformed data prepared for Machine Learning

Raw data is organized by time-based partitions:

```text
raw/
  year=YYYY/
    month=MM/
      day=DD/
        hour=HH/
```

This structure helps:

* query data by time range more easily
* avoid scanning the entire bucket
* simplify downstream processing
* verify newly ingested data quickly

#### Prepare the ingestion pipeline

The ingestion pipeline uses the following components:

```text
MQTT Simulator
-> AWS IoT Core
-> IoT Rule
-> Amazon Data Firehose
-> Amazon S3 Raw
```

The main resources that should already exist include:

```text
IoT Thing Group:
local-aqi-dev-iot-group

IoT Policy:
local-aqi-dev-iot-policy

IoT Rule:
Rule that routes data from the MQTT topic to Firehose

Firehose delivery stream:
local-aqi-dev-firehose-telemetry
```

MQTT messages must follow the shared telemetry schema.

Example:

```json
{
  "schema_version": "1.0",
  "station_id": "station_001",
  "ts_utc": "2026-07-31T08:00:00Z",
  "pm25_ugm3": 42.5,
  "pm10_ugm3": 61.2,
  "temperature_c": 30.5,
  "humidity_pct": 72,
  "source": "simulator"
}
```

Required fields:

```text
schema_version
station_id
ts_utc
pm25_ugm3
pm10_ugm3
temperature_c
humidity_pct
source
```

#### Prepare the Machine Learning environment

The forecasting model is trained using Amazon SageMaker AI.

Training data includes:

* historical PM2.5 data that has already been collected and processed
* normalized datasets stored in S3 Processed
* time series grouped by station and timestamp

Realtime data from the MQTT Simulator is mainly used to validate the ingestion pipeline and to add new operational data. The team does not need to wait for the simulator to generate enough new data before training begins.

The expected Machine Learning flow is:

```text
S3 Raw
-> Data Processing
-> S3 Processed
-> SageMaker Training Job
-> Model Artifact
-> SageMaker Endpoint
```

The SageMaker execution role should have at least:

```text
s3:ListBucket
s3:GetObject
s3:PutObject
s3:DeleteObject if the pipeline overwrites processed data
```

These permissions should be limited to the two project buckets instead of granting `s3:*` across the entire account.

#### Prepare monitoring and alerting

The following services should be monitored through Amazon CloudWatch:

```text
Amazon Data Firehose
SageMaker Training Jobs
SageMaker Endpoint
Backend API
SNS publish result
```

Firehose should have:

```text
Destination error logs
Amazon CloudWatch error logging: Enabled
```

Expected log groups include:

```text
/aws/kinesisfirehose/local-aqi-dev-firehose-telemetry
/aws/sagemaker/TrainingJobs
/aws/sagemaker/Endpoints/aqi-endpoint-test
```

Amazon SNS is used to send email alerts when PM2.5 values or forecast outputs exceed the configured threshold.

The email subscription must be in this state:

```text
Confirmed
```

before alert testing is executed.

#### Pre-deployment checklist

Before moving to the next deployment sections, confirm:

```text
[ ] Region ap-southeast-1 is being used
[ ] The IAM user can sign in to AWS Console
[ ] The IAM role for IoT Core has been created
[ ] The IAM role for Firehose has been created
[ ] The IAM role for SageMaker has been created
[ ] The S3 Raw bucket has been created
[ ] The S3 Processed bucket has been created
[ ] The Firehose delivery stream has been created
[ ] CloudWatch error logging has been enabled
[ ] SageMaker can access data in S3
[ ] The SNS email subscription has been confirmed
[ ] AWS Budgets have been configured
```

Once these conditions are satisfied, the team can continue with:

```text
IoT ingestion
Data processing
Machine Learning
Backend API
SNS alert
Integration testing
```
