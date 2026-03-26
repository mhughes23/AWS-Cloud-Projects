# AWS Monitoring & Security Pipeline

> Centralized logging, real-time threat detection, and automated alerting on AWS — built to simulate production-grade security operations.

---

## Overview

This project implements a fully integrated AWS security monitoring pipeline using **CloudTrail**, **CloudWatch**, **IAM**, and **SNS**. It demonstrates how to detect suspicious activity, enforce least-privilege access, and trigger automated alerts — mirroring real-world cloud security workflows.

---

## Architecture

```
AWS CloudTrail
      │
      ▼
CloudWatch Logs
      │
      ▼
Metric Filters  ──►  CloudWatch Alarms  ──►  SNS Notifications
```

---

## Tech Stack

| Service | Purpose |
|---|---|
| AWS CloudTrail | API activity logging across the account |
| AWS CloudWatch | Log ingestion, metric filters, and alarms |
| AWS IAM | Access control and least-privilege enforcement |
| AWS SNS | Real-time alert notifications |

---

## Features

### 1. CloudTrail Logging
- Captures all API calls across the AWS account
- Tracks user actions, resource changes, and authentication events
- Stores logs to S3 with optional integrity validation

### 2. CloudWatch Log Integration
- Centralized, real-time log ingestion from CloudTrail
- Log groups scoped per environment for easy querying and retention control

### 3. Metric Filters & Alerting
Filters parse log data to detect security-relevant events and feed CloudWatch Alarms:

| Filter | Detects |
|---|---|
| `ConsoleLoginFailures` | Failed AWS Console sign-in attempts |
| `UnauthorizedAPICalls` | `AccessDenied` / `UnauthorizedOperation` errors |
| `RootAccountUsage` | Any activity by the root account |
| `IAMPolicyChanges` | Policy creation, deletion, or attachment |

**Example filter pattern — failed console logins:**
```json
{ ($.eventName = "ConsoleLogin") && ($.errorMessage = "Failed authentication") }
```

### 4. CloudWatch Alarms
- Alarm triggers when metric threshold is breached (e.g., ≥3 failed logins in 5 minutes)
- Alarms transition through `OK → ALARM → INSUFFICIENT_DATA` states
- Each alarm is linked to an SNS topic for downstream notification

### 5. SNS Notifications
- Alerts delivered via email and/or SMS on alarm state change
- Easily extensible to Lambda, Slack webhooks, or PagerDuty integrations

---

## IAM Configuration

Follows **least-privilege** principles:

- CloudTrail write-only access to the designated S3 bucket
- CloudWatch Logs role scoped to specific log groups
- SNS publish permissions restricted to the alarm execution role
- No wildcard (`*`) actions in any custom policy

---

## Setup & Deployment

### Prerequisites
- AWS CLI configured with appropriate credentials
- Terraform v1.3+ (or AWS Console access)
- An existing SNS topic or email subscription endpoint

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/your-username/aws-monitoring-security.git
cd aws-monitoring-security

# 2. Configure variables
cp terraform.tfvars.example terraform.tfvars
# Edit terraform.tfvars with your AWS region, SNS email, etc.

# 3. Deploy infrastructure
terraform init
terraform plan
terraform apply
```

### Verify Setup
1. Navigate to **CloudTrail** → confirm trail is active and logging to S3
2. Navigate to **CloudWatch Logs** → confirm log group is receiving events
3. Navigate to **CloudWatch Alarms** → confirm alarms are in `OK` state
4. Trigger a test alert by simulating a failed login or unauthorized API call

---

## Testing Scenarios

| Scenario | How to Simulate | Expected Result |
|---|---|---|
| Failed login | Attempt console login with wrong password | `ConsoleLoginFailures` alarm triggers |
| Unauthorized API | Call an API without required permissions | `UnauthorizedAPICalls` alarm triggers |
| Root account use | Sign in as root | `RootAccountUsage` alarm triggers |
| IAM policy change | Attach/detach a policy via CLI | `IAMPolicyChanges` alarm triggers |

---

## Project Structure

```
aws-monitoring-security/
├── terraform/
│   ├── main.tf              # Core infrastructure
│   ├── cloudtrail.tf        # Trail and S3 bucket config
│   ├── cloudwatch.tf        # Log groups, filters, and alarms
│   ├── iam.tf               # Roles and policies
│   ├── sns.tf               # Notification topics
│   └── variables.tf
├── policies/
│   ├── cloudtrail-s3.json   # S3 bucket policy for CloudTrail
│   └── cloudwatch-role.json # IAM role for CloudWatch Logs
├── docs/
│   └── architecture.png
└── README.md
```

---

## Key Learnings

- How CloudTrail integrates with CloudWatch Logs for real-time analysis
- Writing CloudWatch metric filter patterns to isolate security events
- Structuring IAM roles that follow least-privilege in a monitoring context
- Building an end-to-end alerting pipeline from log event to notification

---

