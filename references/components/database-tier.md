# Database Tier

Lab4 implements a secure database access pattern. An RDS MySQL 8.0 instance (ecommerce-db) runs in private subnets and is accessed via a bastion host in a public subnet using AWS Systems Manager Session Manager. VPC endpoints for SSM, SSM Messages, and EC2 Messages enable Session Manager connectivity without traversing the public internet. A gateway VPC endpoint provides S3 access. A NAT gateway in the public subnet allows outbound internet access from private subnets. A CloudWatch alarm monitors RDS connection counts.

## Component Diagram

```graph
<components>
<id icon="database">{C1} - AWS::RDS::DBInstance ecommerce-db</id>
<id icon="server">{C2} - AWS::EC2::Instance Lab4 Bastion Host</id>
<id icon="shield">{C3} - AWS::EC2::SecurityGroup Lab4 RDS Security Group</id>
<id icon="shield">{C4} - AWS::EC2::SecurityGroup Lab4 Bastion Security Group</id>
<id icon="plug">{C5} - AWS::EC2::VPCEndpoint SSM Endpoint</id>
<id icon="plug">{C6} - AWS::EC2::VPCEndpoint SSM Messages Endpoint</id>
<id icon="plug">{C7} - AWS::EC2::VPCEndpoint EC2 Messages Endpoint</id>
<id icon="plug">{C8} - AWS::EC2::VPCEndpoint S3 Gateway Endpoint</id>
<id icon="arrow-up-right">{C9} - AWS::EC2::NatGateway NAT Gateway</id>
<id icon="list">{C10} - AWS::RDS::DBSubnetGroup DB Subnet Group</id>
<id icon="settings">{C11} - AWS::RDS::DBParameterGroup DB Parameter Group</id>
</components>
<connections>
<con>{C2} -> {C1} (mechanism="MySQL:3306", description="Bastion connects to RDS over private network")</con>
<con>{C3} -> {C4} (mechanism="Ingress", description="RDS SG allows inbound MySQL from bastion SG")</con>
<con>{C1} -> {C3} (mechanism="SecurityGroup", description="RDS instance uses RDS security group")</con>
<con>{C2} -> {C4} (mechanism="SecurityGroup", description="Bastion uses bastion security group")</con>
<con>{C1} -> {C10} (mechanism="Placement", description="RDS placed in DB subnet group spanning private subnets")</con>
<con>{C1} -> {C11} (mechanism="Configuration", description="RDS uses custom parameter group")</con>
<con>{C2} -> {C5} (mechanism="HTTPS", description="Bastion reaches SSM via VPC endpoint")</con>
<con>{C2} -> {C6} (mechanism="HTTPS", description="Bastion reaches SSM Messages via VPC endpoint")</con>
<con>{C2} -> {C7} (mechanism="HTTPS", description="Bastion reaches EC2 Messages via VPC endpoint")</con>
</connections>
```

## Components

| Component | Description | Type | Sample Resources |
|:----------|:------------|:-----|:-----------------|
| ecommerce-db | MySQL 8.0.45 RDS instance (db.t3.medium) in private subnets | AWS::RDS::DBInstance | ecommerce-db (ecommerce-db.c89ogu44iavq.us-east-1.rds.amazonaws.com:3306) |
| Lab4 Bastion Host | t3.micro EC2 instance in public subnet for database access via SSM | AWS::EC2::Instance | i-027c8f54b29aa0683 |
| Lab4 RDS Security Group | Allows MySQL (3306) inbound from bastion security group only | AWS::EC2::SecurityGroup | sg-07f7c6d12018fcfb7 |
| Lab4 Bastion Security Group | Security group for bastion host and VPC endpoints | AWS::EC2::SecurityGroup | sg-01738becb3264642b |
| SSM VPC Endpoint | Interface endpoint for AWS Systems Manager | AWS::EC2::VPCEndpoint | vpce-0589d0f6ee823a9ec |
| SSM Messages VPC Endpoint | Interface endpoint for SSM Messages (Session Manager) | AWS::EC2::VPCEndpoint | vpce-06ad1bb786dfec632 |
| EC2 Messages VPC Endpoint | Interface endpoint for EC2 Messages (Session Manager) | AWS::EC2::VPCEndpoint | vpce-0236b2dc4a10d62dd |
| S3 Gateway Endpoint | Gateway endpoint for S3 access from private subnets | AWS::EC2::VPCEndpoint | vpce-009d16451586562fe |
| NAT Gateway | Provides outbound internet access for private subnets | AWS::EC2::NatGateway | nat-083aa3972390c7c7b |
| DB Subnet Group | Spans Lab4-Private-Subnet-1 and Lab4-Private-Subnet-2 | AWS::RDS::DBSubnetGroup | devops-agent-combined-lab4dbsubnetgroup-lq3etj9n5lpu |
| DB Parameter Group | Custom MySQL 8.0 parameter group for ecommerce-db | AWS::RDS::DBParameterGroup | devops-agent-combined-lab4dbparametergroup-ekop58jstygn |

## Observability

| Name | Type | Linked Component |
|:-----|:-----|:-----------------|
| Lab4-RDS-HighConnections | CloudWatch Alarm | ecommerce-db (AWS/RDS DatabaseConnections > 100, dimension DBInstanceIdentifier=ecommerce-db) |
| RDSOSMetrics | CloudWatch Log Group | ecommerce-db (RDS Enhanced Monitoring OS-level metrics) |
| /aws/rds/instance/ecommerce-db/error | CloudWatch Log Group | ecommerce-db (MySQL error log) |
| AWS/RDS CPUUtilization | CloudWatch Metric | ecommerce-db (CPU, connections, storage, memory, IOPS, latency, throughput) |
