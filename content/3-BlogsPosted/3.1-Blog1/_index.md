---
title: "Three Habits That Help AWS Beginners Avoid Unexpected Charges"
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
includeInReport: false
---

**Published article:** https://www.facebook.com/share/p/1BxtXE5ien/?

## Introduction

When learning AWS, beginners often create resources for a lab and then forget where those resources were created or whether they are still running. A single EC2 instance, public IPv4 address, NAT Gateway, endpoint, or log group may continue generating charges after the exercise is finished.

The most effective solution is not memorizing every price. It is developing a repeatable routine for creating, tracking, and cleaning up resources.

## Habit 1: Agree on a Region Before Creating Resources

AWS resources are normally viewed and managed within a Region. If a team creates EC2 in Singapore, a bucket in another Region, and an alarm somewhere else, the environment becomes difficult to audit and clean up.

Before beginning a lab, the team should choose a Region and verify it in both the Console and CLI. For this project, the development Region was `ap-southeast-1`.

Useful checks include:

```bash
aws configure get region
aws sts get-caller-identity
```

The Region should still be specified explicitly for important CLI commands. This makes scripts easier to review and reduces the risk of modifying a resource in the wrong location.

## Habit 2: Add Tags When Resources Are Created

Tags make it possible to identify the project, environment, owner, and planned cleanup time of a resource. Adding tags later is easy to forget, so they should be treated as part of resource creation.

A small student project can begin with the following set:

| Tag | Example purpose |
|---|---|
| `Project` | Groups resources belonging to one project |
| `Environment` | Distinguishes `dev`, `test`, and `demo` |
| `Owner` | Identifies the responsible person or team |
| `AutoCleanupDate` | Records when a temporary resource should be reviewed |

Tags do not stop charges by themselves. Their value is that they make resource inventory and cost allocation clearer.

## Habit 3: Check and Clean Up After Every Lab

Cleanup should be a standard final step rather than an optional task. A useful checklist includes:

- Stop or terminate temporary EC2 instances and confirm whether EBS volumes remain.
- Check public IPv4 addresses, Elastic IP addresses, load balancers, NAT Gateways, and VPC endpoints.
- Inspect S3 buckets, incomplete multipart uploads, and non-current object versions.
- Review CloudWatch log groups and retention periods.
- Check SageMaker endpoints, notebooks, processing jobs, and training jobs.
- Confirm whether Firehose streams, IoT rules, DynamoDB tables, or SNS subscriptions are still required.

Deletion order matters. Dependencies should be removed safely, and shared resources should never be deleted simply because they appear in a cleanup list.

## AWS Budgets as an Additional Warning Layer

AWS Budgets can notify a team when actual or forecast spending crosses a threshold. It is useful, but it is not an immediate circuit breaker. Billing data can be delayed, and a notification does not automatically remove the resource that caused the charge.

Budgets should therefore supplement, not replace, Region discipline, tagging, inventory, and cleanup.

## Lessons Learned

The three habits are simple:

1. Use one agreed Region.
2. Tag resources as soon as they are created.
3. Perform a cleanup review after every lab.

Following these habits helped the Local AQI team track EC2, S3, IoT Core, Firehose, SageMaker, DynamoDB, SNS, and CloudWatch resources throughout the internship.

## Official AWS Documentation

- [AWS Regions and Availability Zones](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-regions-availability-zones.html)
- [Tagging AWS resources](https://docs.aws.amazon.com/tag-editor/latest/userguide/tagging.html)
- [Managing costs with AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
