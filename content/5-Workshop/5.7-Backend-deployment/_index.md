---
title: "Deploy the FastAPI Service"
weight: 7
chapter: false
pre: " <b> 5.7. </b> "
---

#### Overview

In this section, you will deploy the backend service of the **Local AQI Forecasting & Alert System** to Amazon EC2.

The FastAPI service performs the following functions:

- Exposes an API for checking system health.
- Accepts subscriptions for AQI alert thresholds.
- Stores subscription and cooldown state in Amazon DynamoDB.
- Invokes a SageMaker Endpoint to retrieve forecast results.
- Sends email alerts through Amazon SNS.

Backend flow:

```text
User
    → FastAPI on Amazon EC2
        → Amazon DynamoDB
        → Amazon SageMaker Endpoint
        → Amazon SNS
```

The backend uses the EC2 IAM role to access AWS services. The application runs under `systemd` so that it starts automatically with EC2 and restarts if the process fails.

#### Content

- [Prepare EC2 and the IAM role](5.7.1-prepare/)
- [Install and configure the backend](5.7.2-install/)
- [Run the backend with systemd](5.7.3-systemd/)
- [Test the API and alert cycle](5.7.4-test/)
