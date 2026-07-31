---
title: "Install and Configure the Backend Service"
weight: 2
chapter: false
pre: " <b> 5.7.2 </b> "
---

#### Connect to EC2

1. Open **Amazon EC2**.
2. Select `local-aqi-dev-ec2-backend`.
3. Choose **Connect**.
4. Select the **Session Manager** tab.
5. Choose **Connect**.

Check the Python version and install the required tools:

```bash
python3 --version
sudo dnf install -y python3-pip git rsync
```

#### Download the Source Code

```bash
cd /home/ec2-user
git clone <REPOSITORY_URL> AWS-FCJ-local_aqi_forecast
cd AWS-FCJ-local_aqi_forecast
```

Run the bootstrap script to create the application directory and `systemd` service:

```bash
sudo bash backend/deploy/ec2-user-data.sh
```

Synchronize the backend source to `/opt/local-aqi-backend`:

```bash
sudo rsync -a --delete \
  --exclude '.env' \
  --exclude '.venv' \
  backend/ /opt/local-aqi-backend/

sudo chown -R ec2-user:ec2-user /opt/local-aqi-backend
```

#### Install Dependencies

```bash
cd /opt/local-aqi-backend
python3 -m venv .venv
source .venv/bin/activate
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

#### Configure Environment Variables

```bash
cp .env.example .env
chmod 600 .env
nano .env
```

Update the file:

```dotenv
AWS_REGION=ap-southeast-1
SUBSCRIBERS_TABLE=local-aqi-subscribers-dev
ALERTS_TOPIC_ARN=arn:aws:sns:ap-southeast-1:ACCOUNT_ID:local-aqi-alerts-dev

SAGEMAKER_ENDPOINT_NAME=ENDPOINT_NAME
SAGEMAKER_CONTENT_TYPE=application/json
SAGEMAKER_TIMEOUT_SECONDS=5
SAGEMAKER_MAX_ATTEMPTS=3
FORECAST_HORIZON_HOURS=24

STATION_DATA_FILE=/opt/local-aqi-backend/runtime/stations.json
ALLOW_SAMPLE_STATION_DATA=false
ALERT_COOLDOWN_SECONDS=3600
```

Replace `ACCOUNT_ID` and `ENDPOINT_NAME` with actual values. Configure `SAGEMAKER_ENDPOINT_NAME` only after the endpoint reaches `InService` and its request/response format has been agreed with the ML team.

Do not store AWS access keys, secret keys, or personal email addresses in `.env`.

#### Validate the Source Code

```bash
python -m unittest discover -s tests -v
python -m compileall -q app tests
python scripts/validate_openapi.py
```

Continue when all checks finish successfully.
