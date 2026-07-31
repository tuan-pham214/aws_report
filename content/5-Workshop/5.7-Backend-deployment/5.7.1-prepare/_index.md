---
title: "Prepare EC2 and the IAM Role"
weight: 1
chapter: false
pre: " <b> 5.7.1 </b> "
---

#### Create an IAM Role for the Backend

1. Open the **AWS Management Console**.
2. Search for and open **IAM**.
3. In the left navigation pane, choose **Roles**.
4. Choose **Create role**.
5. For **Trusted entity type**, choose **AWS service**.
6. For **Use case**, choose **EC2**.
7. Choose **Next**.

Attach the `AmazonSSMManagedInstanceCore` policy so that the EC2 instance can be managed through AWS Systems Manager Session Manager.

Next, create the runtime policy for the backend:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "SubscriberTable",
      "Effect": "Allow",
      "Action": [
        "dynamodb:GetItem",
        "dynamodb:PutItem",
        "dynamodb:UpdateItem",
        "dynamodb:Scan"
      ],
      "Resource": "arn:aws:dynamodb:ap-southeast-1:ACCOUNT_ID:table/local-aqi-subscribers-dev"
    },
    {
      "Sid": "AlertTopic",
      "Effect": "Allow",
      "Action": [
        "sns:GetSubscriptionAttributes",
        "sns:ListSubscriptionsByTopic",
        "sns:Publish",
        "sns:Subscribe"
      ],
      "Resource": "arn:aws:sns:ap-southeast-1:ACCOUNT_ID:local-aqi-alerts-dev"
    },
    {
      "Sid": "ForecastEndpoint",
      "Effect": "Allow",
      "Action": "sagemaker:InvokeEndpoint",
      "Resource": "arn:aws:sagemaker:ap-southeast-1:ACCOUNT_ID:endpoint/ENDPOINT_NAME"
    }
  ]
}
```

Replace `ACCOUNT_ID` and `ENDPOINT_NAME` with values from the deployment environment.

Name the role:

```text
local-aqi-backend-ec2-role
```

Choose **Create role**.

#### Create a Security Group

1. Open **Amazon EC2**.
2. In the left navigation pane, choose **Security Groups**.
3. Choose **Create security group**.
4. Enter the name `local-aqi-backend-sg`.
5. Select the VPC used by the project.
6. Add the following **Inbound rule**:

| Type | Protocol | Port range | Source |
| --- | --- | --- | --- |
| Custom TCP | TCP | `8000` | My IP or the CIDR allowed for the demo |

Do not open SSH port `22`. Manage the server through Session Manager.

#### Launch EC2

1. In **Amazon EC2**, choose **Instances** and then **Launch instances**.
2. Enter the name `local-aqi-dev-ec2-backend`.
3. Select Amazon Linux as the AMI.
4. Select an instance type suitable for testing, such as `t3.micro`.
5. For **Key pair**, choose **Proceed without a key pair**.
6. Select the `local-aqi-backend-sg` Security Group.
7. Under **Advanced details**, select the instance profile for `local-aqi-backend-ec2-role`.
8. Require **IMDSv2**.
9. Configure encrypted `gp3` storage and delete the volume on termination.

Add these tags:

```text
Project=local-aqi-forecasting
Environment=dev
Owner=quang-tuan
Module=backend
```

Choose **Launch instance**. Wait until the instance is `Running` and appears in Systems Manager before continuing.
