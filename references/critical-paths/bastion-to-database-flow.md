# Bastion to Database Flow

Secure database access flow from an operator through the bastion host to the RDS MySQL instance in Lab4. An operator initiates an SSM Session Manager session to the bastion host via VPC interface endpoints (SSM, SSM Messages, EC2 Messages), avoiding the need for SSH keys or open inbound ports. From the bastion host, the operator connects to the RDS MySQL instance over port 3306 through the private network. The RDS security group only allows MySQL traffic from the bastion security group.

## Flow

```graph
<components>
<id icon="user">{C1} - Operator</id>
<id icon="plug">{C2} - SSM VPC Endpoints</id>
<id icon="server">{C3} - Lab4 Bastion Host</id>
<id icon="shield">{C4} - Lab4 Bastion Security Group</id>
<id icon="shield">{C5} - Lab4 RDS Security Group</id>
<id icon="database">{C6} - ecommerce-db RDS Instance</id>
</components>
<connections>
<con>{C1} -> {C2} (description="Operator starts SSM Session Manager session via VPC endpoints", sequence="1")</con>
<con>{C2} -> {C3} (description="Session Manager connects to bastion host without SSH", sequence="2")</con>
<con>{C3} -> {C4} (description="Bastion initiates MySQL connection, bastion SG allows outbound", sequence="3")</con>
<con>{C4} -> {C5} (description="RDS SG allows inbound MySQL 3306 from bastion SG", sequence="4")</con>
<con>{C5} -> {C6} (description="MySQL connection established to ecommerce-db", sequence="5")</con>
</connections>
```

### Components

| Component | Responsibility |
|:----------|:---------------|
| Operator | Initiates SSM Session Manager session from AWS Console or CLI |
| SSM VPC Endpoints | Provide private connectivity for Session Manager (SSM, SSM Messages, EC2 Messages) |
| Lab4 Bastion Host | Intermediary EC2 instance (i-027c8f54b29aa0683) for secure database access |
| Lab4 Bastion Security Group | Controls bastion network access, shared with VPC endpoints |
| Lab4 RDS Security Group | Restricts MySQL access to bastion security group only |
| ecommerce-db RDS Instance | MySQL 8.0.45 database (db.t3.medium) hosting the ecommerce database |

## Observability

### Metrics
| Metric | Source | Dimensions |
|:-------|:-------|:-----------|
| DatabaseConnections | AWS/RDS | DBInstanceIdentifier=ecommerce-db |
| CPUUtilization | AWS/RDS | DBInstanceIdentifier=ecommerce-db |
| FreeableMemory | AWS/RDS | DBInstanceIdentifier=ecommerce-db |
| FreeStorageSpace | AWS/RDS | DBInstanceIdentifier=ecommerce-db |
| ReadIOPS | AWS/RDS | DBInstanceIdentifier=ecommerce-db |
| WriteIOPS | AWS/RDS | DBInstanceIdentifier=ecommerce-db |
| ReadLatency | AWS/RDS | DBInstanceIdentifier=ecommerce-db |
| WriteLatency | AWS/RDS | DBInstanceIdentifier=ecommerce-db |
