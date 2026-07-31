---
title: "Monitoring & Quality Assurance"
date: 2026-07-31
weight: 3
chapter: false
pre: "<b>3. </b>"
---

## 1. Objective

Monitor service health, collect the required logs, and test the complete workflow before acceptance.

## 2. Monitoring and Logging

Monitor these components:

- AWS IoT Core and the IoT Rule.
- Amazon Data Firehose.
- Amazon S3 Raw and S3 Processed.
- SageMaker Processing or Training Jobs.
- The backend API.
- Amazon SNS.

Check the following information:

```text
Incoming record count
Successful delivery count
Failed delivery count
Data freshness
Training job status
API errors
SNS delivery result
```

When reporting an error, include the time, resource name, error message, log location, and action in progress.

![Firehose metric confirming successful delivery to S3](/images/5-Workshop/5.4-Ingestion/firehose-delivery-metrics.png)

![CloudWatch Logs for the SageMaker Training Job](/images/5-Workshop/5.3-DevOps/cloudwatch-log-events.png)

## 3. Module Tests

### Data Ingestion

- Send one message and then multiple messages.
- Send data from multiple stations.
- Test a message with a missing field.
- Test a failed delivery case.

### Data Processing

- Read JSON from S3 Raw.
- Process concatenated JSON objects.
- Check null values and duplicate records.
- Check negative values and UTC timestamps.
- Write and read back a Parquet file.

### Machine Learning

- Read the processed dataset.
- Run training.
- Check the model artifact.
- Generate a 24-hour forecast.
- Record MAE and RMSE.
- Check the Training Job status.

### Backend Service

- Check service health.
- Request a forecast for a valid station ID.
- Handle a station that does not exist.
- Handle an endpoint that is not ready.
- Verify successful SNS publishing.
- Confirm the email subscription.

## 4. Integration Testing

Acceptance flow:

```text
Simulator
→ AWS IoT Core
→ IoT Rule
→ Firehose
→ S3 Raw
→ Data Processing
→ S3 Processed
→ ML Forecast
→ Backend
→ SNS Email
```

The normal scenario verifies that data from multiple stations passes through the complete pipeline and produces a forecast. The threshold scenario verifies that the backend triggers SNS. Error scenarios cover missing fields, invalid data types, duplicate records, invalid station IDs, and API requests for unknown stations.

![Ingestion evidence showing actual objects written to S3 Raw](/images/5-Workshop/5.3-DevOps/ingestion-evidence.png)

![DeepAR evaluation result on SageMaker](/images/5-Workshop/5.6-Machine-learning/deepar_sagemaker_evaluation.png)

## 5. Result Template

```text
Test case:
Input:
Expected result:
Actual result:
Status: Pass / Fail
Evidence:
Owner:
```

## 6. Acceptance Criteria

- The simulator sends data successfully.
- IoT Core receives the message.
- Firehose writes data to S3 Raw.
- Processed data can be read as Parquet.
- The model produces a forecast.
- The backend returns the expected response.
- SNS sends an email when the threshold is exceeded.
- Logs and evidence are available for each critical step.

## 7. Outcomes

- Each module has defined test cases.
- The project uses a consistent issue-reporting process.
- End-to-end test evidence is available.
- Failures can be isolated to a specific stage of the workflow.
