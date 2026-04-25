# Web Application Tier

Lab2 implements a load-balanced web application pattern. An Application Load Balancer (Lab2-ALB) receives HTTP traffic on port 80 and forwards it through a target group (Lab2-TG) to EC2 instances managed by an Auto Scaling Group (Lab2-ASG). The ASG uses a launch template to provision t3.micro instances across two availability zones. A CloudWatch alarm (Lab2-ASG-CPU-High) triggers when CPU utilization exceeds 60%. A standalone Lambda function (Lab2-RemoteLoadTrigger) exists for generating synthetic load against the ALB.

## Component Diagram

```graph
<components>
<id icon="globe">{C1} - AWS::ElasticLoadBalancingV2::LoadBalancer Lab2 ALB</id>
<id icon="list">{C2} - AWS::ElasticLoadBalancingV2::Listener Lab2 ALB Listener</id>
<id icon="target">{C3} - AWS::ElasticLoadBalancingV2::TargetGroup Lab2 Target Group</id>
<id icon="layers">{C4} - AWS::AutoScaling::AutoScalingGroup Lab2 ASG</id>
<id icon="server">{C5} - AWS::EC2::Instance Lab2 ASG Instance</id>
<id icon="file-text">{C6} - AWS::EC2::LaunchTemplate Lab2 Launch Template</id>
<id icon="shield">{C7} - AWS::EC2::SecurityGroup Lab2 ALB Security Group</id>
<id icon="shield">{C8} - AWS::EC2::SecurityGroup Lab2 EC2 Security Group</id>
<id icon="zap">{C9} - AWS::Lambda::Function Lab2 Remote Load Trigger</id>
</components>
<connections>
<con>{C1} -> {C2} (mechanism="HTTP:80", description="ALB listener receives inbound traffic")</con>
<con>{C2} -> {C3} (mechanism="HTTP", description="Listener forwards to target group")</con>
<con>{C3} -> {C5} (mechanism="HTTP", description="Target group routes to registered instances")</con>
<con>{C4} -> {C3} (mechanism="Registration", description="ASG registers instances with target group")</con>
<con>{C4} -> {C6} (mechanism="Template", description="ASG uses launch template to create instances")</con>
<con>{C4} -> {C5} (mechanism="Lifecycle", description="ASG manages instance lifecycle")</con>
<con>{C1} -> {C7} (mechanism="SecurityGroup", description="ALB uses ALB security group")</con>
<con>{C5} -> {C8} (mechanism="SecurityGroup", description="EC2 instances use EC2 security group")</con>
<con>{C8} -> {C7} (mechanism="Ingress", description="EC2 SG allows inbound from ALB SG")</con>
<con>{C9} -> {C1} (mechanism="HTTP", description="Lambda generates load against ALB")</con>
</connections>
```

## Components

| Component | Description | Type | Sample Resources |
|:----------|:------------|:-----|:-----------------|
| Lab2 ALB | Application Load Balancer receiving HTTP traffic on port 80 | AWS::ElasticLoadBalancingV2::LoadBalancer | arn:aws:elasticloadbalancing:us-east-1:194722413282:loadbalancer/app/Lab2-ALB/317aff2800384fc8 |
| Lab2 ALB Listener | HTTP listener on port 80 forwarding to target group | AWS::ElasticLoadBalancingV2::Listener | arn:aws:elasticloadbalancing:us-east-1:194722413282:listener/app/Lab2-ALB/317aff2800384fc8/7be1e25a1dbd3054 |
| Lab2 Target Group | Target group routing traffic to ASG-managed EC2 instances | AWS::ElasticLoadBalancingV2::TargetGroup | arn:aws:elasticloadbalancing:us-east-1:194722413282:targetgroup/Lab2-TG/f9c358a5d351c97f |
| Lab2 Auto Scaling Group | Manages EC2 instances across two AZs (us-east-1a, us-east-1b) | AWS::AutoScaling::AutoScalingGroup | Lab2-ASG |
| Lab2 ASG Instance | t3.micro EC2 instance launched by ASG | AWS::EC2::Instance | i-01277c92f4b9db1ea |
| Lab2 Launch Template | Defines instance configuration for ASG-launched instances | AWS::EC2::LaunchTemplate | lt-0e324f3e3928d41ac |
| Lab2 ALB Security Group | Allows inbound HTTP traffic to the ALB | AWS::EC2::SecurityGroup | sg-0a245138030676747 |
| Lab2 EC2 Security Group | Allows inbound traffic from ALB security group only | AWS::EC2::SecurityGroup | sg-0b0d5b84221d31120 |
| Lab2 Remote Load Trigger | Python 3.12 Lambda function for generating synthetic load against the ALB | AWS::Lambda::Function | arn:aws:lambda:us-east-1:194722413282:function:Lab2-RemoteLoadTrigger |

## Observability

| Name | Type | Linked Component |
|:-----|:-----|:-----------------|
| Lab2-ASG-CPU-High | CloudWatch Alarm | Lab2 Auto Scaling Group (AWS/EC2 CPUUtilization > 60%, dimension AutoScalingGroupName=Lab2-ASG) |
| AWS/EC2 CPUUtilization | CloudWatch Metric | Lab2 ASG Instance (per-instance and per-ASG CPU metrics) |
| AWS/AutoScaling GroupDesiredCapacity | CloudWatch Metric | Lab2 Auto Scaling Group (tracks desired, in-service, and total instance counts) |
