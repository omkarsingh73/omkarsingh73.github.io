- [[#Compute|Compute]]
- [[#Storage|Storage]]
- [[#Databases|Databases]]
- [[#Networking|Networking]]
- [[#Security|Security]]
- [[#Monitoring & DevOps|Monitoring & DevOps]]
- [[#Interview Tips|Interview Tips]]
- [[#Key Takeaways|Key Takeaways]]

## Compute

### EC2
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Virtual servers with full OS control                                    |
| **Core Use Cases** | Custom apps, legacy lifting, batch jobs, HPC                    |
| **When to Use** | Need OS-level control, custom kernel, legacy dependencies             |
| **When NOT to Use** | Stateless workloads (use Lambda), simple APIs (use FaaS)          |
| **Key Concepts** | AMIs, Instance Types, EBS Volumes, Security Groups, Key Pairs         |
| **Scaling**     | Auto Scaling Groups, EC2 Fleet, Spot Fleet                             |
| **Security**    | IAM Roles, Security Groups, EC2 Instance Connect, SSM Agent           |
| **Cost Opto**   | Spot Instances, Reserved Instances, Instance Families (e.g. `c5n` for networking) |
| **Interview Qs** | *Diff between On-Demand vs Spot? How handle instance termination?* |
| **Prod Issues** | Instance launch failures, EBS throttling, network congestion          |
| **Best Practices** | Use ASG, tag resources, enable CloudWatch Logs, patch via SSM     |
| **Real-World Ex** | Java microservice hosting with ALB + ASG                               |

| **ECS**                     | **EKS**                       |
| --------------------------- | ----------------------------- |
| AWS-managed orchestration   | Kubernetes on AWS             |
| Simpler learning curve      | Portable to other clouds      |
| Good for Docker-native apps | Complex apps, CI/CD pipelines |
| AWS Fargate integration     | Requires worker nodes         |

---

### Lambda
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Event-driven serverless compute                                        |
| **Core Use Cases** | API backends, data transformations, async processing, scheduled tasks |
| **When to Use** | Stateless, short-lived (<15 min), event-driven workloads              |
| **When NOT to Use** | Long-running jobs, stateful apps, heavy memory use (>10 GB)       |
| **Key Concepts** | Functions, Layers, Event Sources (S3, DynamoDB, SQS), Cold Starts    |
| **Scaling**     | Concurrent executions (limit 1000+), auto-scaled by AWS                |
| **Security**    | Execution Role, VPC Configuration, Environment Encryption              |
| **Cost Opto**   | Free Tier (1M reqs/month), duration-based pricing                      |
| **Interview Qs** | *How to avoid cold starts? What is Lambda Provisioned Concurrency?* |
| **Prod Issues** | Cold starts, throttling, large deployment packages                    |
| **Best Practices** | Use Layers for shared code, X-Ray tracing, DLQ for errors          |
| **Real-World Ex** | Image resize on S3 upload, webhook handler for GitHub               |

---

### Auto Scaling
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Dynamic scaling for EC2, ECS, RDS, Lambda                               |
| **Core Use Cases** | Web apps, batch workers, database read replicas                      |
| **Key Concepts** | Launch Templates, Scaling Policies (target, step, simple)             |
| **Scaling**     | CPU-based, request-based, scheduled, predictive                        |
| **Security**    | IAM Roles for scaling actions                                          |
| **Interview Qs** | *Diff between target tracking vs step scaling?*                    |
| **Prod Issues** | Flapping scales, cooldown periods causing delays                       |
| **Best Practices** | Use multi-AZ, health checks, CloudWatch Alarms integration          |

---

## Storage

### S3
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Object storage with unlimited scalability                               |
| **Core Use Cases** | Static websites, backups, data lake, media storage                    |
| **Key Concepts** | Buckets, Objects, Versioning, Lifecycle Policies, ACLs/Bucket Policies |
| **Security**    | Encryption (SSE-S3/SSE-KMS), Block Public Access, IAM Policies         |
| **Cost Opto**   | Intelligent-Tiering, Glacier for archives, ZSTD compression            |
| **Interview Qs** | *Event notifications? Consistency model?*                         |
| **Prod Issues** | Eventual consistency reads, permission errors, storage class transitions |
| **Best Practices** | Enable Versioning, enable logging, use S3 Access Points            |
| **Real-World Ex** | User profile pictures, static website hosting, log aggregation       |

---

### EFS
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Scalable NFS file system                                                |
| **Core Use Cases** | Shared storage for EC2 clusters, container storage, web servers       |
| **Key Concepts** | Mount Targets, Performance Modes (General/Throughput), Lifecycle Management |
| **Scaling**     | Auto-scaled storage and throughput                                     |
| **Security**    | IAM policies, VPC security groups, encryption at rest                   |
| **Interview Qs** | *How does EFS diff from S3?*                                        |
| **Prod Issues** | Network latency, mount failures, throughput limits                    |
| **Best Practices** | Use IAM authorization, monitor with CloudWatch, use tags          |
| **Real-World Ex** | Shared config files across EC2 fleet, container persistent storage   |

---

## Databases

### RDS
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Managed relational DB (MySQL, PostgreSQL, Oracle, SQL Server, MariaDB) |
| **Core Use Cases** | OLTP, reporting, legacy apps, microservice databases                  |
| **Key Concepts** | Multi-AZ, Read Replicas, Parameter Groups, Subnet Groups               |
| **Scaling**     | Vertical (instance upgrade), Read Replicas, Aurora Serverless          |
| **Security**    | IAM DB Auth, SSL, VPC isolation, KMS encryption                         |
| **Cost Opto**   | Reserved Instances, Aurora Serverless (pay-per-second)                 |
| **Interview Qs** | *When to use Read Replicas vs Multi-AZ?*                             |
| **Prod Issues** | Failover times, parameter misconfigurations, storage full               |
| **Best Practices** | Enable automated backups, monitoring, use parameter groups          |
| **Real-World Ex** | E-commerce order processing, user account management                 |

**RDS vs DynamoDB**
| **RDS**                         | **DynamoDB**                      |
|---------------------------------|-----------------------------------|
| Relational, ACID compliant      | NoSQL, eventual consistency       |
| Complex queries, JOINs          | Simple key-value, GSI/LSI         |
| High operational overhead       | Fully managed, auto-scaling       |
| Use for OLTP, reporting         | Use for high-write workloads      |

---

### DynamoDB
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Fully managed NoSQL DB                                                  |
| **Core Use Cases** | Session stores, catalogs, IoT data, high-throughput apps             |
| **Key Concepts** | Items, Primary Keys, GSI/LSI, RCU/WCU, On-Demand vs Provisioned       |
| **Scaling**     | Auto-scaling (provisioned), unlimited with On-Demand                   |
| **Security**    | IAM policies, encryption at rest, point-in-time recovery               |
| **Cost Opto**   | Use On-Demand for unpredictable traffic, archive to S3 with TTL        |
| **Interview Qs** | *How to model a user-order relationship?*                           |
| **Prod Issues** | Throttling, hot partitions, TTL delays                                |
| **Best Practices** | Use sparse indexes, batch writes, monitor CloudWatch metrics        |
| **Real-World Ex** | Product catalog, leaderboard for gaming, IoT telemetry               |

---

## Networking

### VPC
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Virtual network isolated to your AWS account                            |
| **Key Concepts** | Subnets (public/private), Route Tables, NACLs, Security Groups          |
| **Best Practices** | Use 3-tier architecture, private subnets for DBs, NAT GW for outbound |
| **Interview Qs** | *Public vs Private subnet? NACL vs Security Group?*                 |

### API Gateway
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **What it is**  | Managed API gateway for REST/WebSocket APIs                              |
| **Core Use Cases** | Expose microservices, mobile backend, server-less APIs                |
| **Key Concepts** | Resources/Methods, Deployment, Stages, Usage Plans, WAF integration    |
| **Security**    | IAM Auth, Cognito Auth, API Keys, Resource Policies                     |
| **Interview Qs** | *Caching? Throttling?*                                              |
| **Best Practices** | Use WAF, enable logging, custom domain with ACM                     |

---

## Security

### IAM
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **Key Concepts** | Users/Groups/Roles/Policies, Least Privilege, Policy Inheritance       |
| **Best Practices** | Use Roles over long-term keys, MFA, IAM Access Analyzer             |

### Security Groups vs NACL
| **Security Groups** | **NACLs**                              |
|---------------------|-----------------------------------------|
| Stateful            | Stateless                               |
| Instance-level      | Subnet-level                            |
| Allow-by-default    | Deny-by-default                         |
| Priority-based      | Rule-number based                       |

---

## Monitoring & DevOps

### CloudWatch
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **Key Concepts** | Metrics, Logs, Alarms, Dashboards, Events                            |
| **Best Practices** | Set alarms for error rates, latency, create custom dashboards        |

### CloudFormation
| Aspect          | Details                                                                 |
|-----------------|-------------------------------------------------------------------------|
| **Key Concepts** | Templates (JSON/YAML), Stack, Drift Detection, StackSets                |
| **Best Practices** | Use modular templates, enable termination protection              |

---

## Interview Tips
- **Compare services** (e.g., *ECS vs EKS*, *RDS vs DynamoDB*)
- **Focus on trade-offs**: consistency vs latency, cost vs control
- **Discuss scaling strategies** for each service
- **Mention security defaults** (encryption, IAM roles)
- **Real-world examples** show production experience

## Key Takeaways
1. **Match service to workload pattern** (stateless vs stateful, relational vs NoSQL)
2. **Security is default** - never disable encryption or public access
3. **Cost scales with usage patterns** - optimize based on traffic
4. **Observability first** - enable logging/metrics from day one
5. **Automate everything** - IaC, CI/CD, scaling policies
