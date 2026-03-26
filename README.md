|---|---------|-------------|
| 1 | [Terraform VPC Infrastructure](#1-terraform-vpc-infrastructure) | IaC, networking, resource provisioning |
| 2 | [EC2 Hardening & AMI Creation](#2-ec2-hardening-and-ami-creation) | Linux admin, Bash, secure configuration |
| 3 | [AWS Monitoring & Security Pipeline](#3-aws-monitoring-and-security-pipeline) | Logging, alerting, IAM, threat detection |

---

### 1. Terraform VPC Infrastructure

Deploys a complete AWS network environment using Terraform — from a custom VPC and subnets through to a hardened EC2 instance, with routing, an internet gateway, and security groups all configured as code.

**Skills demonstrated:** Infrastructure as Code · AWS networking (VPC, subnets, routing) · Secure resource provisioning

→ [View project](./terraform-vpc/)

---

### 2. EC2 Hardening and AMI Creation

Launches a Linux EC2 instance, applies a security hardening baseline via Bash scripts (SSH lockdown, service minimization, logging), then snapshots the result as a reusable AMI for consistent future deployments.

**Skills demonstrated:** Linux system administration · Bash scripting · CIS-aligned hardening · AMI lifecycle management

→ [View project](./ec2-hardening/)

---

### 3. AWS Monitoring and Security Pipeline

Builds an end-to-end security observability pipeline: CloudTrail captures API activity account-wide, CloudWatch metric filters parse the log stream for high-signal events, and alarms deliver real-time SNS notifications when thresholds are breached.

**Skills demonstrated:** CloudTrail logging · CloudWatch metric filters & alarms · SNS alerting · IAM least-privilege · Threat simulation

→ [View project](./monitoring-pipeline/)

---

## Tech Stack

| Category | Tools |
|---|---|
| Cloud | AWS (EC2, VPC, IAM, CloudTrail, CloudWatch, SNS, S3) |
| IaC | Terraform |
| OS & Scripting | Linux (Ubuntu), Bash |
| Version Control | Git, GitHub |

---

## Purpose

These projects document a progression through foundational cloud engineering disciplines — provisioning infrastructure with code, hardening compute resources, and building security monitoring into the environment from the start. Each project is independently deployable and reflects patterns used in production cloud environments.

---

## Roadmap

- [ ] Add CI/CD pipeline for automated Terraform deployments (GitHub Actions)
- [ ] Refactor Terraform into reusable, parameterized modules
- [ ] Integrate CloudWatch alerts with a SIEM (e.g., Wazuh)
- [ ] Add a fourth project: S3 data pipeline with lifecycle policies and access logging
