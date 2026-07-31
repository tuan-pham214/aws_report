---
title: "Project Coordination"
date: 2026-07-31
weight: 2
chapter: false
pre: "<b>2. </b>"
---

## 1. Objective

Manage progress, assign ownership, and track dependencies across modules so the team does not end up with isolated outputs that cannot be integrated into one working system.

## 2. GitHub Task Management

Tasks are split into the following tracks:

```text
Ingestion
Data
Machine Learning
Backend
DevOps
Integration
```

Each task includes:

- Task name
- Owner
- Primary owner
- Goal
- Priority
- Checklist
- Prerequisite
- Evidence
- Definition of Done
- Next task

The shared status flow is:

```text
Todo
In Progress
Blocked
Review
Done
```

![GitHub Project board for the Local AQI sprint](/images/5-Workshop/5.3-DevOps/github-project-board.png)

A task is only moved to `Done` when it has a real output and supporting evidence. If a task is blocked, the team must state clearly which person, resource, or dependency is causing the blockage.

![Example task with owner, goal and checklist](/images/5-Workshop/5.3-DevOps/github-task-example.png)

## 3. Dependency & Integration Coordination

The main dependency flow of the project is:

```text
Simulator
-> S3 Raw
-> S3 Processed
-> ML Dataset
-> Forecast Output
-> Backend
-> SNS
```

### 3.1. Ingestion -> Data

The two teams need to align on:

- MQTT topic
- telemetry payload
- telemetry schema v1
- UTC timestamps
- station IDs
- field names
- data types
- measurement units

### 3.2. Data -> Machine Learning

The two teams need to align on:

- input S3 URI
- processed dataset structure
- hourly data frequency
- missing-value handling rules
- duplicate-handling rules
- partition convention
- time range and station list

### 3.3. Machine Learning -> Backend

The two teams need to align on:

- model input
- forecast output
- forecast horizon
- station ID
- error response
- alert threshold
- how the backend reads forecast artifacts or calls the model

## 4. Progress Reporting Workflow

When updating a task, each member should answer:

```text
Which task are you working on?
What part has been completed?
What is the current output?
Where is the evidence stored?
What is currently blocked, and by whom or by which resource?
What is the next task?
Were any AWS resources created?
Did this create additional cost?
```

## 5. Documentation Coordination

Each member writes documentation for the module they own. The DevOps role is responsible for:

- keeping one shared format
- checking resource names and region consistency
- checking alignment between team inputs and outputs
- consolidating evidence
- reviewing the content before it is included in the report and demo

Documentation is grouped by content type:

- architecture overview
- telemetry schema
- processed dataset definition
- test plan
- demo script
- evidence checklist

![Shared project documentation structure](/images/5-Workshop/5.3-DevOps/project-docs-structure.png)

## 6. Shared Definition of Done

A task is only accepted when:

- the code or AWS resource is working
- there is a real test
- there is an expected output
- there are logs, screenshots, S3 objects, or API responses
- there is a rerun guide
- no credentials are exposed
- the related documentation has been updated

## 7. Outcomes Achieved

- Tasks are divided by track and dependency
- Each member has clear ownership and responsibility
- The handoff points between teams are clearly identified
- Blocked tasks are detected early
- Documentation and evidence are standardized before final consolidation
