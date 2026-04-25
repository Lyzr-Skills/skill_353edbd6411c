# Network Security Lab

Lab3 demonstrates network security monitoring with a standalone EC2 web server and VPC flow logging. The web server (Lab3-WebServer) is a t3.micro instance in a public subnet with a security group allowing common web ports. VPC flow logs capture all network traffic across the Workshop VPC and write to a CloudWatch Logs log group for analysis. A Lambda function (Lab3DelayedAlarmFunction) provides delayed alarm processing capabilities.

## Component Diagram

```graph
<components>
<id icon="server">{C1} - AWS::EC2::Instance Lab3 Web Server</id>
<id icon="shield">{C2} - AWS::EC2::SecurityGroup Lab3 Web Server Security Group</id>
<id icon="activity">{C3} - AWS::EC2::FlowLog VPC Flow Log</id>
<id icon="file-text">{C4} - AWS::Logs::LogGroup VPC Flow Logs Log Group</id>
<id icon="zap">{C5} - AWS::Lambda::Function Lab3 Delayed Alarm Function</id>
<id icon="file-text">{C6} - AWS::Logs::LogGroup Lambda Log Group</id>
</components>
<connections>
<con>{C1} -> {C2} (mechanism="SecurityGroup", description="Web server uses security group for traffic filtering")</con>
<con>{C3} -> {C4} (mechanism="CloudWatch Logs", description="Flow log writes captured VPC traffic to log group")</con>
<con>{C5} -> {C6} (mechanism="CloudWatch Logs", description="Lambda function logs to CloudWatch")</con>
</connections>
```

## Components

| Component | Description | Type | Sample Resources |
|:----------|:------------|:-----|:-----------------|
| Lab3 Web Server | t3.micro EC2 instance serving as a web server in a public subnet | AWS::EC2::Instance | i-0a306df99abdd3eba |
| Lab3 Web Server Security Group | Security group allowing common web ports (HTTP, HTTPS, SSH) | AWS::EC2::SecurityGroup | sg-01b29d83b5538cac4 |
| VPC Flow Log | Captures all network traffic for the Workshop VPC (vpc-022249fb5a023ef57) | AWS::EC2::FlowLog | fl-0e68b062938dc0828 |
| VPC Flow Logs Log Group | CloudWatch Logs destination for VPC flow log data | AWS::Logs::LogGroup | /aws/vpc/flowlogs/Lab3-networksec |
| Lab3 Delayed Alarm Function | Python 3.12 Lambda for delayed alarm processing | AWS::Lambda::Function | arn:aws:lambda:us-east-1:194722413282:function:devops-agent-combined-Lab3DelayedAlarmFunction-R8xeVqZg4JOi |
| Lambda Log Group | CloudWatch Logs for the Delayed Alarm Lambda function | AWS::Logs::LogGroup | /aws/lambda/devops-agent-combined-Lab3DelayedAlarmFunction-R8xeVqZg4JOi |

## Observability

| Name | Type | Linked Component |
|:-----|:-----|:-----------------|
| /aws/vpc/flowlogs/Lab3-networksec | CloudWatch Log Group | VPC Flow Log (all VPC network traffic records) |
| /aws/lambda/devops-agent-combined-Lab3DelayedAlarmFunction-R8xeVqZg4JOi | CloudWatch Log Group | Lab3 Delayed Alarm Function (Lambda execution logs) |
