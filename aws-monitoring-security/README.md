# AWS Monitoring & Security Pipeline

> Centralized logging, real-time threat detection, and automated alerting on AWS — built to simulate production-grade security operations.

[![AWS](https://img.shields.io/badge/AWS-CloudTrail%20%7C%20CloudWatch%20%7C%20SNS%20%7C%20IAM-FF9900?logo=amazonaws&logoColor=white)](https://aws.amazon.com)
[![IaC](https://img.shields.io/badge/IaC-Terraform-7B42BC?logo=terraform&logoColor=white)](https://www.terraform.io)
[![License](https://img.shields.io/badge/License-MIT-blue.svg)](./LICENSE)

---

## Overview

This project implements a fully integrated AWS security monitoring pipeline that captures API activity across an entire AWS account, filters logs for security-relevant events, and delivers real-time alerts when thresholds are breached. It mirrors the kind of security observability work done in production cloud environments, from log ingestion to automated incident notification.

**Core capabilities:**
- Continuous API activity logging via CloudTrail with S3 persistence
- Structured log ingestion and querying through CloudWatch Logs
- Custom metric filters that isolate high-signal security events (failed logins, unauthorized calls, root usage, IAM changes)
- Threshold-based alarms that trigger SNS notifications via email or SMS
- IAM roles and policies built on strict least-privilege principles throughout

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                      AWS Account                        │
│                                                         │
│   API Calls → AWS CloudTrail → S3 Bucket (log archive)  │
│                    │                                    │
│                    ▼                                    │
│             CloudWatch Logs                             │
│                    │                                    │
│          ┌─────────┴──────────┐                         │
│          ▼                    ▼                         │
│   Metric Filters        Log Insights                    │
│          │              (ad-hoc queries)                │
│          ▼                                              │
│   CloudWatch Alarms                                     │
│          │                                              │
│          ▼                                              │
│   SNS Topic → Email / SMS / Lambda / Webhook            │
└─────────────────────────────────────────────────────────┘
```

---

## Tech Stack

| Service | Role |
|---|---|
| **AWS CloudTrail** | Captures all API calls across the account and writes to S3 |
| **AWS CloudWatch Logs** | Ingests CloudTrail logs for real-time filtering and querying |
| **AWS CloudWatch Alarms** | Triggers on metric threshold breaches and manages state transitions |
| **AWS IAM** | Enforces least-privilege access across all pipeline components |
| **AWS SNS** | Delivers real-time alerts via email, SMS, or downstream integrations |
| **Terraform** | Provisions and manages all infrastructure as code |

---

## Features

### CloudTrail Logging

CloudTrail captures every API call made in the account — management events, data events, and authentication activity and ships them to a dedicated S3 bucket with optional log file integrity validation enabled.

### CloudWatch Log Integration

Logs are streamed from CloudTrail into CloudWatch Log Groups, scoped per environment. This enables real-time filtering, long-term retention policies, and ad-hoc querying via CloudWatch Log Insights.

### Metric Filters & Alerting

Custom metric filters parse the log stream and emit numeric metrics whenever a security-relevant event pattern is matched. These feed directly into CloudWatch Alarms.

| Filter Name | Detects |
|---|---|
| `ConsoleLoginFailures` | Failed AWS Console sign-in attempts |
| `UnauthorizedAPICalls` | `AccessDenied` or `UnauthorizedOperation` errors |
| `RootAccountUsage` | Any API activity by the root account |
| `IAMPolicyChanges` | Policy creation, deletion, or attachment events |

**Example — failed console login filter pattern:**
```json
{ ($.eventName = "ConsoleLogin") && ($.errorMessage = "Failed authentication") }
```

### CloudWatch Alarms

Each metric filter feeds an alarm with a configurable threshold and evaluation period. Alarms cycle through `OK → ALARM → INSUFFICIENT_DATA` states and publish to an SNS topic on each transition.

**Example:** `ConsoleLoginFailures` alarm triggers after ≥ 3 failures within any 5-minute window.

### SNS Notifications

Alerts are delivered to subscribed endpoints (email, SMS) when an alarm fires. The SNS topic can be extended to trigger Lambda functions, Slack webhooks, or PagerDuty without changing the upstream pipeline.

---

## IAM Configuration

All roles and policies are built on **least-privilege** principles — no wildcard (`*`) actions in any custom policy:

- CloudTrail has write-only access scoped to the designated S3 bucket
- CloudWatch Logs role is scoped to specific log groups
- SNS publish permissions are restricted to the alarm execution role only
- No cross-service permissions are granted beyond what each component requires

---

## Setup & Deployment

### Prerequisites

- AWS CLI configured with credentials for the target account
- Terraform v1.3+
- An SNS-compatible email address or phone number for alert subscriptions

### Deploy

```bash
# 1. Clone the repository
git clone https://github.com/your-username/aws-monitoring-security.git
cd aws-monitoring-security

# 2. Set your variables
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars: set your AWS region, SNS email, S3 bucket name, etc.

# 3. Deploy infrastructure
terraform init
terraform plan
terraform apply
```

### Verify the Pipeline

After deployment, confirm each layer is healthy before testing alerts:

1. **CloudTrail** → Confirm the trail is active and writing to S3
2. **CloudWatch Logs** → Confirm the log group is receiving events
3. **CloudWatch Alarms** → Confirm all alarms are in `OK` state
4. **SNS** → Confirm your email subscription is confirmed (check inbox)

---

## Testing Scenarios

Simulate real security events to validate end-to-end alert delivery:

| Scenario | How to Simulate | Expected Result |
|---|---|---|
| Failed console login | Attempt login with a wrong password | `ConsoleLoginFailures` alarm triggers |
| Unauthorized API call | Call an API without required IAM permissions | `UnauthorizedAPICalls` alarm triggers |
| Root account use | Sign in as the root user | `RootAccountUsage` alarm triggers |
| IAM policy change | Attach or detach a policy via AWS CLI | `IAMPolicyChanges` alarm triggers |

---

## Key Learnings

- How CloudTrail integrates with CloudWatch Logs to enable real-time log analysis
- Writing CloudWatch metric filter patterns to isolate high-signal security events from noisy log streams
- Structuring IAM roles that enforce least-privilege across a multi-service pipeline
- Building an end-to-end alerting workflow from raw log event through to notification delivery

---


