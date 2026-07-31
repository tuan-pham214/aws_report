---
title: "DynamoDB TTL Is Not a Timer"
weight: 3
chapter: false
pre: " <b> 3.3. </b> "
includeInReport: false
---

**Published article:** https://www.facebook.com/share/p/1Ehu9A9z6w/?

## Introduction

DynamoDB Time to Live (TTL) is useful for automatically removing data that is no longer needed. However, TTL is not a precise scheduler. When an item reaches its expiration timestamp, DynamoDB marks it as eligible for background deletion; the item is not guaranteed to disappear immediately.

## How DynamoDB TTL Works

TTL uses an attribute containing a Unix epoch timestamp in **seconds**. After TTL is enabled for that exact attribute name, DynamoDB evaluates items and later removes expired items in the background.

The important distinction is:

```text
expiration time reached ≠ item deleted immediately
```

Expired items may remain in the table for a period of time before the service deletes them.

## Why Expired Items May Still Appear

Query and Scan operate on items that still physically exist in the table. If background deletion has not occurred, an expired item may still be returned. Applications must not assume that “visible in DynamoDB” means “still valid for business use.”

## Applications Must Enforce Expiration

For correctness-sensitive logic, compare the expiration value with the current time:

```python
import time

def is_active(item: dict) -> bool:
    expires_at = int(item.get("expires_at", 0))
    return expires_at > int(time.time())
```

An even safer pattern is to include expiration in a conditional write. For example, an AQI alert cooldown lock can be acquired only when no lock exists or the previous lock has expired:

```text
attribute_not_exists(subscriber_id) OR cooldown_until <= :now
```

TTL can then remove old lock records later as housekeeping, while the conditional expression provides immediate business correctness.

## Filtering Expired Items

A Query or Scan can use a filter such as `expires_at > :now` to hide expired results from the application. The filter is applied after DynamoDB reads candidate items, so it improves result correctness but does not necessarily reduce all read work.

Whenever possible, design keys and access patterns so that expiration is part of the application decision rather than relying only on a broad Scan.

## Appropriate Uses for TTL

- Session or token records that no longer need to be retained.
- Temporary cache entries.
- Old telemetry or staging records governed by retention rules.
- Expired locks after application logic has already stopped honoring them.

## Inappropriate Uses for TTL

- Executing a task at an exact time.
- Sending a notification at a precise deadline.
- Releasing a lock without checking its timestamp.
- Implementing a scheduler or queue.

EventBridge Scheduler, Step Functions, SQS delay mechanisms, or application scheduling logic are more suitable when timing must be precise.

## AQI Alert Cooldown Example

The Local AQI backend prevents repeated notifications for the same station by storing a cooldown lock in DynamoDB. The alert request checks `cooldown_until` atomically. If the time is still in the future, the request is skipped. If the lock is missing or expired, the backend replaces it and publishes the alert.

The record also contains `expires_at` for TTL cleanup, but the alert decision never waits for DynamoDB to delete the record.

## Updating an Expired Item

Until an expired item is physically deleted, it can still be read or updated. A background deletion may also race with an update. Applications should use conditions or replace the item through a controlled workflow rather than assuming the old record has disappeared.

## TTL, DynamoDB Streams, and Global Tables

TTL deletions can appear in DynamoDB Streams as service deletions, which allows downstream processing. With Global Tables, deletion behavior and replicated write costs should be reviewed for every Region used by the table.

## Common Configuration Mistakes

- Writing milliseconds instead of seconds.
- Storing the timestamp as a String instead of a Number.
- Enabling TTL for a different attribute name.
- Assuming that enabling TTL deletes items immediately.
- Forgetting to verify TTL after restoring or recreating a table.
- Using TTL as a task scheduler.

## Conclusion

DynamoDB TTL is an asynchronous cleanup feature. Applications must decide whether an item is still valid by checking time in reads and conditional writes. Treating TTL as housekeeping rather than as a timer produces safer sessions, locks, and cooldown mechanisms.

## Official AWS Documentation

- [Using time to live in DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/TTL.html)
- [Condition expressions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html)
- [TTL and DynamoDB Streams](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/time-to-live-ttl-streams.html)
