# AWS VPC Infrastructure — Terraform

## Overview
Deployed a secure AWS VPC environment using Terraform (Infrastructure as Code),
including networking components and a hardened EC2 instance with restricted access.

## Architecture
- Custom VPC (10.0.0.0/16)
- Public subnet with Internet Gateway
- Route table configured for internet access
- Security group — SSH restricted to my IP only
- EC2 instance (Ubuntu 22.04) deployed inside VPC

## Technologies
- AWS (EC2, VPC, Security Groups, Internet Gateway)
- Terraform
- Git

## Prerequisites
- Terraform installed
- AWS CLI configured (`aws configure`)
- Valid AWS credentials with EC2/VPC permissions

## Deployment
```bash
# Initialize
terraform init

# Preview changes
terraform plan

# Deploy
terraform apply

# Tear down
terraform destroy
```

## What This Demonstrates
- Infrastructure as Code (repeatable, version-controlled deployments)
- AWS networking fundamentals (VPC, subnets, routing)
- Security best practices (least-privilege security groups)
- Automated provisioning vs manual console clicks

## Screenshots
- terraform-apply.png — successful deployment output
- ec2-running.png — instance running inside VPC
<img width="1710" height="1112" alt="Screenshot 2026-03-24 at 11 41 55 PM" src="https://github.com/user-attachments/assets/9684567b-a081-45d0-86db-5643e14b4a83" />
- aws-vpc.png — VPC visible in AWS console
<img width="1710" height="1112" alt="Screenshot 2026-03-27 at 9 56 01 AM" src="https://github.com/user-attachments/assets/8250e983-96ae-4cf2-8825-9dfc76eea63c" />

