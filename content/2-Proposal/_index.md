---
title: "Proposal"
date: 2026-07-31
weight: 2
chapter: false
pre: " <b> 2. </b> "
---
{{% notice warning %}}
⚠️ **Note:** The information below presents the planned scope and implementation approach of the project. The architecture, cost estimates, and technical details may continue to be refined during implementation and testing.
{{% /notice %}}

This section summarizes the team project planned for the internship period, including the problem context, system architecture, implementation roadmap, budget considerations, and major risks.

# Local AQI Forecasting & Alert System
## A local air quality forecasting and alerting system on AWS

### 1. Executive Summary

The Local AQI Forecasting & Alert System is designed to collect, store, process, and forecast air quality data for individual monitoring stations. In the MVP phase, the project focuses on PM2.5 forecasting, while remaining extensible to PM10 and other environmental indicators in future versions.

In the first version, the team uses historical data from OpenAQ to build a simulator for multiple monitoring stations. Telemetry data is sent through MQTT to AWS IoT Core, routed through Amazon Data Firehose, and stored on Amazon S3 using a data lake approach.

The data is then cleaned and standardized with Amazon SageMaker Processing. A time-series forecasting model is trained and deployed on Amazon SageMaker. A FastAPI backend running on Amazon EC2 provides forecast APIs and triggers Amazon SNS to send email alerts whenever forecasted PM2.5 values exceed the configured safety threshold.

The MVP is expected to support 3 simulated stations, generate 24-hour PM2.5 forecasts, and send alerts by email. The architecture is designed so it can later scale to more stations, more environmental indicators, and additional notification channels.

### 2. Problem Statement

#### Current Problem

Air quality in large urban areas can vary significantly across locations and over time. A single city-wide AQI value often fails to reflect the real pollution level in a specific district, residential area, school, or local community.

Sensitive groups such as older adults, children, people with respiratory conditions, and people who spend significant time outdoors often do not have enough early warning information to take preventive action before pollution levels rise.

Most existing platforms mainly provide current or historical data. Short-term forecasting and station-level alerting capabilities are still limited.

#### Proposed Solution

The system builds a complete data pipeline with the following flow:

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

The main system capabilities include:

* Simulating data from multiple monitoring stations.
* Ingesting real-time telemetry through MQTT.
* Storing both raw and processed data on Amazon S3.
* Cleaning and standardizing data for Machine Learning workflows.
* Forecasting PM2.5 values for the next 24 hours for each station.
* Providing forecast results through a REST API.
* Sending email alerts when forecast values exceed the safety threshold.
* Monitoring ingestion, processing, and alert delivery with Amazon CloudWatch.

#### Practical Use Cases

Schools, healthcare facilities, or local administrative units could subscribe to alerts from the system. When PM2.5 is forecast to exceed a threshold in the coming hours, users could:

* Reduce outdoor activities.
* Adjust outdoor school or community schedules.
* Close windows or turn on air purifiers.
* Prepare protective measures for sensitive groups.
* Respond proactively to pollution risk instead of reacting only after pollution has already peaked.

#### Expected Benefits

* Provides early warnings instead of only displaying current measurements.
* Creates a centralized, reusable, and verifiable data pipeline.
* Reduces manual data collection and reporting effort.
* Establishes a foundation for future air quality forecasting research.
* Can be extended to PM10, temperature, humidity, wind, and other environmental indicators.
* Can later be expanded with dashboards, a web application, or push notifications.

### 3. Solution Architecture

The system follows an event-driven architecture combined with a structured data pipeline. Each component has a clear responsibility, making implementation, testing, and future scaling easier to manage.

#### End-to-End Processing Flow

1. A Python simulator reads historical data from OpenAQ and simulates multiple monitoring stations.
2. The simulator sends telemetry messages to AWS IoT Core through MQTT.
3. An AWS IoT Rule receives messages from the configured topic.
4. The IoT Rule forwards messages to Amazon Data Firehose.
5. Firehose batches records and writes them to Amazon S3 Raw.
6. SageMaker Processing reads the raw data, validates the schema, cleans it, and produces processed datasets.
7. SageMaker Training uses the processed data to train a time-series forecasting model.
8. The trained model is deployed as a SageMaker Endpoint.
9. FastAPI on Amazon EC2 calls the endpoint to retrieve forecast results.
10. When forecast values exceed the safety threshold, the backend sends notifications through Amazon SNS.
11. Amazon CloudWatch collects metrics and logs for monitoring and troubleshooting.

#### AWS Services Used

* **AWS IoT Core**: Ingests telemetry data from simulated stations through MQTT.
* **AWS IoT Rules Engine**: Filters and routes messages from IoT Core to Firehose.
* **Amazon Data Firehose**: Buffers records and continuously delivers them to Amazon S3.
* **Amazon S3 Raw**: Stores original telemetry data.
* **Amazon S3 Processed**: Stores cleaned and standardized data.
* **Amazon SageMaker Processing**: Validates, cleans, and transforms the data.
* **Amazon SageMaker Training**: Trains the time-series forecasting model.
* **Amazon SageMaker Endpoint**: Serves the trained model for inference through an API.
* **Amazon EC2**: Runs the FastAPI backend and alerting logic.
* **Amazon SNS**: Sends alert emails to subscribed users.
* **Amazon CloudWatch**: Monitors metrics, logs, and operational errors.
* **AWS IAM**: Controls permissions across users and AWS services.
* **AWS Budgets**: Tracks spending and sends alerts when usage exceeds thresholds.

#### Component Design

##### Data Source and Simulator

The main data source is the OpenAQ dataset. Historical data is preprocessed and fed into a Python simulator to mimic the behavior of monitoring stations.

The MVP uses 3 simulated stations. Each message is expected to contain the following key fields:

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

Optional extended fields may include:

```text
wind_speed_mps
wind_dir_deg
latitude
longitude
```

##### Data Ingestion and Routing

The simulator connects to AWS IoT Core using certificates and publishes messages to the development MQTT topic.

Planned topic convention:

```text
telemetry/aqi/dev/{station_id}
```

The IoT Rule is expected to use the following statement:

```sql
SELECT * FROM 'telemetry/aqi/dev/+'
```

After receiving a message, the IoT Rule uses an IAM service role to forward records to Firehose.

##### Data Storage

The system uses two main S3 buckets:

```text
local-aqi-dev-s3-raw
local-aqi-dev-s3-processed
```

Raw data is stored under the prefix:

```text
raw/telemetry/
```

Error records may be stored under:

```text
raw/errors/
```

Processed data is stored in the processed bucket using a time-based or station-based structure to support training and querying.

##### Machine Learning

SageMaker Processing is responsible for:

* Reading data from S3 Raw.
* Validating the schema.
* Handling missing values.
* Removing or flagging anomalous records.
* Standardizing timestamps.
* Sorting records by station and time.
* Producing train, validation, and test datasets.
* Writing cleaned outputs to S3 Processed.

The model is expected to use DeepAR or another suitable time-series forecasting approach on SageMaker.

The main model outputs include:

* PM2.5 forecasts for the next 24 hours.
* Forecast values for each timestamp.
* Prediction intervals or forecast quantiles if supported.
* Alert status based on configured thresholds.

##### Backend and Alerts

FastAPI is deployed on Amazon EC2 and is expected to provide the following endpoints:

```text
GET /health
GET /stations
GET /forecast/{station_id}
POST /subscriptions
```

The backend either calls the SageMaker Endpoint directly or reads saved forecast results. When a forecast exceeds the configured threshold, the backend triggers Amazon SNS to send alert emails to subscribed users.

### 4. Technical Implementation

#### Implementation Phases

The project is planned across 4 weeks with the following major phases:

##### Phase 1 - Design and Standardization

* Finalize the MVP scope.
* Select PM2.5 as the primary forecasting target.
* Set the forecast horizon to 24 hours.
* Select 3 stations for the first version.
* Standardize the telemetry schema.
* Finalize naming and tagging conventions.
* Design the AWS architecture.
* Create IAM users, IAM roles, and budget alerts.

##### Phase 2 - Data Ingestion

* Prepare OpenAQ data.
* Build the Python simulator.
* Create certificates and IoT policies.
* Configure the MQTT topic.
* Create the IoT Rule.
* Connect IoT Core to Firehose.
* Configure Firehose delivery to S3 Raw.
* Test multiple messages across multiple stations.
* Collect evidence for the full ingestion pipeline.

Phase milestone:

```text
Simulator -> IoT Core -> Firehose -> S3 Raw
```

Real JSON telemetry must appear in the raw bucket.

##### Phase 3 - Data Processing and Machine Learning

* Build the processing script.
* Clean and standardize the data.
* Generate train, validation, and test datasets.
* Train the forecasting model.
* Evaluate model performance with metrics such as MAE and RMSE.
* Compare results against a simple baseline.
* Save model artifacts.
* Deploy the SageMaker Endpoint.
* Test inference for each station.

##### Phase 4 - API, Alerting, and End-to-End Testing

* Build the FastAPI backend.
* Deploy the backend on EC2.
* Connect the backend to the SageMaker Endpoint.
* Create the SNS topic and email subscriptions.
* Implement alert-threshold comparison logic.
* Test the API.
* Test email delivery.
* Test the full flow from simulator to alert recipient.
* Finalize deployment evidence and project documentation.

#### Technical Requirements

##### Development Environment

* Python 3.10 or a compatible version.
* Git and GitHub.
* AWS CLI.
* Visual Studio Code.
* A Python virtual environment.
* Paho MQTT.
* Pandas.
* Boto3.
* FastAPI.
* Uvicorn.
* SageMaker Python SDK.

Example base dependencies:

```bash
pip install paho-mqtt pandas boto3 fastapi uvicorn sagemaker
```

##### Security Configuration

* Do not use the root account for daily work.
* Each team member should use a separate IAM user.
* Enable MFA for accounts with administrative access.
* Apply the principle of least privilege.
* Private keys and certificates must never be committed to Git.
* The `certs/` directory and `.env` file must be included in `.gitignore`.
* The IoT Core role should only be allowed to write to the designated Firehose stream.
* The Firehose role should only be allowed to write to the required S3 bucket and prefix.
* The SageMaker execution role should only access the buckets used for processing and training.

##### Naming Convention

Resource names follow the structure:

```text
local-aqi-{environment}-{service-or-purpose}
```

Examples:

```text
local-aqi-dev-s3-raw
local-aqi-dev-s3-processed
local-aqi-dev-firehose-telemetry
local-aqi-dev-iot-to-firehose-role
local-aqi-dev-firehose-to-s3-role
local-aqi-dev-sagemaker-execution-role
```

##### Tagging Convention

Resources that support tagging are expected to use:

```text
Project     = local-aqi-forecasting
Environment = dev
Owner       = team-member-name
Module      = ingestion | data | ml | backend | devops
```

### 5. Timeline and Milestones

The project is expected to be implemented over approximately 4 weeks.

#### Week 1 - Architecture and Ingestion

* Finalize version 1 of the architecture.
* Finalize the telemetry schema.
* Create S3 Raw and S3 Processed.
* Build the simulator.
* Configure AWS IoT Core.
* Configure Firehose.
* Complete the IoT Rule -> Firehose -> S3 Raw flow.

**Milestone 1**

```text
Real simulator data appears in S3 Raw.
```

#### Week 2 - Data Processing

* Review data quality.
* Build the processing pipeline.
* Standardize data by station.
* Create train, validation, and test datasets.
* Store cleaned data in S3 Processed.

**Milestone 2**

```text
Processed data is ready for model training.
```

#### Week 3 - Machine Learning

* Build a baseline model.
* Train DeepAR or another suitable forecasting model.
* Evaluate MAE and RMSE.
* Review forecasts by station.
* Deploy the SageMaker Endpoint.

**Milestone 3**

```text
The SageMaker Endpoint returns 24-hour PM2.5 forecasts.
```

#### Week 4 - Backend and Alerting

* Build the FastAPI service.
* Deploy it on EC2.
* Connect the API to the SageMaker Endpoint.
* Configure SNS.
* Test alert emails.
* Run end-to-end tests.
* Finalize documentation, screenshots, and report materials.

**Milestone 4**

```text
Users receive an email when forecasted PM2.5 exceeds the threshold.
```

#### Post-MVP Extensions

* Extend forecasting from 24 hours to 48 hours.
* Increase the number of stations.
* Add PM10, temperature, humidity, and wind indicators.
* Build a visual dashboard.
* Add push notifications or SMS.
* Automate retraining.
* Monitor model drift and data drift.
* Integrate data from real monitoring sensors.

### 6. Budget Estimation

The main cost drivers of the project come from:

* Amazon SageMaker Processing.
* SageMaker Training.
* SageMaker Endpoint.
* Amazon EC2.
* Amazon S3.
* Amazon Data Firehose.
* AWS IoT Core.
* Amazon SNS.
* Amazon CloudWatch.

Because the MVP data volume is relatively small, IoT Core, Firehose, S3, and SNS costs are expected to remain low. The components requiring the closest cost control are SageMaker Endpoint, SageMaker Processing, training jobs, and EC2, since these resources can continue generating charges if not stopped on time.

Exact costs should be recalculated with the AWS Pricing Calculator based on:

* Region `ap-southeast-1`.
* Number of messages per month.
* Size of each telemetry record.
* S3 storage volume.
* Number of Processing Job runs.
* Training Job duration.
* SageMaker instance type.
* SageMaker Endpoint uptime.
* EC2 instance type and running duration.
* Number of alert emails sent through SNS.

#### Budget Control Principles

The team sets a maximum AWS budget of 100 USD for the MVP phase and applies the following control thresholds:

* **10 USD or 5% of the budget**: Review which services are generating cost.
* **25 USD or 15% of the budget**: Reassess resources and runtime duration.
* **50 USD or 30% of the budget**: Pause non-essential resources.
* **100 USD or 50% of available credits**: Freeze scope and keep only MVP components.

#### Cost Optimization Measures

* Run SageMaker Processing and Training as jobs instead of keeping them active continuously.
* Stop or delete the SageMaker Endpoint after testing.
* Stop EC2 when it is not in use.
* Use a small instance for the demo backend.
* Limit CloudWatch log retention.
* Remove unnecessary test data.
* Apply S3 lifecycle policies when data volume grows.
* Review AWS Cost Explorer and AWS Budgets daily.
* Apply full tagging to identify cost by module and owner.

> The final cost figures will be updated after the team completes an AWS Pricing Calculator estimate based on the actual deployment configuration.

### 7. Risk Assessment

#### Risk Matrix

| Risk | Impact | Likelihood | Mitigation |
| --- | ---: | ---: | --- |
| Simulator cannot connect to AWS IoT Core | High | Medium | Check endpoint, certificate, policy, and MQTT topic |
| IoT Rule does not forward data to Firehose | High | Medium | Check SQL, IAM role, rule action, and CloudWatch Logs |
| Firehose cannot write to S3 | High | Medium | Check bucket policy, execution role, prefix, and error output |
| Messages do not match the schema | High | Medium | Validate schema before publish and inside the Processing Job |
| Multi-station data gets mixed incorrectly | High | Medium | Require `station_id` and validate partitioning and timestamps |
| Data is missing or discontinuous | High | High | Use resampling, mark missing values, and apply handling rules |
| Forecast model does not outperform the baseline | High | Medium | Compare against baseline, tune parameters, and adjust features |
| Data leakage occurs during dataset splitting | High | Medium | Split by time and keep the test set fully unseen |
| SageMaker Endpoint generates ongoing cost | High | Medium | Enable it only for testing or demo and delete it afterward |
| EC2 is left running after the demo | Medium | Medium | Use a shutdown checklist and monitor Cost Explorer |
| SNS email delivery fails | Medium | Low | Confirm subscriptions, check topic policy, and inspect logs |
| AWS budget is exceeded | High | Medium | Use AWS Budgets, tagging, and regular cost checks |
| Certificate or private key leakage | Very high | Low | Use `.gitignore`, secret scanning, and certificate revocation if needed |
| The project is not completed within 4 weeks | High | Medium | Keep MVP scope focused and prioritize the end-to-end pipeline first |

#### Mitigation Strategy

##### Ingestion

* Test each layer independently before running end-to-end tests.
* Use the MQTT Test Client to confirm message arrival in IoT Core.
* Monitor `IncomingRecords` and `DeliveryToS3.Success`.
* Configure error actions or logging for IoT Rules.
* Check real S3 objects instead of relying only on metrics.

##### Data and Machine Learning

* Preserve the original raw data for reprocessing if needed.
* Do not modify raw data directly.
* Version the telemetry schema.
* Separate train, validation, and test sets by time.
* Always compare model performance with a baseline.
* Do not deploy the endpoint if the model fails to meet minimum quality criteria.

##### Security and Cost

* Do not share the root account.
* Apply least privilege.
* Enable MFA.
* Do not commit secrets.
* Stop or delete resources after testing.
* Monitor budgets and apply complete resource tagging.

#### Contingency Plan

* If IoT Core is not ready, the simulator can write sample data locally so the processing pipeline can still be developed.
* If Firehose fails, sample telemetry files can be uploaded directly into S3 Raw to continue testing the Data and ML phases.
* If DeepAR does not perform well enough, a baseline model or another forecasting algorithm can be used.
* If the SageMaker Endpoint cannot be kept active, the backend can read batch forecast results stored in S3.
* If push notifications are not completed in time, the MVP will keep Amazon SNS Email only.
* If time becomes limited, the team will prioritize a one-station end-to-end flow first, then expand to 3 stations.

### 8. Expected Outcomes

#### Technical Deliverables

After completion, the system is expected to achieve the following:

* A Python simulator publishes data from at least 3 stations.
* AWS IoT Core receives MQTT messages on the correct topic.
* The IoT Rule forwards data successfully to Firehose.
* Firehose writes records to S3 Raw.
* The data is cleaned and stored in S3 Processed.
* A PM2.5 forecasting model is trained on SageMaker.
* The SageMaker Endpoint returns 24-hour forecast results.
* FastAPI provides forecast query endpoints by station.
* Amazon SNS sends emails when thresholds are exceeded.
* CloudWatch provides metrics and logs for system validation.
* The full pipeline is supported by implementation and testing evidence.

#### MVP Completion Criteria

```text
1. Real telemetry data exists in S3 Raw.
2. Clean data exists in S3 Processed.
3. Model evaluation results are available on the test set.
4. PM2.5 forecasts are available for each station for the next 24 hours.
5. The API returns valid forecast results.
6. Alert emails are sent when thresholds are exceeded.
7. End-to-end testing and deployment documentation are completed.
8. Total cost remains within the defined budget limit.
```

#### Practical Value

The project demonstrates how IoT, data engineering, machine learning, and cloud application components can be combined into a complete system.

Instead of only showing current air quality conditions, the system helps users act more proactively through short-term forecasting and early alerts. The architecture also creates a foundation for integrating real sensor data later, expanding monitoring coverage, and evolving into a future application that supports community health and environmental awareness.
