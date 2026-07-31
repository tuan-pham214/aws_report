---
title: "Test the API and Alert Cycle"
weight: 4
chapter: false
pre: " <b> 5.7.4 </b> "
---

#### Test the Backend from Your Computer

Get the current public IPv4 address of the EC2 instance. It can change after the instance is stopped and started.

```bash
BASE_URL=http://<PUBLIC_IP>:8000
curl -i "$BASE_URL/health"
```

Open Swagger UI at:

```text
http://<PUBLIC_IP>:8000/docs
```

#### Test the Forecast API

```bash
curl -i "$BASE_URL/forecast/station-01"
```

When the SageMaker Endpoint and station data are configured correctly, the response includes:

- Source-data timestamp.
- Forecast timestamp.
- Forecast horizon.
- Forecast PM2.5 value.
- AQI value.

If the endpoint or source data is unavailable, the API returns HTTP `503` instead of producing a fake forecast.

#### Test Alert Subscription

```bash
curl -i -X POST "$BASE_URL/subscribe/" \
  -H 'Content-Type: application/json' \
  -d '{
    "email": "APPROVED_TEST_EMAIL",
    "station_id": "station-01",
    "threshold_aqi": 150
  }'
```

Amazon SNS sends a confirmation email. Open it and choose **Confirm subscription**.

Only use an address whose owner has consented to receive email. Do not commit real email addresses to the repository or expose them in logs or screenshots.

#### Test Alert Delivery

```bash
curl -i -X POST "$BASE_URL/alert/" \
  -H 'Content-Type: application/json' \
  -d '{
    "station_id": "station-01",
    "pm25": 55.4
  }'
```

The backend performs these steps:

1. Converts PM2.5 to AQI.
2. Finds confirmed subscribers.
3. Checks the alert threshold.
4. Checks the cooldown period.
5. Publishes an Amazon SNS notification when all conditions are met.

#### Configure the Automated Cycle

Enable the timer only after the SageMaker Endpoint, station data, and IAM permissions have been verified.

```bash
cd /opt/local-aqi-backend

sudo install -m 0644 deploy/local-aqi-forecast-cycle.service \
  /etc/systemd/system/

sudo install -m 0644 deploy/local-aqi-forecast-cycle.timer \
  /etc/systemd/system/

sudo systemctl daemon-reload
sudo systemctl enable --now local-aqi-forecast-cycle.timer
```

Review the schedule and the latest run:

```bash
sudo systemctl list-timers local-aqi-forecast-cycle.timer
sudo systemctl status local-aqi-forecast-cycle.timer --no-pager
sudo systemctl status local-aqi-forecast-cycle.service --no-pager
sudo journalctl -u local-aqi-forecast-cycle.service -n 100 --no-pager
```

#### Completion

You have completed the following tasks:

- Deployed the FastAPI backend to Amazon EC2.
- Granted access to DynamoDB, SageMaker, and SNS through an IAM role.
- Started the application with `systemd`.
- Tested the health, forecast, subscribe, and alert APIs.
- Configured the automated forecast and alert cycle.

Stop EC2 and the SageMaker Endpoint when the demo is not in use to limit charges.
