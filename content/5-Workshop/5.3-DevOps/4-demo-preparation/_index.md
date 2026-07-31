---
title: "Demo Preparation"
date: 2026-07-31
weight: 4
chapter: false
pre: "<b>4. </b>"
---

## 1. Objective

Prepare a concise demonstration that proves the system works end to end, from incoming data to forecasts and alerts.

## 2. Presentation Principles

Every part of the demonstration must include:

```text
Input
Action
Output
Evidence
```

Do not merely open an AWS console page and describe its configuration.

## 3. Demonstration Order

1. Introduce the problem and MVP scope.
2. Present the architecture diagram.
3. Run the simulator with multiple stations.
4. Verify messages in the MQTT Test Client.
5. Check Firehose and S3 Raw.
6. Run data processing and create a Parquet file.
7. Present SageMaker Training and the forecast result.
8. Test the backend API.
9. Trigger an SNS alert.
10. Present CloudWatch and AWS Budgets.

## 4. Presentation Assignments

```text
Architecture and DevOps      → M5
Simulator and IoT Core       → Ingestion Engineer
S3 and the data pipeline     → Data Engineer
SageMaker and forecasting    → ML Engineer
Backend and SNS              → Backend Engineer
Conclusion                   → Team Lead
```

![GitHub Project task board and demo status](/images/5-Workshop/5.3-DevOps/github-project-board.png)

## 5. Pre-recording Checklist

- [ ] Signed in to the correct AWS account.
- [ ] Region is `ap-southeast-1`.
- [ ] The simulator runs successfully.
- [ ] MQTT Test Client is subscribed to the correct topic.
- [ ] Firehose is `Active`.
- [ ] S3 Raw contains a new object.
- [ ] Data processing completes successfully.
- [ ] S3 Processed contains a Parquet file.
- [ ] The SageMaker Training Job is `Completed`.
- [ ] Forecast results are ready.
- [ ] The backend API is working.
- [ ] The SNS subscription is confirmed.
- [ ] The alert email has been verified.
- [ ] CloudWatch contains logs.
- [ ] AWS Budgets is accessible.
- [ ] No credentials are exposed.

## 6. Fallback Plan

Prepare a screenshot of the completed Training Job, a forecast JSON or CSV file, a short ingestion-flow video, a sample S3 object, a sample API response, a received SNS email, an offline architecture diagram, and log files as evidence.

When fallback evidence is used, clearly state that it was captured during an earlier successful test.

## 7. Required Evidence

```text
Architecture diagram
GitHub Project Board
Simulator output
MQTT Test Client
Firehose Monitoring
S3 Raw object
Parquet file in S3 Processed
SageMaker Training Job
Forecast result
API response
SNS email
CloudWatch Logs
AWS Budgets
```

![End-to-end architecture used for the demo flow](/images/5-Workshop/5.1-Workshop-overview/5.3-devops-local-aqi-final-architecture.png)

## 8. Expected Outcome

- Data is sent from multiple stations.
- IoT Core receives the messages.
- Firehose writes data to S3.
- The pipeline creates processed data.
- The model produces a forecast.
- The backend exposes the result through its API.
- SNS sends an alert.
- Monitoring and cost controls are in place.
