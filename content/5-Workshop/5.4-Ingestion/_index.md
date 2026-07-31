---
title : "Ingestion: Simulator -> IoT Core -> Firehose -> S3 Raw"
date : 2026-07-31
weight : 4
chapter : false
pre : " <b> 5.4. </b> "
---

#### Role objective

The Ingestion role is responsible for moving telemetry from the simulator into AWS and proving that real objects appear in S3 Raw.

This is the highest-priority MVP milestone, because Data Preparation, Machine Learning, and Backend cannot be considered complete without real raw data.

#### Target flow

```text
Simulator
-> AWS IoT Core
-> IoT Rule
-> Kinesis Data Firehose
-> S3 Raw
```

## Step-by-Step Deployment: Data Ingestion & Routing

### Step 1: Cloud Resource Verification

Ensure that the **Amazon Data Firehose** delivery stream has already been provisioned by the Data Engineering team.

Verify that Firehose is configured to:

* receive data in batches
* write data securely into the Amazon S3 bucket
* store data under the following prefix:

```text
raw/telemetry/
```

![Active Amazon Data Firehose stream used by the project](/images/5-Workshop/5.4-Ingestion/firehose-stream-active.png)

### Step 2: AWS IoT Core Security Configuration

Navigate to:

```text
AWS IoT Core
-> Security
-> Policies
```

Create a restrictive IoT Policy that only allows:

```text
iot:Connect
iot:Publish
```

The publish permission must be limited to the exact topic:

```text
telemetry/aqi/dev
```

#### Apply standard tags

Apply the project tag set so the resource can be tracked and managed consistently:

```text
Project: local-aqi-forecasting
Environment: dev
Owner: [Responsible member]
Module: ingestion
```

Then navigate to:

```text
AWS IoT Core
-> Security
-> Certificates
```

Create a new certificate and download:

* Root CA
* Private Key
* Device Certificate

After the certificate is created, attach the IoT Policy to it.

### Step 3: Configure message routing from IoT Rule to Firehose

Navigate to:

```text
AWS IoT Core
-> Message routing
-> Rules
```

Create a new IoT Rule, for example:

```text
Route_To_Firehose
```

#### SQL statement

Use the following SQL statement to capture all telemetry published into the topic:

```sql
SELECT * FROM 'telemetry/aqi/dev'
```

#### Rule action configuration

For the action, choose:

```text
Data Firehose stream
```

or:

```text
Send a message to a Data Firehose stream
```

Then select the previously created delivery stream.

#### IAM Role configuration

Select the existing IAM role or ask the administrator to create:

```text
local-aqi-dev-iot-to-firehose
```

This role must allow AWS IoT Core to perform:

```text
firehose:PutRecord
```

on the project Firehose delivery stream.

#### Trust relationship

The IAM role must include a Trust Policy that allows the following service principal to assume the role:

```text
iot.amazonaws.com
```

Example:

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

### Step 4: Prepare the local workspace

Clone the project repository onto the local machine:

```bash
git clone <repository-url>
```

Move the downloaded AWS certificates into:

```text
certs/
```

Make sure this folder is listed in `.gitignore` to avoid exposing credentials.

Example:

```gitignore
certs/
*.pem
*.key
*.crt
```

Place the cleaned historical dataset into:

```text
data/
```

It is recommended to use a virtual environment before installing dependencies.

Example:

```bash
python -m venv .venv
```

Activate the environment on Windows:

```bash
.venv\Scripts\activate
```

Activate the environment on Linux or WSL:

```bash
source .venv/bin/activate
```

Install the required libraries:

```bash
pip install paho-mqtt pandas
```

### Step 5: Run the simulator and verify the system

Run the simulator script to start streaming data:

```bash
python simulator.py
```

![Simulator sending records from multiple stations](/images/5-Workshop/5.4-Ingestion/simulator-records-sent.png)

![Telemetry message schema received through AWS IoT Core](/images/5-Workshop/5.4-Ingestion/message-schema.png)

#### Verify metrics

Navigate to:

```text
AWS Console
-> Amazon Data Firehose
-> Select the delivery stream
-> Monitoring
```

Or:

```text
AWS Console
-> CloudWatch
-> Metrics
-> Firehose
```

Check the following metrics:

```text
IncomingRecords
DeliveryToS3.Success
```

Use the following statistic:

```text
Sum
```

These metrics should confirm that data is flowing into the system.

![Firehose IncomingRecords metric](/images/5-Workshop/5.4-Ingestion/firehose-incoming-records.png)

![Firehose delivery success metric](/images/5-Workshop/5.4-Ingestion/firehose-delivery-metrics.png)

#### Verify data in S3

Open the destination S3 bucket and check whether Firehose has written batched JSON objects into the partitioned structure.

Example:

```text
raw/
└── year=2026/
    └── month=07/
        └── day=31/
            └── hour=10/
```

Firehose may wait until the buffer size or buffer interval threshold is reached before writing objects into S3. Because of that, data may not appear immediately after the simulator sends messages.

Example test result:

```text
Firehose received 37 records and successfully delivered one buffered object into Amazon S3.
```

#### Outcomes that must be proven

+ successful publish to the correct topic
+ the IoT Rule routes records to Firehose
+ Firehose receives records and successfully writes objects into S3 Raw
+ the payload stored in S3 matches the simulator output
+ the system is tested with multiple records instead of a single isolated message

## Achieved result

After completing the ingestion phase, the team reaches the most important MVP milestone:

```text
Simulator
-> AWS IoT Core
-> Firehose
-> S3 Raw
```

This creates the real upstream input required for the Data Preparation, Machine Learning, and Backend teams to continue.
