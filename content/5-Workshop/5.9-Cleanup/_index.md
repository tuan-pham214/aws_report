---
title: "Resource Cleanup"
date: 2026-07-31
weight: 9
chapter: false
pre: " <b> 5.9. </b> "
---

#### Objective

Delete or stop resources that are no longer needed after completing the workshop to avoid unexpected charges.

{{% notice warning %}}
Only remove resources after backing up the required data, forecast results, model artifacts, and all screenshots needed for the report.
{{% /notice %}}

## 1. Delete the SageMaker Endpoint

An endpoint can continue generating charges while it remains `InService`. Delete the resources in this order:

1. SageMaker Endpoint.
2. Endpoint Configuration.
3. SageMaker Model that is no longer required.

```bash
aws sagemaker delete-endpoint --endpoint-name <ENDPOINT_NAME>
aws sagemaker delete-endpoint-config --endpoint-config-name <ENDPOINT_CONFIG_NAME>
aws sagemaker delete-model --model-name <MODEL_NAME>
```

## 2. Stop EC2 and Review Network Resources

- Stop or terminate the backend EC2 instance.
- Release any unused Elastic IP address.
- Review Security Groups, NAT Gateways, and VPC Endpoints.
- Delete a VPC only after all dependent resources have been removed.

![Review and delete an unused VPC](/images/5-Workshop/5.6-Cleanup/vpc.png)

## 3. Delete Firehose, the IoT Rule, and Certificates

1. Delete the IoT Rule that routes data to Firehose.
2. Deactivate and then delete the IoT Certificate.
3. Delete any IoT Policy that is no longer attached.
4. Delete the Firehose delivery stream.

Verify the resource name and Region before deletion so that resources belonging to another team are not affected.

## 4. Empty and Delete S3 Buckets

Back up any required data first:

```bash
aws s3 sync s3://local-aqi-dev-s3-raw ./backup/raw
aws s3 sync s3://local-aqi-dev-s3-processed ./backup/processed
```

Then remove all objects and delete the bucket:

```bash
aws s3 rm s3://local-aqi-dev-s3-raw --recursive
aws s3 rb s3://local-aqi-dev-s3-raw
```

![Delete data from an S3 bucket](/images/5-Workshop/5.6-Cleanup/delete-s3.png)

## 5. Delete the CloudFormation Stack

If resources were created by CloudFormation, delete the stack after handling buckets and other resources that retain data.

![Delete a CloudFormation stack](/images/5-Workshop/5.6-Cleanup/delete-stack.png)

## 6. Delete an Unused Hosted Zone

Review every record before deleting a Route 53 Hosted Zone.

![Delete a Route 53 Hosted Zone](/images/5-Workshop/5.6-Cleanup/delete-zone.png)

## 7. Delete SNS and Monitoring Resources

- Delete the project's SNS Subscriptions and SNS Topic.
- Delete CloudWatch Log Groups that no longer need to be retained.
- Delete test alarms and dashboards.
- Review log retention periods to avoid long-term charges.

## 8. Delete IAM Roles and Policies

Only delete a role after the services using it have been removed. Detach inline policies, managed policies, and the instance profile before deleting the role.

## 9. Final Check

- [ ] The SageMaker Endpoint has been deleted.
- [ ] EC2 has been stopped or terminated.
- [ ] Firehose and the IoT Rule have been deleted.
- [ ] S3 data has been backed up and deleted according to the plan.
- [ ] SNS, CloudWatch, and IAM have been reviewed.
- [ ] No unused Elastic IP, NAT Gateway, or other chargeable resource remains.
- [ ] AWS Budgets shows no unexpected charges.

After this step, the workshop environment no longer contains continuously running resources that could incur charges.
