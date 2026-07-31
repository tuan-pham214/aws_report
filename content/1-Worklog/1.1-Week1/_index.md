---
title: "Week 1 Worklog"
weight: 1
chapter: false
pre: " <b> 1.1. </b> "
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

- Understand the AWS shared responsibility model, Regions, Availability Zones, and fundamental service groups.
- Become familiar with the AWS Management Console, AWS CLI, IAM, and the rule of never storing access credentials in source code.
- Understand how EC2, VPC, Security Groups, S3, and monitoring services support the AQI backend.

### Daily Tasks

| Day | Task | Start Date | Completion Date | Result |
|---|---|---|---|---|
| Monday | Learned the working process, defined the scope of the **Local AQI Forecasting and Alert System**, and clarified my Backend/API Engineer role. Sketched the flow from telemetry sources to APIs and alerts. | 16/06/2025 | 16/06/2025 | Established an overall view of the project, backend inputs and outputs, and collaboration boundaries with the Data/ML members. |
| Tuesday | Studied AWS global infrastructure, Regions, Availability Zones, the shared responsibility model, and Compute, Storage, Networking, and Security services. | 17/06/2025 | 17/06/2025 | Distinguished AWS security responsibilities from those of the project team and selected a consistent development Region. |
| Wednesday | Practiced using the AWS Console and CLI; studied IAM users, roles, policies, MFA, and least privilege. Configured a local profile without committing access keys. | 18/06/2025 | 18/06/2025 | Performed read-only account and resource checks with the CLI and established safe credential-handling rules. |
| Thursday | Studied EC2, AMI, EBS, VPC, subnets, route tables, and Security Groups. Reviewed how FastAPI could run on EC2 and receive traffic only through required ports. | 19/06/2025 | 19/06/2025 | Explained the request path to the backend and how to restrict inbound rules instead of exposing administrative ports broadly. |
| Friday | Studied S3, CloudWatch, CloudTrail, and AWS Budgets; prepared a checklist for tagging, logging, auditing, and cost alerts. | 20/06/2025 | 20/06/2025 | Completed the common foundation used during later implementation weeks. |

### Achievements

- Understood the AWS concepts required to read and discuss the project architecture.
- Used the Console and CLI for fundamental tasks and learned why the EC2 backend should use an IAM role instead of static access keys.
- Explained the relationship between VPC, subnets, Security Groups, EC2, and S3 in the deployment environment.
- Created an initial checklist for tagging, logs, auditing, and budgets to reduce security and cost risks.
- Clearly defined my Backend/API responsibilities and the integration boundaries with the Data and ML workstreams.
