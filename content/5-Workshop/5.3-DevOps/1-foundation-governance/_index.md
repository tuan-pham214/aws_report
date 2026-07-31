---
title: "Foundation & Governance"
date: 2026-07-31
weight: 1
chapter: false
pre: "<b>1. </b>"
---

## 1. Objective

Establish a consistent AWS environment for the whole team, control cost generation, and ensure that all members deploy resources under the same architecture, region, and management conventions.

## 2. AWS Environment Standardization

The team standardized all resource deployment in:

```text
ap-southeast-1 - Asia Pacific (Singapore)
```

Shared requirements:

- Verify the correct AWS Account and region before creating resources.
- Do not deploy resources in other regions without coordination.
- Report the resource name, service, and purpose after creation.
- Use the same naming convention and tag convention.
- Use the approved AWS CLI profile or AWS Console account.

## 3. Budget & Cost Monitoring

AWS Budgets were configured to track both **actual cost** and **resource usage** during project implementation.

At the moment, the team is using two main budgets:

- `My Monthly Cost Budget`: tracks total monthly AWS cost with a `100 USD` limit.
- `Daily usage`: tracks resource usage in hourly units with a `0.2 hour` limit.

At the time of review, the monthly cost remained low compared with the configured limit. The budget was in `Healthy` status and no alert threshold had been exceeded.

![AWS Budgets overview](/images/5-Workshop/5.3-DevOps/aws-budget-overview.png)

### Cost alert thresholds

For the `My Monthly Cost Budget`, the following thresholds were configured:

```text
12.5%  -> Alert when actual cost exceeds 12.50 USD
25%    -> Alert when actual cost exceeds 25.00 USD
50%    -> Alert when actual cost exceeds 50.00 USD
85%    -> Alert when actual cost exceeds 85.00 USD
90%    -> Alert when actual cost exceeds 90.00 USD
100%   -> Alert when actual cost exceeds 100.00 USD
100%   -> Alert when forecasted cost exceeds 100.00 USD
```

These thresholds help the team detect abnormal resource usage early and respond before costs rise too far.

![AWS monthly budget and alert configuration](/images/5-Workshop/5.3-DevOps/aws-budget-alerts.png)

### Internal control rules

Based on the AWS Budgets thresholds, the team applies the following internal control process:

```text
From 12.5 USD -> Check which services are generating cost
From 25 USD   -> Review running resources and the responsible owner
From 50 USD   -> Limit the creation of non-essential resources
From 85 USD   -> Pause experimental resources
From 90 USD   -> Keep only resources required for the MVP
From 100 USD  -> Stop and inspect all chargeable resources
```

### Services that require close monitoring

The services most likely to generate noticeable cost include:

* Amazon EC2.
* SageMaker Processing and Training.
* SageMaker Endpoint.
* Amazon CloudWatch Logs.
* Data transfer.
* Any continuously running or time-based billed resources.

Before creating a resource that may generate cost, each member should report:

```text
Service:
Resource name:
Instance type or configuration:
Purpose:
Expected runtime:
Owner:
```

## 4. Service Quotas

Besides budget tracking, the team also needs to monitor Service Quotas to avoid being blocked during MVP implementation, especially for services such as SageMaker or resources limited by instance-type quota.

Early quota checks help the team:

- know in advance which resources can be created,
- avoid depending entirely on a live demo when quota is not ready,
- and prepare fallback options when necessary.

![SageMaker service quota status](/images/5-Workshop/5.3-DevOps/sagemaker-service-quota.png)

## 5. Architecture, Naming & Tag Convention

The agreed high-level architecture is:

```text
Simulator
-> AWS IoT Core
-> IoT Rule
-> Kinesis Data Firehose
-> S3 Raw
-> Data Processing
-> S3 Processed
-> SageMaker
-> FastAPI
-> SNS
```

The naming convention is:

```text
local-aqi-{environment}-{resource-purpose}
```

Examples:

```text
local-aqi-dev-s3-raw
local-aqi-dev-s3-processed
local-aqi-dev-firehose-to-s3
local-aqi-dev-sagemaker-execution-role
```

Minimum tag convention:

```text
Project=local-aqi
Environment=dev
Owner=<member-name>
ManagedBy=manual
CostCenter=student-project
```

![AWS resource tags used by the project](/images/5-Workshop/5.3-DevOps/aws-resource-tags.png)
