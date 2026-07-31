---
title: "Week 2 Worklog"
weight: 2
chapter: false
pre: " <b> 1.2. </b> "
includeInReport: true
reportTableColumns:
  - Day
  - Task
  - Completion Date
reportHeadings:
  - Weekly Objectives
  - Daily Tasks
  - Achievements
reportType: worklog
---

### Weekly Objectives

- Study AWS services suitable for AQI ingestion, storage, forecasting, and alerting.
- Design the logical architecture and define the contract and responsibility of each component.
- Evaluate deployment choices in terms of security, scalability, and internship-environment cost.

### Daily Tasks

| Day | Task | Start Date | Completion Date | Result |
|---|---|---|---|---|
| Monday | Studied OpenAQ, the Python telemetry simulator, MQTT topics, and Quality of Service. Compared AWS IoT Core with Mosquitto on EC2. | 23/06/2025 | 23/06/2025 | Defined two ingestion approaches and evaluated operations, connection security, cost, and AWS integration. |
| Tuesday | Studied Kinesis Data Firehose buffering, retry behavior, and S3 delivery. Designed `raw` and `processed` zones partitioned by station and time. | 24/06/2025 | 24/06/2025 | Completed the MQTT-to-S3 record flow and a traceable prefix convention. |
| Wednesday | Studied SageMaker Processing, DeepAR, and SageMaker Endpoint. Coordinated with Data/ML members on input data, the 24–48-hour horizon, and API output. | 25/06/2025 | 25/06/2025 | Drafted a forecast request/response contract without coupling the API to model-training details. |
| Thursday | Designed FastAPI on EC2 with `health`, `forecast`, `subscribe`, and `alert` APIs. Studied SNS subscription confirmation and threshold-based email alerts. | 26/06/2025 | 26/06/2025 | Defined backend responsibilities and the business flow from a user request to SNS. |
| Friday | Reviewed IAM, VPC/Security Groups, CloudWatch, CloudTrail, and AWS Budgets. Considered MQTT disconnects, Firehose retries, SageMaker timeouts, and pending SNS subscriptions. | 27/06/2025 | 27/06/2025 | Completed the logical architecture with security, monitoring, failure-handling, and cost controls. |

### Achievements

- Described the end-to-end architecture: OpenAQ/Python simulator → MQTT → IoT Core or Mosquitto → Firehose → S3 → SageMaker → FastAPI → SNS.
- Distinguished the purposes of raw data, processed data, and the inference endpoint.
- Drafted telemetry and forecast contracts that allowed workstreams to develop independently.
- Defined the core APIs and explicit error states.
- Established a minimum operational approach based on least privilege, restricted ports, and early budget monitoring.
