---
title: "Week 6 Worklog"
weight: 6
chapter: false
pre: " <b> 1.6. </b> "
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

- Complete FastAPI integration with the SageMaker Endpoint using the agreed Data/ML contract.
- Complete forecast retrieval, AQI conversion, threshold evaluation, and SNS alerting.
- Test the end-to-end path and handle configuration, timeout, and invalid-response failures.

### Daily Tasks

| Day | Task | Start Date | Completion Date | Result |
|---|---|---|---|---|
| Monday | Built the latest-observation provider by `station_id` and mapped records to the SageMaker payload. Validated timezone, PM2.5 units, temperature/humidity, and horizon. | 21/07/2025 | 21/07/2025 | Created a defined inference schema separated from FastAPI’s internal model. |
| Tuesday | Built a SageMaker Runtime client using the EC2 IAM role with bounded timeouts and retries. Coordinated endpoint name and response schema with the ML member. | 22/07/2025 | 22/07/2025 | FastAPI invoked the SageMaker Endpoint and received a contract-compliant result. |
| Wednesday | Completed `GET /forecast/{station_id}` with observation time, forecast time, horizon, forecast PM2.5, and AQI. Mapped missing configuration to 503, timeout to 504, and invalid output to 502. | 23/07/2025 | 23/07/2025 | Returned a consistent forecast schema and tested error responses. |
| Thursday | Completed the forecast–alert cycle: converted PM2.5 to AQI, selected confirmed subscribers for the station/threshold, published through SNS, and added a cooldown lock. | 24/07/2025 | 24/07/2025 | Delivered alerts to eligible recipients without duplicate sends for repeated processing. |
| Friday | Coordinated end-to-end testing from station data and SageMaker through FastAPI to an SNS alert email; reviewed logs by station, cycle status, and latency. | 25/07/2025 | 25/07/2025 | Confirmed the deployed forecasting and alerting cycle. |

### Achievements

- Completed the forecast API with an explicit output contract and schema validation.
- Integrated SageMaker Runtime through an IAM role with bounded timeout and retry behavior.
- Completed threshold evaluation and SNS alerting with cooldown protection.
- Coordinated verification of the endpoint, model schema, and forecast result with Data/ML members.
- Successfully tested the flow from station data to forecast and email alert.
