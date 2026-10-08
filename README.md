# AWS Infrastructure Monthly Cost Estimate

Below is the detailed A-to-Z breakdown of our expected monthly AWS infrastructure costs for both the Development and Production environments. This estimate is based on our current infrastructure-as-code (Terraform) setup in the `ap-south-2` (Hyderabad) region and the architecture defined in our High-Level Design (HLD).

> [!NOTE]
> Prices are estimated based on standard AWS pricing for the Asia Pacific (Hyderabad) region. Usage-based resources (such as NAT data processing, ALB capacity units, and exact SMS counts) are modeled on our expected baseline traffic.

## Cost Breakdown Table

| Resource Category | Dev Environment | Prod Environment | Cost Drivers & Assumptions |
| :--- | :--- | :--- | :--- |
| **VPC & NAT Gateway** | ~$35.00 | ~$105.00 | **Dev**: 1 NAT Gateway.<br>**Prod**: 3 NAT Gateways (1 per Availability Zone for High Availability). |
| **ALB (Load Balancer)** | ~$22.00 | ~$22.00 | Fixed hourly rate ($16.42) + baseline capacity usage. |
| **ECS Fargate (Compute)** | ~$36.00 | ~$216.00 | **Dev**: 2 Tasks (0.5 vCPU, 1 GB RAM). <br>**Prod**: 3 Tasks (2 vCPU, 4 GB RAM). |
| **RDS PostgreSQL** | ~$14.00 | ~$111.00 | **Dev**: `db.t4g.micro` (Single-AZ) + 20 GB Storage.<br>**Prod**: `db.t4g.medium` (Multi-AZ) + 100 GB Storage. |
| **RDS Proxy** | ~$22.00 | ~$22.00 | Priced per vCPU of the underlying database. Both instances have 2 vCPUs. |
| **WAF (Web Firewall)** | ~$6.00 | ~$6.00 | 1 WebACL + 1 Managed Rule Group + estimated request traffic. |
| **Secrets Manager** | ~$0.40 | ~$0.40 | Secure credential storage. |
| **ElastiCache (Redis)** | ~$12.00 | ~$96.00 | **Dev**: `cache.t4g.micro` (Single Node).<br>**Prod**: `cache.t4g.medium` (Multi-AZ / 2 Nodes). |
| **Serverless Pipeline**<br>*(SQS, Lambda, EventBridge)* | ~$2.00 | ~$10.00 | **Highly cost-effective**. First 1 million Lambda/SQS requests are free. Primary costs come from S3 document storage and ECR image storage. |
| **SNS (SMS) & SES** | ~$1.50 | ~$28.00 | **Dev**: ~500 SMS/mo.<br>**Prod**: 5,000 to 10,000 SMS/mo (at ~$0.0028 per message). SES emails remain largely within the free tier. |
| --- | --- | --- | --- |
| **Total Estimated Cost** | **~$150.90 / mo** | **~$616.40 / mo** | |

## 📊 Summary & Budget Ranges

To account for traffic spikes, unpredictable data transfer costs, and serverless usage slightly exceeding the free tiers as we scale, we should budget within the following safe ranges:

### 🛠️ Development Environment
**Expected Range: $150 — $180 per month**
The Dev environment is highly cost-optimized. We keep baseline costs low by using smaller compute instances (`t4g.micro`), running single-AZ databases, and provisioning only a single NAT Gateway.

### 🚀 Production Environment
**Expected Range: $600 — $650 per month**
The Production environment scales up significantly to guarantee high availability and fault tolerance:
- **Redundancy:** We deploy 3 NAT Gateways (one in each Availability Zone) to ensure the network never goes down.
- **Compute:** ECS costs scale up as we run 3 larger containers (2 vCPU / 4 GB each) to handle live user traffic.
- **Database:** RDS costs reflect the use of `db.t4g.medium` with **Multi-AZ** enabled. This provisions a synchronous standby replica in a secondary zone to prevent data loss or downtime during a database failure.
- **Messaging:** SMS costs are scaled up to accommodate 5,000 - 10,000 monthly messages.
