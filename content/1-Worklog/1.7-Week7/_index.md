---
title: "Week 7 Worklog"
weight: 7
chapter: false
pre: " <b> 1.7. </b> "
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

- Test the backend’s main flows, edge cases, and integration failures.
- Complete monitoring, auditing, and security controls for the AWS environment.
- Review cost, remove unnecessary configuration, and finalize the operational checklist.

### Daily Tasks

| Day | Task | Start Date | Completion Date | Result |
|---|---|---|---|---|
| Monday | Expanded tests for PM2.5-to-AQI boundaries, invalid input, and OpenAPI responses. Tested duplicate subscriptions, SNS states, DynamoDB failures, SageMaker timeout/invalid schema, and alert cooldown. | 28/07/2025 | 28/07/2025 | Covered main flows, edge cases, and dependency failures while preventing duplicate alerts under concurrent requests. |
| Tuesday | Standardized application and CloudWatch logs; configured metrics/alarms for API 5xx, latency, Firehose failure, SageMaker errors, and SNS failures. Reviewed least-privilege IAM, Security Groups, SSM access, S3 encryption, and CloudTrail. | 29/07/2025 | 29/07/2025 | Completed monitoring, alerts, security/audit checks, and response guidance for important operational signals. |
| Wednesday | Reviewed AWS Budgets, cost tags, EC2 sizing, Firehose buffering, CloudWatch retention, and S3 lifecycle. Ran regression tests and finalized demo and handover checklists. | 30/07/2025 | 30/07/2025 | Completed cost optimization for the MVP and confirmed system stability for reporting. |

### Achievements

- Covered health, forecast, subscribe, alert, and important edge cases with tests.
- Mapped DynamoDB, SageMaker, and SNS failures to safe and consistent responses.
- Completed CloudWatch monitoring and CloudTrail auditing for high-risk points.
- Reviewed IAM, Security Groups, EC2 management, and verified that logs contained no sensitive data.
- Finalized cost controls for right-sizing, retention, S3 lifecycle, tagging, and AWS Budgets alerts.
