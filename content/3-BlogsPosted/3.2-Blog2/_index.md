---
title: "Reserved Concurrency vs. Provisioned Concurrency in AWS Lambda"
weight: 2
chapter: false
pre: " <b> 3.2. </b> "
includeInReport: false
---

**Published article:** https://www.facebook.com/share/p/1KpgVzwNtB/?

## Introduction

Reserved Concurrency and Provisioned Concurrency both contain the word “concurrency,” but they solve different problems. Reserved Concurrency controls how much concurrency a function can use and guarantees that capacity against other functions. Provisioned Concurrency pre-initializes execution environments to reduce cold-start latency.

## What Is Lambda Concurrency?

Concurrency is the number of requests that a Lambda function is processing at the same time. If one invocation runs for one second and ten requests arrive together, the function may need approximately ten concurrent execution environments.

Concurrency therefore affects both scalability and the load placed on downstream services such as databases or third-party APIs.

## Reserved Concurrency

Reserved Concurrency assigns a concurrency quota to a function. It has two effects:

- The function can use up to the configured number of concurrent executions.
- That amount is reserved from the account’s regional concurrency pool for the function.

This is useful when a function must be protected from noisy neighbors or when its downstream database must not receive more than a controlled number of concurrent requests.

### When to Use It

- Protecting a database or external API from traffic spikes.
- Reserving capacity for a critical function.
- Preventing one function from consuming all regional concurrency.

Reserved Concurrency does **not** pre-initialize environments and therefore does not eliminate cold starts.

## Provisioned Concurrency

Provisioned Concurrency keeps a configured number of execution environments initialized for a published version or alias. Requests handled within that provisioned amount can avoid most cold-start initialization work.

### When to Use It

- User-facing APIs with strict and predictable latency requirements.
- Functions with heavy framework or SDK initialization.
- Workloads with predictable peak periods.

Provisioned Concurrency has an additional cost while it is configured, even when the pre-initialized environments are not fully used. It should therefore be sized from measurements instead of guesswork.

## Quick Comparison

| Criterion | Reserved Concurrency | Provisioned Concurrency |
|---|---|---|
| Main purpose | Reserve and limit concurrency | Pre-initialize execution environments |
| Sets a maximum | Yes | Not by itself |
| Reduces cold starts | No | Yes, within provisioned capacity |
| Configuration scope | Function | Published version or alias |
| Additional configuration charge | No | Yes |
| Suitable for | Protecting functions and downstream systems | APIs requiring stable latency |

## Can They Be Used Together?

Yes. A function can use Reserved Concurrency to set a safe upper limit and Provisioned Concurrency on an alias to keep part of that capacity warm.

For example, Reserved Concurrency can be set to 50 while Provisioned Concurrency is set to 10. The first ten concurrent requests can use pre-initialized environments; additional requests can scale on demand up to the reserved limit.

## Example in an AQI Forecasting System

Suppose a Lambda function receives AQI forecast requests and calls a downstream inference service. Reserved Concurrency can protect the inference service from a sudden burst, while Provisioned Concurrency can reduce latency for the normal interactive workload.

The appropriate values should be selected from request volume, execution duration, latency targets, downstream capacity, and cost.

## Common Misunderstandings

- Reserved Concurrency does not remove cold starts.
- Provisioned Concurrency is not automatically a maximum limit.
- Provisioning an alias does not accelerate invocations sent to `$LATEST`.
- Provisioning more capacity is not always better because unused capacity still costs money.
- The two settings are complementary rather than mutually exclusive.

## Conclusion

Use Reserved Concurrency when the problem is capacity isolation or protection. Use Provisioned Concurrency when the problem is initialization latency. Combine them when a function requires both predictable latency and a controlled scaling boundary.

## Official AWS Documentation

- [Understanding Lambda function scaling](https://docs.aws.amazon.com/lambda/latest/dg/lambda-concurrency.html)
- [Configuring reserved concurrency](https://docs.aws.amazon.com/lambda/latest/dg/configuration-concurrency.html)
- [Configuring provisioned concurrency](https://docs.aws.amazon.com/lambda/latest/dg/provisioned-concurrency.html)
