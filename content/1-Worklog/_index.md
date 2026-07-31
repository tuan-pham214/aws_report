---
title: "Worklog"
weight: 1
chapter: false
pre: " <b> 1. </b> "
---

During my internship, I worked on the **Local Air Quality Forecasting and Alert System** as a Backend/API Engineer. The roadmap began with AWS fundamentals and architecture analysis, then progressed to API development, forecast integration, alert delivery, testing, monitoring, and project handover.

My internship lasted from **June 15, 2025 to August 15, 2025**. Because June 15 was a Sunday, the detailed worklog begins on Monday, June 16. The internship report had to be submitted before July 31, so the entries below document work completed through **July 30, 2025**. The period from July 31 to August 15 remained part of the internship but is not included in the submitted daily worklog.

The team implemented the following system flow: OpenAQ data or a Python telemetry simulator publishes data through MQTT to AWS IoT Core or Mosquitto on EC2; Kinesis Data Firehose delivers it to the `raw` and `processed` zones in Amazon S3; SageMaker Processing and DeepAR prepare and forecast the data; a SageMaker Endpoint serves inference; FastAPI on EC2 provides the application API; and Amazon SNS sends alerts. IAM, VPC and Security Groups, CloudWatch, CloudTrail, and AWS Budgets were used throughout deployment and operations.

The eight-week worklog is organized as follows:

1. [Week 1 – Building an AWS Foundation](1.1-week1/)
2. [Week 2 – AWS Services and Architecture for the AQI System](1.2-week2/)
3. [Week 3 – Requirements Analysis and Data Ingestion](1.3-week3/)
4. [Week 4 – Data Lake Organization and FastAPI Skeleton](1.4-week4/)
5. [Week 5 – Data Processing, Baseline Model, and Alert Subscription](1.5-week5/)
6. [Week 6 – Integrating Forecasting with FastAPI and Amazon SNS](1.6-week6/)
7. [Week 7 – Testing, Monitoring, Security, and Cost Optimization](1.7-week7/)
8. [Week 8 – Documentation, Demo, and Final Report](1.8-week8/)
