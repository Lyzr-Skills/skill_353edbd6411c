# IAM and Security

Global IAM roles and instance profiles provide identity and access management for all three lab workloads. Each lab EC2 instance has a dedicated instance profile and role with SSM managed instance core permissions for Session Manager access. Lambda functions have execution roles with basic execution policies. Lab4 includes an RDS enhanced monitoring role. Two KMS keys are provisioned in us-east-1 for encryption. An auto scaling service-linked role supports Lab2 ASG operations.

## Component Diagram

```graph
<components>
<id icon="key">{C1} - AWS::IAM::Role Lab2 EC2 Role</id>
<id icon="user">{C2} - AWS::IAM::InstanceProfile Lab2 EC2 Profile</id>
<id icon="key">{C3} - AWS::IAM::Role Lab2 Load Trigger Role</id>
<id icon="key">{C4} - AWS::IAM::Role Lab3 EC2 Role</id>
<id icon="user">{C5} - AWS::IAM::InstanceProfile Lab3 EC2 Profile</id>
<id icon="key">{C6} - AWS::IAM::Role Lab3 Delayed Alarm Role</id>
<id icon="key">{C7} - AWS::IAM::Role Lab3 VPC Flow Logs Role</id>
<id icon="key">{C8} - AWS::IAM::Role Lab4 Bastion Role</id>
<id icon="user">{C9} - AWS::IAM::InstanceProfile Lab4 Bastion Profile</id>
<id icon="key">{C10} - AWS::IAM::Role Lab4 RDS Monitoring Role</id>
<id icon="key">{C11} - AWS::IAM::Role Auto Scaling Service Role</id>
<id icon="lock">{C12} - AWS::KMS::Key KMS Keys</id>
</components>
<connections>
<con>{C2} -> {C1} (mechanism="IAM", description="Instance profile binds to EC2 role")</con>
<con>{C5} -> {C4} (mechanism="IAM", description="Instance profile binds to EC2 role")</con>
<con>{C9} -> {C8} (mechanism="IAM", description="Instance profile binds to bastion role")</con>
</connections>
```

## Components

| Component | Description | Type | Sample Resources |
|:----------|:------------|:-----|:-----------------|
| Lab2 EC2 Role | IAM role for Lab2 ASG instances, grants SSM access | AWS::IAM::Role | arn:aws:iam::194722413282:role/Lab2-EC2-Role |
| Lab2 EC2 Instance Profile | Instance profile binding Lab2 EC2 Role to instances | AWS::IAM::InstanceProfile | arn:aws:iam::194722413282:instance-profile/Lab2-EC2-Profile |
| Lab2 Load Trigger Role | Execution role for Lab2 Remote Load Trigger Lambda | AWS::IAM::Role | arn:aws:iam::194722413282:role/Lab2-RemoteLoadTrigger-Role |
| Lab3 EC2 Role | IAM role for Lab3 web server, grants SSM access | AWS::IAM::Role | arn:aws:iam::194722413282:role/devops-agent-combined-Lab3EC2Role-VTXRdsxYr7TP |
| Lab3 EC2 Instance Profile | Instance profile binding Lab3 EC2 Role to web server | AWS::IAM::InstanceProfile | arn:aws:iam::194722413282:instance-profile/devops-agent-combined-Lab3EC2InstanceProfile-yJx234RhpClC |
| Lab3 Delayed Alarm Role | Execution role for Lab3 Delayed Alarm Lambda | AWS::IAM::Role | arn:aws:iam::194722413282:role/devops-agent-combined-Lab3DelayedAlarmRole-pL3Vwqpx7E20 |
| Lab3 VPC Flow Logs Role | IAM role allowing VPC flow logs to write to CloudWatch Logs | AWS::IAM::Role | arn:aws:iam::194722413282:role/devops-agent-combined-Lab3VPCFlowLogsRole-y1XmbWd7jyYo |
| Lab4 Bastion Role | IAM role for bastion host with SSM and CloudWatch Agent access | AWS::IAM::Role | arn:aws:iam::194722413282:role/devops-agent-combined-Lab4BastionRole-tMAEAS5DieZt |
| Lab4 Bastion Instance Profile | Instance profile binding Lab4 Bastion Role to bastion host | AWS::IAM::InstanceProfile | arn:aws:iam::194722413282:instance-profile/devops-agent-combined-Lab4BastionInstanceProfile-XNkKhPdvqOwZ |
| Lab4 RDS Monitoring Role | IAM role for RDS Enhanced Monitoring to publish OS metrics | AWS::IAM::Role | arn:aws:iam::194722413282:role/devops-agent-combined-Lab4RDSMonitoringRole-hU966DB2paOX |
| Auto Scaling Service Role | Service-linked role for Auto Scaling operations | AWS::IAM::Role | arn:aws:iam::194722413282:role/aws-service-role/autoscaling.amazonaws.com/AWSServiceRoleForAutoScaling |
| KMS Key 1 | KMS encryption key | AWS::KMS::Key | e91eefe5-6e30-471b-a192-f696fec5ffa1 |
| KMS Key 2 | KMS encryption key | AWS::KMS::Key | 5904f168-1634-4d0b-8604-b2a03f91eaa8 |

## Observability

| Name | Type | Linked Component |
|:-----|:-----|:-----------------|
| RDSOSMetrics | CloudWatch Log Group | Lab4 RDS Monitoring Role (Enhanced Monitoring publishes OS-level metrics) |
