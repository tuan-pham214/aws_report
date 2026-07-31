---
title: "Access Policies and IAM Authorization"
date: 2026-07-31
weight: 8
chapter: false
pre: " <b> 5.8. </b> "
---

## Overview

In the **Local AQI Forecasting & Alert System** project, multiple AWS services need to exchange data with each other, including:

* AWS IoT Core receiving data from the simulator.
* Amazon Data Firehose delivering data into Amazon S3.
* Amazon SageMaker reading data, processing data, and training the model.
* Amazon EC2 running the API that serves forecast results.
* Amazon SNS sending air-quality alert emails.

To keep these components working safely, the project uses **AWS Identity and Access Management (IAM)** to control users, roles, and access permissions.

The main principle applied here is **Least Privilege**, meaning that each user or service is granted only the permissions required to complete its own tasks.

> **Note:** The root account must not be shared with project members. It should only be used for special account-level operations such as billing management, account recovery, or advanced security configuration.

---

## 1. IAM User Management

Each team member working on AWS is assigned a separate IAM User instead of sharing the root account.

Using separate IAM Users helps:

* identify which member performed each action
* reduce the risk of exposing root credentials
* disable or revoke access for individual members when needed
* control the exact service scope available to each person
* trace activity through AWS CloudTrail if configured

The project administrator account should have **Multi-Factor Authentication (MFA)** enabled for stronger security.

---

## 2. Role-based access inside the team

Permissions are granted based on each member's responsibility in the project.

### Administration and DevOps

The AWS administrator or DevOps owner handles:

* creating and managing IAM Users
* creating IAM Roles for AWS services
* configuring AWS Budgets
* monitoring active resources
* helping team members troubleshoot authorization issues
* revoking temporary permissions after setup is complete

### Data Engineering

Members responsible for data engineering need access to:

* AWS IoT Core
* Amazon Data Firehose
* Amazon S3
* Amazon CloudWatch Logs when debugging ingestion issues

### Machine Learning

Members responsible for Machine Learning need access to:

* Amazon SageMaker
* Amazon S3 Raw
* Amazon S3 Processed
* processing jobs, training jobs, and model deployment resources

### Backend

Members responsible for the backend need access to:

* Amazon EC2
* Amazon SNS
* Amazon S3 or SageMaker Endpoint, depending on how forecast results are retrieved
* CloudWatch Logs for application troubleshooting

Role-based access reduces the risk of one account changing resources outside its assigned module.

---

## 3. AWS IoT Policy for the simulator device

The simulator sends air-quality telemetry into AWS IoT Core over MQTT.

An AWS IoT Policy is attached to the simulator certificate to allow the device to:

* connect to AWS IoT Core
* publish data to the assigned topic
* avoid publishing freely to unrelated topics in the system

The topic used in the development environment is:

```text
telemetry/aqi/dev
```

Example AWS IoT Policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iot:Connect",
      "Resource": "arn:aws:iot:ap-southeast-1:<AWS_ACCOUNT_ID>:client/${iot:Connection.Thing.ThingName}"
    },
    {
      "Effect": "Allow",
      "Action": "iot:Publish",
      "Resource": "arn:aws:iot:ap-southeast-1:<AWS_ACCOUNT_ID>:topic/telemetry/aqi/dev"
    }
  ]
}
```

In the real policy, `<AWS_ACCOUNT_ID>` must be replaced with the project AWS account ID.

The policy does not use `"Resource": "*"` for publish access, which helps prevent the simulator from sending data outside the approved topic scope.

The simulator certificate is attached to the corresponding policy.

---

## 4. IAM Role for the AWS IoT Rule

The AWS IoT Rule receives data from the MQTT topic and forwards the records to Amazon Data Firehose.

To do this, the IoT Rule needs an IAM Role whose trust relationship allows AWS IoT Core to use the role.

Example trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "iot.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

The role is only granted permission to send records into the project's delivery stream.

Example permission policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "firehose:PutRecord",
        "firehose:PutRecordBatch"
      ],
      "Resource": "arn:aws:firehose:ap-southeast-1:<AWS_ACCOUNT_ID>:deliverystream/<FIREHOSE_STREAM_NAME>"
    }
  ]
}
```

This policy is restricted to the exact Firehose delivery stream instead of granting access to every stream in the account.

---

## 5. IAM Role for Amazon Data Firehose

Amazon Data Firehose needs an IAM Role to write telemetry data into the S3 Raw bucket.

The role used by the project follows the naming convention:

```text
local-aqi-dev-firehose-to-s3
```

Trust policy for Firehose:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "firehose.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Permission policy restricted to the raw-data bucket:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetBucketLocation",
        "s3:ListBucket"
      ],
      "Resource": "arn:aws:s3:::local-aqi-dev-s3-raw"
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:AbortMultipartUpload",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::local-aqi-dev-s3-raw/raw/telemetry/*"
    }
  ]
}
```

Firehose is only allowed to write into:

```text
raw/telemetry/
```

Restricting access to the prefix helps reduce the risk of Firehose writing into the wrong area of the bucket.

If Firehose writes logs to CloudWatch Logs, the role may also need permissions such as:

```json
{
  "Effect": "Allow",
  "Action": [
    "logs:PutLogEvents"
  ],
  "Resource": "arn:aws:logs:ap-southeast-1:<AWS_ACCOUNT_ID>:log-group:<LOG_GROUP_NAME>:log-stream:*"
}
```

---

## 6. SageMaker Execution Role

Amazon SageMaker needs an execution role to read input data, write processed outputs, and store training artifacts.

Project role name:

```text
local-aqi-dev-sagemaker-execution-role
```

Trust policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "sagemaker.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```

Main permissions granted to this role:

* read data from the S3 Raw bucket
* read data from the S3 Processed bucket
* write and delete data in the S3 Processed bucket
* perform the SageMaker actions required for Processing Jobs, Training Jobs, and Endpoints

Example S3 access policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::local-aqi-dev-s3-raw",
        "arn:aws:s3:::local-aqi-dev-s3-processed"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:GetObject"
      ],
      "Resource": [
        "arn:aws:s3:::local-aqi-dev-s3-raw/*",
        "arn:aws:s3:::local-aqi-dev-s3-processed/*"
      ]
    },
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:DeleteObject"
      ],
      "Resource": "arn:aws:s3:::local-aqi-dev-s3-processed/*"
    }
  ]
}
```

SageMaker must not be given write or delete access to the S3 Raw bucket. This helps preserve the original data collected from the ingestion pipeline.

---

## 7. Amazon SNS access permissions

Amazon SNS is used to send alerts when air-quality values exceed the configured threshold.

The backend application only needs permission to publish to the project SNS Topic.

Example policy:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:ap-southeast-1:<AWS_ACCOUNT_ID>:<SNS_TOPIC_NAME>"
    }
  ]
}
```

The policy should not grant administrative SNS actions such as:

* `sns:DeleteTopic`
* `sns:SetTopicAttributes`
* `sns:Subscribe`
* `sns:Unsubscribe`

This allows the application to send alerts without changing the SNS Topic configuration itself.

---

## 8. PassRole permission

Some members need to create or configure resources that use IAM Roles, for example:

* attaching a role to an IoT Rule
* attaching a role to Firehose
* selecting an execution role for SageMaker

In these cases, the IAM User needs `iam:PassRole`.

This permission should only apply to project roles:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": "iam:PassRole",
      "Resource": [
        "arn:aws:iam::<AWS_ACCOUNT_ID>:role/local-aqi-dev-firehose-to-s3",
        "arn:aws:iam::<AWS_ACCOUNT_ID>:role/local-aqi-dev-sagemaker-execution-role",
        "arn:aws:iam::<AWS_ACCOUNT_ID>:role/local-aqi-dev-iot-to-firehose"
      ]
    }
  ]
}
```

It is not recommended to grant:

```json
{
  "Action": "iam:PassRole",
  "Resource": "*"
}
```

because a user could then attach higher-privilege roles to services outside the approved control scope.

After configuration is completed, temporary IAM permissions such as `iam:CreateRole`, `iam:AttachRolePolicy`, or `iam:PassRole` should be revoked if they are no longer needed.

---

## 9. Resource naming convention

IAM Roles and policies follow the naming structure:

```text
<project>-<environment>-<service>-<purpose>
```

Examples:

```text
local-aqi-dev-firehose-to-s3
local-aqi-dev-sagemaker-execution-role
local-aqi-dev-iot-to-firehose
```

This convention helps:

* identify which project owns the resource
* distinguish development and production environments
* identify which service is using the role
* make search and cleanup easier

---

## 10. Resource tagging

Resources that support tagging are assigned common tags:

| Tag         | Example value |
| ----------- | ------------- |
| Project     | local-aqi-forecasting |
| Environment | dev |
| Owner       | Name of the responsible member |
| Module      | ingestion, data, ml, backend, or devops |

Example:

```text
Project     = local-aqi-forecasting
Environment = dev
Owner       = Quynh-Tam
Module      = ingestion
```

Tags are used to:

* track resources by project
* identify the responsible owner
* categorize cost
* support verification and cleanup
* avoid deleting resources from another project by mistake

---

## 11. Access verification

After configuring IAM Roles and policies, the team verifies permissions through real flows.

### Verify IoT Core to Firehose

The simulator publishes a message to:

```text
telemetry/aqi/dev
```

The IoT Rule forwards the message into Firehose.

If the IAM Role is missing permissions, the rule will not be able to call:

```text
firehose:PutRecord
```

### Verify Firehose to S3

After sending data, the team checks the bucket:

```bash
aws s3 ls s3://local-aqi-dev-s3-raw/raw/telemetry/ --recursive
```

If the Firehose role is working correctly, new objects should appear in the S3 Raw bucket.

### Verify SageMaker read/write access

Inside the SageMaker environment, the team verifies read access:

```bash
aws s3 ls s3://local-aqi-dev-s3-raw/
```

Then verifies the ability to write processed output:

```bash
aws s3 cp processed-data.csv \
s3://local-aqi-dev-s3-processed/processed/processed-data.csv
```

SageMaker must not be allowed to delete or modify original raw data in the S3 Raw bucket.

### Verify SNS

The backend or AWS CLI sends a test alert:

```bash
aws sns publish \
  --topic-arn arn:aws:sns:ap-southeast-1:<AWS_ACCOUNT_ID>:<SNS_TOPIC_NAME> \
  --subject "Local AQI Test Alert" \
  --message "This is a system test alert."
```

If the policy is correct and the subscription has been confirmed, the alert email should be delivered successfully.

---

## 12. Result

After the IAM and policy configuration is completed:

* team members do not share the root account
* the administrator account is protected with MFA
* the simulator can publish only to the project MQTT topic
* the IoT Rule can only send data to the specified Firehose delivery stream
* Firehose can only write data into the S3 Raw bucket
* SageMaker can read source data but cannot delete original raw data
* the backend can publish alerts only to the project SNS Topic
* temporary IAM permissions are revoked after setup
* resources are named and tagged consistently

Applying the **Least Privilege** principle reduces the risk of unauthorized access, limits the blast radius of credential exposure, and makes AWS resource management clearer and safer.
