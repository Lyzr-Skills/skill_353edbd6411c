# ALB Web Traffic Flow

End-to-end HTTP request flow from the internet through the Application Load Balancer to auto-scaled EC2 instances in Lab2. Inbound traffic arrives at the ALB on port 80, passes through the ALB security group, is forwarded by the listener to the target group, and reaches a healthy EC2 instance managed by the Auto Scaling Group. The EC2 security group only permits inbound traffic originating from the ALB security group, enforcing a layered security model.

## Flow

```graph
<components>
<id icon="globe">{C1} - Internet Client</id>
<id icon="shield">{C2} - Lab2 ALB Security Group</id>
<id icon="server">{C3} - Lab2 ALB</id>
<id icon="list">{C4} - Lab2 ALB Listener</id>
<id icon="target">{C5} - Lab2 Target Group</id>
<id icon="shield">{C6} - Lab2 EC2 Security Group</id>
<id icon="server">{C7} - Lab2 ASG Instance</id>
</components>
<connections>
<con>{C1} -> {C2} (description="HTTP request enters ALB security group on port 80", sequence="1")</con>
<con>{C2} -> {C3} (description="Traffic allowed through to ALB", sequence="2")</con>
<con>{C3} -> {C4} (description="ALB listener receives request", sequence="3")</con>
<con>{C4} -> {C5} (description="Listener forwards to target group via default action", sequence="4")</con>
<con>{C5} -> {C6} (description="Target group routes to registered instance, EC2 SG evaluates", sequence="5")</con>
<con>{C6} -> {C7} (description="Traffic allowed from ALB SG, request reaches EC2 instance", sequence="6")</con>
</connections>
```

### Components

| Component | Responsibility |
|:----------|:---------------|
| Internet Client | Originates HTTP requests to the ALB public DNS endpoint |
| Lab2 ALB Security Group | Filters inbound traffic, allows HTTP port 80 from internet |
| Lab2 ALB | Distributes incoming HTTP traffic across availability zones |
| Lab2 ALB Listener | Evaluates requests on port 80 and applies forwarding rules |
| Lab2 Target Group | Health-checks instances and routes to healthy targets |
| Lab2 EC2 Security Group | Permits inbound traffic only from the ALB security group |
| Lab2 ASG Instance | Processes HTTP request and returns response |

## Observability

### Metrics
| Metric | Source | Dimensions |
|:-------|:-------|:-----------|
| RequestCount | AWS/ApplicationELB | LoadBalancer=app/Lab2-ALB/317aff2800384fc8 |
| TargetResponseTime | AWS/ApplicationELB | LoadBalancer=app/Lab2-ALB/317aff2800384fc8 |
| HealthyHostCount | AWS/ApplicationELB | TargetGroup=targetgroup/Lab2-TG/f9c358a5d351c97f |
| UnHealthyHostCount | AWS/ApplicationELB | TargetGroup=targetgroup/Lab2-TG/f9c358a5d351c97f |
| CPUUtilization | AWS/EC2 | AutoScalingGroupName=Lab2-ASG |
| GroupInServiceInstances | AWS/AutoScaling | AutoScalingGroupName=Lab2-ASG |
