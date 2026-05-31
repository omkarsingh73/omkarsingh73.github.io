- [[#Top AWS Interview Questions (100)|Top AWS Interview Questions (100)]]
- [[#Scenario-Based Questions|Scenario-Based Questions]]
- [[#Service Deep Dives with Interview Focus|Service Deep Dives with Interview Focus]]
- [[#Interview Tips|Interview Tips]]
- [[#Common Mistakes|Common Mistakes]]
- [[#Key Takeaways|Key Takeaways]]

# AWS Interview Questions - Quick Revision Guide

## Top AWS Interview Questions (100)

### Beginner (1-20)
| # | Question | Concise Answer |
|---|----------|----------------|
| 1 | What is AWS? | Cloud platform offering IaaS, PaaS, SaaS services. Pay-as-you-go pricing. |
| 2 | Diff IaaS vs PaaS vs SaaS | IaaS (EC2): Infrastructure control. PaaS (RDS): Managed apps. SaaS (WorkMail): End-user apps. |
| 3 | What is an EC2 instance? | Virtual server with OS, storage, networking. Full control but requires management. |
| 4 | What is S3? | Object storage with unlimited scalability. Durable, highly available. |
| 5 | What is VPC? | Isolated virtual network for AWS resources. Custom CIDR, subnets, routing. |
| 6 | What is IAM? | Identity and Access Management. Users/roles/policies for least privilege. |
| 7 | What is a Lambda function? | Serverless compute for event-driven workloads. Stateless, <15 min duration. |
| 8 | What is RDS? | Managed relational database service (MySQL, PostgreSQL, etc.). |
| 9 | What is Auto Scaling? | Dynamically adjusts EC2 capacity based on metrics (CPU, requests). |
| 10 | What is CloudWatch? | Monitoring service for metrics, logs, alarms, dashboards. |
| 11 | What is Route 53? | DNS service with health checks, latency-based routing. |
| 12 | What is an AMI? | Template for EC2 instances. Contains OS, apps, configurations. |
| 13 | What is a Security Group? | Virtual firewall for EC2 instances. Stateful, allow/deny traffic at instance level. |
| 14 | What is SNS? | Pub/Sub messaging service. Fan-out notifications to endpoints. |
| 15 | What is SQS? | Fully managed message queue. Decouples producers and consumers. |
| 16 | What is CloudFront? | CDN for caching content globally. Reduces latency, offloads origin. |
| 17 | What is DynamoDB? | Fully managed NoSQL DB. Low-latency, automatic scaling. |
| 18 | What is Elastic Load Balancer (ELB)? | Distributes traffic across EC2 instances. Health checks, SSL termination. |
| 19 | What is EBS? | Persistent block storage for EC2. IO-bound performance varies by type. |
| 20 | What is Glacier? | Low-cost archival storage. Retrieval times from minutes to hours. |

### Intermediate (21-60)
| # | Question | Concise Answer |
|---|----------|----------------|
| 21 | Diff Spot vs On-Demand Instances | Spot: Cheaper, interrupted. On-Demand: Full price, no interruption. |
| 22 | When to use Lambda vs EC2? | Lambda: Stateless, event-driven. EC2: Long-running, OS control needed. |
| 23 | What is Multi-AZ RDS? | Synchronous replication across AZs. Failover < 30 sec for HA. |
| 24 | How do Security Groups diff from NACLs? | SG: Stateful, instance-level. NACL: Stateless, subnet-level. |
| 25 | What is EventBridge? | Event bus for routing events between AWS services/custom apps. |
| 26 | What is Fargate? | Serverless compute for containers. No node management. |
| 27 | What is CloudFormation? | IaC service. Define infrastructure as code (JSON/YAML). |
| 28 | What is S3 Lifecycle Policy? | Automate transition/delete of objects (e.g., move to Glacier after 90 days). |
| 29 | What is DAX? | In-memory cache for DynamoDB. Microsecond latency for read-heavy workloads. |
| 30 | What is API Gateway? | Managed API gateway for REST/WebSocket APIs. Throttling, caching, auth. |
| 31 | How does ALB diff from NLB? | ALB: HTTP/HTTPS, path-based routing. NLB: TCP/UDP, static IP, extreme performance. |
| 32 | What is CloudTrail? | Auditing service. Logs all API calls + management events. |
| 33 | What is KMS? | Key Management Service. Encrypt/decrypt data, manage CMKs. |
| 34 | What is Secrets Manager? | Secure storage/retrieval of secrets (DB creds, API keys). Auto-rotation. |
| 35 | What is ECS? | Container orchestration for Docker. AWS-managed scheduling. |
| 36 | What is EKS? | Managed Kubernetes service. Portable, extensible orchestration. |
| 37 | What is VPC Peering? | Connect two VPCs privately. Non-transitive, no overlapping CIDRs. |
| 38 | What is Transit Gateway? | Hub for connecting VPCs/VPNs. Scalable, central management. |
| 39 | What is S3 Event Notification? | Trigger Lambda/SQS/SNS on object creation (e.g., image resize). |
| 40 | What is RDS Read Replica? | Async copy of primary DB. Offload reads, cross-AZ/region. |
| 41 | What is CloudFront Signed URLs? | Restrict access to content via signed URLs/cookies. |
| 42 | What is Lambda Provisioned Concurrency? | Pre-initialized functions to eliminate cold starts. |
| 43 | What is Auto Scaling Cooldown? | Time window before scaling actions can repeat. Prevents flapping. |
| 44 | What is IAM Role vs Policy? | Role: Identity with temporary credentials. Policy: Permissions attached to entities. |
| 45 | What is DynamoDB TTL? | Auto-delete items after a set time. Useful for logs/sessions. |
| 46 | What is CloudWatch Alarms? | Trigger actions (SNS, Lambda) when metrics breach thresholds. |
| 47 | What is S3 Versioning? | Preserve older object versions. Protect against accidental deletion. |
| 48 | What is Elasticache? | Managed in-memory cache (Redis/Memcached). |
| 49 | What is AWS WAF? | Web Application Firewall. Protect APIs/websites from common exploits. |
| 50 | What is CodePipeline? | CI/CD service. Automate build/test/deploy workflows. |
| 51 | What is S3 Transfer Acceleration? | Use CloudFront edge locations to speed up S3 transfers. |
| 52 | What is RDS Global Database? | Low-latency global reads. Multi-region replication. |
| 53 | What is Lambda Layers? | Share code/libraries across functions. |
| 54 | What is Spot Fleet? | Combine multiple Spot Instance pools for capacity optimization. |
| 55 | What is API Gateway Caching? | Cache API responses to reduce latency/backend load. |
| 56 | What is CloudSearch? | Managed search service. Full-text search for apps. |
| 57 | What is AWS Outposts? | Extend AWS to on-premises environments. |
| 58 | What is Kinesis Data Streams? | Real-time data streaming pipeline. |
| 59 | What is EFS? | Shared file system for EC2. Scales automatically. |
| 60 | What is AWS Config? | Auditing service. Track resource configurations and changes. |

### Advanced (61-100)
| # | Question | Concise Answer |
|---|----------|----------------|
| 61 | How to handle Lambda cold starts? | Use Provisioned Concurrency, keep functions warm, optimize package size. |
| 62 | Design a HA web app architecture. | Multi-AZ ALB + EC2 ASG + Multi-AZ RDS + S3 + CloudFront. |
| 63 | When to use SNS vs SQS? | SNS: Fan-out to many subscribers. SQS: Queue for single consumer. |
| 64 | How to troubleshoot high latency in DynamoDB? | Check RCU/WCU, partition keys, GSI design, DAX. |
| 65 | How to secure S3 bucket? | Block public access, Bucket Policies, encryption (SSE-S3/KMS), logging. |
| 66 | Diff between ALB and NLB? | ALB: HTTP(L7), path-based routing. NLB: TCP/UDP(L4), static IP, higher throughput. |
| 67 | How to implement blue-green deployment? | Use ALB target groups + Route 53 weighted routing or CodeDeploy. |
| 68 | What is eventual consistency in DynamoDB? | Writes confirmed before reads. May need to handle stale data. |
| 69 | How to handle DB schema migrations in RDS? | Use AWS DMS or native tools with zero-downtime strategies. |
| 70 | What is AWS Backup? | Centralized backup service for EBS, RDS, DynamoDB, etc. |
| 71 | How to optimize Lambda cost? | Right-size memory, set timeout, use Power Tuning, DLQ for retries. |
| 72 | What is AWS Outposts use case? | Hybrid cloud scenarios where AWS services needed on-prem. |
| 73 | How to implement circuit breaker pattern? | Use SQS as buffer + Lambda to check downstream health. |
| 74 | What is AWS Fault Injection Simulator? | Inject failures (AZ outages, latency) to test resiliency. |
| 75 | How to handle cross-region replication? | S3 CRR, DynamoDB Global Tables, RDS read replicas. |
| 76 | What is AWS Service Catalog? | Offer approved products (AMIs, templates) to users. |
| 77 | How to debug ECS task failures? | Check CloudWatch Logs, task definition, IAM roles, security groups. |
| 78 | What is AWS App Mesh? | Service mesh for microservices communication (traffic control, observability). |
| 79 | How to implement rate limiting? | API Gateway throttling, DynamoDB sharding, custom Lambda. |
| 80 | What is AWS Nitro Enclaves? | Secure execution environment for sensitive workloads (e.g., DB encryption). |
| 81 | How to handle large file uploads to S3? | Use S3 Multipart Upload (+10 parts for >100 MB). |
| 82 | What is AWS Global Accelerator? | Route traffic to nearest endpoint using Anycast IPs. Improves latency. |
| 83 | How to implement auto-scaling for RDS? | Aurora Serverless v2 (pay-per-second, scales in < seconds). |
| 84 | What is AWS Systems Manager (SSM)? | Manage and automate operations (patching, config, remote execution). |
| 85 | How to troubleshoot VPC flow logs? | Analyze logs in CloudWatch for denied traffic/reachability issues. |
| 86 | What is AWS Lambda Destinations? | Route function output to SQS/SNS/EventBridge after execution. |
| 87 | How to implement OAuth2 with Cognito? | User pools for auth, identity pools for federated access. |
| 88 | What is AWS Resilience Hub? | Validate disaster recovery targets (RTO/RPO) for apps. |
| 89 | How to encrypt data at rest in S3? | Enable SSE-S3, SSE-KMS, or client-side encryption. |
| 90 | What is AWS Well-Architected Tool? | Review architecture against best practices (security, cost, etc.). |
| 91 | How to handle schema evolution in Kinesis? | Use schema registry or embed version in records. |
| 92 | What is AWS DataSync? | Agentless data transfer between on-prem and AWS storage. |
| 93 | How to implement custom metrics in CloudWatch? | Use CloudWatch Agent or putMetricData API. |
| 94 | What is AWS Transit Gateway Attachment? | Connect VPCs/VPNs to a central Transit Gateway. |
| 95 | How to troubleshoot Lambda DNS issues in VPC? | Use VPC resolver, check security groups, enable DNS hostnames. |
| 96 | What is AWS Control Tower? | Landing zone setup for multi-account AWS environments. |
| 97 | How to implement audit logging for S3? | Enable AWS CloudTrail + S3 data events. |
| 98 | What is AWS Macie? | Automate sensitive data discovery/classification in S3. |
| 99 | How to optimize EKS costs? | Use Spot Instances, Fargate profiles for stateless pods, right-size nodes. |
| 100 | What is AWS Graviton? | ARM-based processors for EC2 (e.g., T4g, M6g) – up to 40% better price/performance. |

---

## Scenario-Based Questions

### 1. Design a global e-commerce platform
- **Requirements**: Low-latency product catalog, order processing, inventory sync across regions.
- **Solution**:
  - **Frontend**: S3 + CloudFront (edge caching)
  - **API**: API Gateway + Lambda (AWS regions) + DynamoDB (global tables)
  - **Orders**: ECS/Fargate in multiple regions + RDS Multi-AZ + cross-region read replicas
  - **Inventory**: EventBridge fan-out to SNS → SQS → Lambda for stock updates
  - **Search**: OpenSearch (distributed across AZs)

### 2. Debug “Lambda function timed out”
- **Checklist**:
  1. Increase timeout settings (max 15 min)
  2. Check CloudWatch Logs for errors
  3. Optimize code (reduce dependencies, use layers)
  4. Verify VPC config (if used) – security groups, NLB/NAT
  5. Enable X-Ray for tracing

### 3. High RDS CPU utilization
- **Root Causes**:
  - Long-running queries (check slow query logs)
  - Missing indexes
  - Insufficient instance size
  - Misconfigured parameter groups
- **Fix**: 
  - Add read replicas, optimize queries, scale instance, use Aurora Serverless

---

## Service Deep Dives with Interview Focus

### EC2
- **What it is**: Virtual servers with full OS control.
- **Core Use Cases**: Legacy apps, custom kernels, burst capacity.
- **When to Use**: Need OS-level access, stateful apps.
- **When NOT to Use**: Stateless APIs (use Lambda/Fargate).
- **Key Concepts**: AMI, Instance Types (C/X/M/R), EBS Volumes, Key Pairs.
- **Scaling**: Auto Scaling Groups, Spot Fleet, EC2 Fleet.
- **Security**: IAM Roles, Security Groups, EC2 Instance Connect.
- **Cost Opto**: Spot Instances, Reserved Instances, Savings Plans.
- **Interview Qs**:
  - Diff On-Demand vs Spot vs Reserved?
  - How to handle instance termination notices?
  - How to secure SSH access?
- **Common Issues**: EBS throttling, network latency, instance metadata vulnerabilities.
- **Best Practices**: Tag resources, use ASG health checks, patch via SSM.
- **Real-World Ex**: Java microservice hosting with ALB + Multi-AZ ASG.

### Lambda
- **What it is**: Serverless event-driven compute.
- **Core Use Cases**: API backends, data pipelines, async processing.
- **When to Use**: Stateless, short-lived (<15 min), event-driven.
- **When NOT to Use**: Long-running, stateful, heavy memory needs.
- **Key Concepts**: Layers, Cold Starts, Provisioned Concurrency, DLQ.
- **Scaling**: Concurrent executions (soft limit 1000+), auto-scaled.
- **Security**: Execution Role, VPC, Environment Encryption.
- **Cost Opto**: Free Tier (1M reqs/month), optimize memory/duration.
- **Interview Qs**:
  - How to avoid cold starts?
  - What’s the max package size?
  - How to handle state?
- **Common Issues**: Cold starts, throttling, large deployment packages.
- **Best Practices**: Use layers, X-Ray tracing, DLQ for errors.
- **Real-World Ex**: Image resize on S3 upload → Lambda → S3.

### RDS
- **What it is**: Managed relational DB (MySQL/PostgreSQL/Oracle/SQL Server).
- **Core Use Cases**: OLTP, reporting, legacy apps.
- **When to Use**: ACID compliance, complex queries, JOINs needed.
- **When NOT to Use**: High-write throughput (use DynamoDB), global distribution.
- **Key Concepts**: Multi-AZ, Read Replicas, Parameter Groups, Subnet Groups.
- **Scaling**: Vertical (instance upgrade), Read Replicas, Aurora Serverless.
- **Security**: IAM DB Auth, SSL, VPC, KMS encryption.
- **Cost Opto**: Reserved Instances, Aurora Serverless (pay-per-second).
- **Interview Qs**:
  - Multi-AZ vs Read Replica?
  - Point-in-time recovery?
  - Automated backups?
- **Common Issues**: Failover delays, parameter misconfigs, storage full.
- **Best Practices**: Enable backups, monitoring, parameter groups.
- **Real-World Ex**: E-commerce order processing.

### DynamoDB
- **What it is**: Fully managed NoSQL DB.
- **Core Use Cases**: Session stores, catalogs, IoT telemetry.
- **When to Use**: High write throughput, low-latency reads, flexible schema.
- **When NOT to Use**: Complex joins, ad-hoc queries.
- **Key Concepts**: Items, PK/SK, GSI/LSI, RCU/WCU, On-Demand/Provisioned.
- **Scaling**: Auto-scaling (provisioned), unlimited (On-Demand).
- **Security**: IAM policies, encryption, PITR.
- **Cost Opto**: Use TTL to archive, On-Demand for spiky traffic.
- **Interview Qs**:
  - How to model a user-order relationship?
  - Diff between GSI and LSI?
  - How to handle throttling?
- **Common Issues**: Hot partitions, throttling, TTL delays.
- **Best Practices**: Sparse indexes, batch writes, monitor metrics.
- **Real-World Ex**: Product catalog for e-commerce.

---

## Interview Tips
- **Compare services**: Focus on trade-offs (cost vs control, latency vs consistency).
- **Discuss scaling**: Mention both vertical/horizontal strategies.
- **Security first**: Always mention IAM roles, encryption defaults.
- **Real-world examples**: Use past projects to demonstrate production experience.
- **Quantify impact**: “Reduced Lambda duration by 40% by optimizing memory.”

## Common Mistakes
- Assuming S3 is instantly consistent (it’s eventually consistent).
- Using EC2 for stateless APIs without load balancing/auto-scaling.
- Not enabling encryption by default (RDS, S3).
- Over-provisioning on-demand DynamoDB without monitoring.

## Key Takeaways
1. **Right-tool-for-job**: Stateless → Lambda; Relational → RDS; High-write → DynamoDB.
2. **Security is default**: Enable encryption, IAM roles, VPC isolation.
3. **Cost scales with patterns**: Use Spot/Fargate/Serverless for variable loads.
4. **Automate everything**: IaC, CI/CD, scaling policies.
5. **Design for failure**: Multi-AZ/region, retries, DLQs.
