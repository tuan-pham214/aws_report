---
title: "Week 3 Worklog"
weight: 3
chapter: false
pre: " <b> 1.3. </b> "
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

- Clarify functional and non-functional requirements and the AQI MVP scope.
- Standardize telemetry records and verify ingestion from the simulator to the raw data zone.
- Prepare a stable data contract for backend, Data Lake, and Data/ML integration.

### Daily Tasks

| Day | Task | Start Date | Completion Date | Result |
|---|---|---|---|---|
| Monday | Analyzed use cases for station telemetry, 24–48-hour forecasts, AQI threshold subscriptions, and alerts. Added latency, traceability, availability, and subscriber-data protection requirements. | 30/06/2025 | 30/06/2025 | Produced MVP requirements and acceptance criteria, separating mandatory functions from extensions. |
| Tuesday | Designed a telemetry schema containing `station_id`, UTC measurement time, PM2.5, temperature, humidity, and source metadata. Defined types, ranges, units, and missing or duplicate-record handling. | 01/07/2025 | 01/07/2025 | Completed the shared schema and valid/invalid integration examples. |
| Wednesday | Studied the OpenAQ API and coordinated development of a Python simulator for periods when public data was discontinuous. Designed environment- and station-specific MQTT topics without exposing credentials. | 02/07/2025 | 02/07/2025 | Produced a repeatable test source and routing convention. |
| Thursday | Coordinated verification of the AWS IoT Core rule or Mosquitto bridge to Firehose, including buffering, retries, and error records before S3 delivery. | 03/07/2025 | 03/07/2025 | Confirmed the ingestion path and completed a checklist for diagnosing records missing from S3. |
| Friday | Inspected raw objects and compared timestamps, partitions, schema, and source-station traceability. Updated integration documentation for ingestion, backend, and Data/ML. | 04/07/2025 | 04/07/2025 | Finalized the input contract and edge-case list for processing. |

### Achievements

- Finalized the MVP scope and acceptance criteria for forecasting, subscription, and alerting.
- Standardized telemetry schema, UTC timestamps, units, and station identifiers.
- Prepared a simulator so testing did not depend entirely on OpenAQ availability.
- Coordinated verification of the MQTT–Firehose–S3 flow and documented ingestion diagnostics.
- Supplied a clear backend data contract while coordinating, rather than claiming sole ownership of, the team’s entire data pipeline.
