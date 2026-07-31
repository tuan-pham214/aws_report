---
title: "Week 5 Worklog"
weight: 5
chapter: false
pre: " <b> 1.5. </b> "
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

- Study data cleaning and baseline modeling to align forecast inputs and outputs with the Data/ML members.
- Build station- and threshold-based alert subscription.
- Test validation, idempotency, and Amazon SNS subscription states.

### Daily Tasks

| Day | Task | Start Date | Completion Date | Result |
|---|---|---|---|---|
| Monday | Coordinated removal of invalid records, missing-data handling, time resampling, and processed-dataset creation. Reviewed PM2.5, temperature, humidity, and station history. | 14/07/2025 | 14/07/2025 | Completed the processed dataset and agreed on forecast-input requirements. |
| Tuesday | Coordinated evaluation of last-value and moving-average baselines against DeepAR using MAE/RMSE. Agreed on horizon, source time, forecast time, and required response values. | 15/07/2025 | 15/07/2025 | Completed baseline evaluation, selected DeepAR, and finalized the ML/backend forecast contract. |
| Wednesday | Built `POST /subscribe` for email, `station_id`, and `threshold_aqi`. Designed a stable subscriber identifier, DynamoDB state, and safe repeated requests. | 16/07/2025 | 16/07/2025 | Validated inputs and prevented duplicate records for identical requests. |
| Thursday | Integrated SNS email confirmation and distinguished `provisioning`, `pending`, `confirmed`, and `error` states. Designed station/threshold filtering for the MVP. | 17/07/2025 | 17/07/2025 | Completed subscription, email confirmation, and recipient filtering. |
| Friday | Added mocked AWS unit/integration tests for invalid emails, out-of-range thresholds, duplicates, DynamoDB/SNS errors, and concurrent requests. Updated OpenAPI. | 18/07/2025 | 18/07/2025 | Covered the main subscription flow and important failure cases. |

### Achievements

- Coordinated the processed dataset and baseline evaluation before DeepAR training.
- Finalized a forecast contract containing source time, horizon, PM2.5, and AQI values.
- Completed a validated, idempotent alert-subscription API.
- Managed subscription state in DynamoDB and SNS instead of application memory.
- Added safe error tests that do not expose sensitive AWS details.
