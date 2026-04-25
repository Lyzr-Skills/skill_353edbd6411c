---
name: understanding-agent-space
description: Provides architectural context for the DevOps Agent Workshop environment. Use this skill when investigating incidents, understanding resource relationships, identifying which lab workload a resource belongs to, or tracing traffic flows across the workshop infrastructure in AWS account 194722413282.
metadata:
  alt-name: understanding-devops-agent-workshop
---

# DevOps Agent Workshop

A single-account AWS workshop environment hosting three lab workloads that demonstrate web application load balancing (Lab2), network security monitoring (Lab3), and database access patterns (Lab4). All labs share a common VPC (10.10.0.0/16) in us-east-1 and are deployed via a single CloudFormation stack (`devops-agent-combined`).

## Environments

- **Workshop**: AWS account 194722413282, primary workloads in us-east-1. us-east-2 contains only AWS default resources (default VPC, default parameter groups) with no active workloads. Global scope holds IAM roles and instance profiles supporting all three labs.

## Architecture

```graph
<components>
  <id icon="server">{C1} - Web Application Tier</id>
  <id icon="shield-check">{C2} - Network Security Lab</id>
  <id icon="database">{C3} - Database Tier</id>
  <id icon="network">{C4} - Workshop Networking</id>
  <id icon="key">{C5} - IAM and Security</id>
</components>
<connections>
  <con>{C1} -> {C4} (mechanism="VPC", description="ALB and EC2 instances deployed in Workshop VPC public subnets")</con>
  <con>{C2} -> {C4} (mechanism="VPC", description="Web server in public subnet, VPC flow logs capture network traffic")</con>
  <con>{C3} -> {C4} (mechanism="VPC", description="RDS in private subnets, bastion in public subnet, NAT and VPC endpoints for connectivity")</con>
  <con>{C1} -> {C5} (mechanism="IAM", description="EC2 instance profile and Lambda execution role")</con>
  <con>{C2} -> {C5} (mechanism="IAM", description="EC2 instance profile, Lambda role, and VPC flow logs role")</con>
  <con>{C3} -> {C5} (mechanism="IAM", description="Bastion instance profile and RDS monitoring role")</con>
</connections>
```

| Container | Description | Key Components | Reference |
|:----------|:------------|:---------------|:----------|
| Web Application Tier | Lab2 load-balanced web application with auto scaling EC2 instances behind an ALB, plus a Lambda function for load testing | ALB (ELBv2 LoadBalancer), Target Group (ELBv2 TargetGroup), Auto Scaling Group (AutoScaling), EC2 Instance (EC2), Launch Template (EC2), Load Trigger Lambda (Lambda) | Read [./references/components/web-application-tier.md](./references/components/web-application-tier.md) |
| Network Security Lab | Lab3 standalone web server with VPC flow log monitoring and a delayed alarm Lambda function | Web Server (EC2 Instance), VPC Flow Log (EC2 FlowLog), Flow Logs Log Group (CloudWatch Logs), Delayed Alarm Lambda (Lambda) | Read [./references/components/network-security-lab.md](./references/components/network-security-lab.md) |
| Database Tier | Lab4 RDS MySQL database accessed via a bastion host using SSM Session Manager, with VPC endpoints for private connectivity | RDS Instance (RDS), Bastion Host (EC2 Instance), SSM VPC Endpoints (EC2 VPCEndpoint), NAT Gateway (EC2 NatGateway), DB Subnet Group (RDS) | Read [./references/components/database-tier.md](./references/components/database-tier.md) |
| Workshop Networking | Shared VPC infrastructure (10.10.0.0/16) providing public and private subnets, internet gateway, NAT gateway, and route tables for all three labs | Workshop VPC (EC2 VPC), Public Subnets (EC2 Subnet), Private Subnets (EC2 Subnet), Internet Gateway (EC2), NAT Gateway (EC2), Route Tables (EC2) | Read [./references/components/workshop-networking.md](./references/components/workshop-networking.md) |
| IAM and Security | IAM roles, instance profiles, and KMS keys supporting all lab workloads | Lab2 EC2 Role (IAM Role), Lab3 EC2 Role (IAM Role), Lab4 Bastion Role (IAM Role), RDS Monitoring Role (IAM Role), KMS Keys (KMS) | Read [./references/components/iam-and-security.md](./references/components/iam-and-security.md) |

## Critical Paths

| Path | Description | Key Components | Reference |
|:-----|:------------|:---------------|:----------|
| ALB Web Traffic Flow | End-to-end HTTP request flow from the internet through the ALB to auto-scaled EC2 instances in Lab2 | Web Application Tier, Workshop Networking | [./references/critical-paths/alb-web-traffic-flow.md](./references/critical-paths/alb-web-traffic-flow.md) |
| Bastion to Database Flow | Secure database access from an operator through SSM Session Manager and a bastion host to the RDS MySQL instance in Lab4 | Database Tier, IAM and Security, Workshop Networking | [./references/critical-paths/bastion-to-database-flow.md](./references/critical-paths/bastion-to-database-flow.md) |
