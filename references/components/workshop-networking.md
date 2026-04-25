# Workshop Networking

The shared VPC infrastructure (DevOpsAgent-Workshop-VPC, 10.10.0.0/16) provides network connectivity for all three lab workloads. Public subnets route through an internet gateway for inbound and outbound internet traffic. Private subnets route through a NAT gateway for outbound-only internet access. Each lab uses dedicated subnets: Lab2 has two public subnets across AZs, Lab3 has one public subnet, and Lab4 has two public and two private subnets. A default VPC (172.31.0.0/16) also exists but is unused by lab workloads.

## Component Diagram

```graph
<components>
<id icon="network">{C1} - AWS::EC2::VPC Workshop VPC</id>
<id icon="layout-grid">{C2} - AWS::EC2::Subnet Lab2 Public Subnets</id>
<id icon="layout-grid">{C3} - AWS::EC2::Subnet Lab3 Public Subnet</id>
<id icon="layout-grid">{C4} - AWS::EC2::Subnet Lab4 Public Subnets</id>
<id icon="layout-grid">{C5} - AWS::EC2::Subnet Lab4 Private Subnets</id>
<id icon="globe">{C6} - AWS::EC2::InternetGateway Internet Gateway</id>
<id icon="arrow-up-right">{C7} - AWS::EC2::NatGateway NAT Gateway</id>
<id icon="git-branch">{C8} - AWS::EC2::RouteTable Public Route Table</id>
<id icon="git-branch">{C9} - AWS::EC2::RouteTable Private Route Table</id>
</components>
<connections>
<con>{C2} -> {C1} (mechanism="VPC", description="Lab2 public subnets in Workshop VPC")</con>
<con>{C3} -> {C1} (mechanism="VPC", description="Lab3 public subnet in Workshop VPC")</con>
<con>{C4} -> {C1} (mechanism="VPC", description="Lab4 public subnets in Workshop VPC")</con>
<con>{C5} -> {C1} (mechanism="VPC", description="Lab4 private subnets in Workshop VPC")</con>
<con>{C6} -> {C1} (mechanism="Attachment", description="IGW attached to Workshop VPC")</con>
<con>{C8} -> {C6} (mechanism="Route", description="0.0.0.0/0 routes to IGW")</con>
<con>{C9} -> {C7} (mechanism="Route", description="0.0.0.0/0 routes to NAT")</con>
<con>{C7} -> {C4} (mechanism="Placement", description="NAT in Lab4 public subnet")</con>
<con>{C2} -> {C8} (mechanism="Association", description="Lab2 subnets use public route table")</con>
<con>{C3} -> {C8} (mechanism="Association", description="Lab3 subnet uses public route table")</con>
<con>{C4} -> {C8} (mechanism="Association", description="Lab4 public subnets use public route table")</con>
<con>{C5} -> {C9} (mechanism="Association", description="Lab4 private subnets use private route table")</con>
</connections>
```

## Components

| Component | Description | Type | Sample Resources |
|:----------|:------------|:-----|:-----------------|
| Workshop VPC | Primary VPC (10.10.0.0/16) hosting all lab workloads | AWS::EC2::VPC | vpc-022249fb5a023ef57 |
| Lab2-Public-Subnet-1 | Public subnet in us-east-1a (10.10.1.0/24) for Lab2 ALB and ASG | AWS::EC2::Subnet | subnet-0996fa89d509712cd |
| Lab2-Public-Subnet-2 | Public subnet in us-east-1b (10.10.2.0/24) for Lab2 ALB and ASG | AWS::EC2::Subnet | subnet-0cb6b1ae502ee6055 |
| Lab3-Public-Subnet | Public subnet in us-east-1a (10.10.10.0/24) for Lab3 web server | AWS::EC2::Subnet | subnet-01531feea1da895f5 |
| Lab4-Public-Subnet-1 | Public subnet in us-east-1a (10.10.20.0/24) for bastion and NAT | AWS::EC2::Subnet | subnet-0ae5e0f2e8df9061f |
| Lab4-Public-Subnet-2 | Public subnet in us-east-1b (10.10.21.0/24) | AWS::EC2::Subnet | subnet-0932697eaada6849c |
| Lab4-Private-Subnet-1 | Private subnet in us-east-1a (10.10.30.0/24) for RDS | AWS::EC2::Subnet | subnet-03806a73337efa189 |
| Lab4-Private-Subnet-2 | Private subnet in us-east-1b (10.10.31.0/24) for RDS | AWS::EC2::Subnet | subnet-04b5413876d5b69de |
| Internet Gateway | Provides internet connectivity for public subnets | AWS::EC2::InternetGateway | igw-06f656461ea21598b |
| NAT Gateway | Provides outbound internet for private subnets, placed in Lab4-Public-Subnet-1 | AWS::EC2::NatGateway | nat-083aa3972390c7c7b |
| Public Route Table | Routes 0.0.0.0/0 to IGW, associated with all public subnets | AWS::EC2::RouteTable | rtb-0082d86a4bec404d3 |
| Private Route Table | Routes 0.0.0.0/0 to NAT, associated with private subnets | AWS::EC2::RouteTable | rtb-0960b783e80bbaae8 |
| VPC Flow Log | Captures all VPC network traffic for security analysis | AWS::EC2::FlowLog | fl-0e68b062938dc0828 |
| Network ACL | Default NACL for Workshop VPC | AWS::EC2::NetworkAcl | acl-08d04debc2304f78e |
| Default VPC | Unused default VPC (172.31.0.0/16) with 6 default subnets | AWS::EC2::VPC | vpc-07d815b6c6b485add |

## Observability

| Name | Type | Linked Component |
|:-----|:-----|:-----------------|
| /aws/vpc/flowlogs/Lab3-networksec | CloudWatch Log Group | VPC Flow Log (all network traffic for vpc-022249fb5a023ef57) |
