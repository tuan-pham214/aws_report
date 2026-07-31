---
title: "Week 4 Worklog"
weight: 4
chapter: false
pre: " <b> 1.4. </b> "
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

- Organize an Amazon S3 Data Lake with separate raw and processed data zones.
- Initialize a testable, configurable, and extensible FastAPI backend.
- Prepare a secure EC2 environment for API deployment and AWS integration.

### Daily Tasks

| Day | Task | Start Date | Completion Date | Result |
|---|---|---|---|---|
| Monday | Coordinated the design of `raw` and `processed` bucket prefixes, station/time partitions, object naming, and traceability metadata. Reviewed versioning, encryption, and lifecycle rules. | 07/07/2025 | 07/07/2025 | Established a shared Data Lake structure for batch processing without mixing original and cleaned data. |
| Tuesday | Reviewed S3 permissions for Firehose, SageMaker Processing, and the backend. Separated read/write permissions by role and avoided account-wide policies. | 08/07/2025 | 08/07/2025 | Completed a component access matrix and resource-scoped policies. |
| Wednesday | Initialized FastAPI modules for configuration, schemas, routers, and services. Added a health endpoint, environment-based settings, and automatic API documentation. | 09/07/2025 | 09/07/2025 | Started the backend skeleton successfully with `/health` and extension points for forecasting and alerting. |
| Thursday | Defined request/response schemas for `forecast`, `subscribe`, and `alert`; validated `station_id`, email, AQI threshold, and PM2.5 data; standardized safe errors. | 10/07/2025 | 10/07/2025 | Completed the initial OpenAPI contract and consistent 4xx/5xx behavior. |
| Friday | Deployed FastAPI on EC2 as a service using an IAM instance role and minimum Security Group rules. Checked startup logs and health after deployment. | 11/07/2025 | 11/07/2025 | Ran the backend on EC2 with a repeatable deployment process and post-update checklist. |

### Achievements

- Completed the logical S3 raw/processed design, including partitioning, encryption, and lifecycle management.
- Built a modular FastAPI backend with externalized configuration and a health endpoint.
- Standardized schemas and safe error responses for the core APIs.
- Applied an EC2 IAM role instead of embedding AWS keys in the application.
- Prepared the foundation for processed-data and alert-subscription integration.
