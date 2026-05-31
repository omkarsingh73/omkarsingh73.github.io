- [[#Compute|Compute]]
- [[#Storage|Storage]]
- [[#Databases|Databases]]
- [[#Networking|Networking]]
- [[#Security|Security]]
- [[#Messaging/Event|Messaging/Event]]
- [[#Monitoring/DevOps|Monitoring/DevOps]]
- [[#Microservices on AWS|Microservices on AWS]]
- [[#Serverless Architecture|Serverless Architecture]]
- [[#Event-Driven Architecture|Event-Driven Architecture]]
- [[#High Availability|High Availability]]
- [[#CDN Architecture|CDN Architecture]]
- [[#Queue-Based Processing|Queue-Based Processing]]
- [[#Auto Scaling Architecture|Auto Scaling Architecture]]
- [[#Blue-Green Deployment|Blue-Green Deployment]]
- [[#Canary Deployment|Canary Deployment]]
- [[#CI/CD Pipelines|CI/CD Pipelines]]
- [[#Disaster Recovery|Disaster Recovery]]
- [[#Caching Strategies|Caching Strategies]]
- [[#API Scaling|API Scaling]]
- [[#Database Scaling|Database Scaling]]
- [[#Key Services|Key Services]]
- [[#Common Commands (CLI)|Common Commands (CLI)]]
- [[#Important Comparisons|Important Comparisons]]
- [[#Frequently Forgotten Concepts|Frequently Forgotten Concepts]]
- [[#Service Limits|Service Limits]]
- [[#Security Reminders|Security Reminders]]
- [[#Architecture Tips|Architecture Tips]]
- [[#Cost Optimization Reminders|Cost Optimization Reminders]]
- [[#Beginner|Beginner]]
- [[#Intermediate|Intermediate]]
- [[#Advanced|Advanced]]
- [[#Scenario-Based|Scenario-Based]]


## Compute

* **EC2:** Virtual servers.  Use for full control, OS customization, long-running processes. *Don't use* for simple, event-driven tasks. **Scaling:** Auto Scaling Groups (ASG). **Security:** Security Groups. **Cost:** Reserved Instances.
    * **Interview:** "When would you choose EC2 over Lambda?"
    * **Production:** Web servers, application servers, batch processing.
* **Lambda:** Serverless compute. Use for event-driven tasks. *Don't use* for long-running, stateful applications. **Scaling:** Automatic, built-in. **Security:** IAM Roles. **Cost:** Pay-per-use.
    * **Interview:** "Explain the cold start problem with Lambda."
    * **Production:** API endpoints, image processing, scheduled tasks.
* **ECS/EKS:** Container orchestration. ECS (AWS managed), EKS (Kubernetes). Use for complex container deployments. **Scaling:** ASG, scaling policies. **Security:** IAM, Network Policies.
* **Auto Scaling:** Automatically adjust EC2 capacity based on demand.

## Storage

* **S3:** Object storage. Use for static assets, backups, data lakes.  *Don't use* as a primary database. **Scaling:** Virtually unlimited. **Security:** IAM, Bucket Policies. **Cost:** Tiered storage options (Standard, Glacier, etc.).
    * **Interview:** "Describe S3 storage tiers and when you'd use each."
    * **Production:** Storing user uploads, website assets, archival data.
* **EFS:** Network file system. Use for shared storage across EC2 instances. *Don't use* for high-performance, low-latency access.

## Databases

* **RDS:** Managed relational databases (MySQL, PostgreSQL, etc.). Use for traditional relational databases. **Scaling:** Read Replicas, Vertical Scaling. **Security:** VPC, IAM.
* **DynamoDB:** NoSQL database. Use for high-throughput, low-latency access patterns. **Scaling:** Automatic. **Security:** IAM. **Cost:** Provisioned vs. On-Demand Capacity.
* **ElastiCache:** In-memory caching (Redis, Memcached). Use to improve application performance.

## Networking

* **VPC:** Isolated network environment.
* **Route53:** DNS service.
* **CloudFront:** CDN.
* **API Gateway:** Manages APIs.
* **Load Balancer:** Distributes traffic.

## Security

* **IAM:** Identity and Access Management.
* **KMS:** Key Management Service.
* **Secrets Manager:** Securely stores secrets.
* **Security Groups vs NACL:** Security Groups (stateful, instance-level), NACL (stateless, subnet-level).

## Messaging/Event

* **SQS:** Message queue.
* **SNS:** Pub/Sub messaging.
* **EventBridge:** Serverless event bus.
* **Kinesis:** Real-time data streaming.

## Monitoring/DevOps

* **CloudWatch:** Monitoring and logging.
* **CloudFormation:** Infrastructure as Code.
* **X-Ray:** Distributed tracing.
* **CodePipeline:** CI/CD pipeline.

**Key Takeaways:** Choosing the right service is crucial for cost, performance, and scalability.  Understand the tradeoffs.

---
## Microservices on AWS

![Microservices](https://mermaid.live/mermaid/clipart/microservice)

* **Components:** API Gateway, ECS/EKS, DynamoDB, SQS.
* **Tradeoffs:** Increased complexity, operational overhead.
* **Failure Handling:** Circuit breakers, retries, bulkheads.

## Serverless Architecture

* **Components:** Lambda, API Gateway, S3, DynamoDB, EventBridge.
* **Benefits:** Cost-effective, scalable, simplified operations.
* **Considerations:** Cold starts, debugging.

## Event-Driven Architecture

![Event Driven](https://mermaid.live/mermaid/clipart/eventdriven)

* **Components:** EventBridge, SQS, Lambda, SNS.
* **Use Cases:** Decoupled services, asynchronous processing.

## High Availability

* **Multi-AZ vs Multi-Region:** AZs within a region (fast failover), Regions (disaster recovery).

## CDN Architecture

![CDN](https://mermaid.live/mermaid/clipart/cdn)

* **Components:** CloudFront, S3, Origin Server.
* **Benefits:** Reduced latency, improved user experience.

## Queue-Based Processing

* Use SQS to decouple producers and consumers. Excellent for asynchronous tasks.

## Auto Scaling Architecture

![Auto Scaling](https://mermaid.live/mermaid/clipart/autoscaling)

* **Components:** ASG, Load Balancer, EC2.

## Blue-Green Deployment

* Minimize downtime during deployments by having two identical environments.

## Canary Deployment

* Gradually roll out new versions to a small subset of users.

## CI/CD Pipelines

* **Components:** CodePipeline, CodeBuild, CodeDeploy.

## Disaster Recovery

* RTO/RPO: Recovery Time Objective, Recovery Point Objective.
* Backup & Restore, Pilot Light, Warm Standby, Active/Active.

## Caching Strategies

* In-memory caching (ElastiCache), CDN caching, database caching.

## API Scaling

* Load balancing, request throttling, API versioning.

## Database Scaling

* Read Replicas, sharding, DynamoDB Global Tables.

**Key Takeaways:** Architecture depends on requirements.  Prioritize resilience, scalability, and cost optimization.

---
# AWS Cheatsheet - Quick Revision

## Key Services

| Service        | Description              |
|----------------|--------------------------|
| EC2            | Virtual Server           |
| S3             | Object Storage           |
| DynamoDB       | NoSQL Database           |
| Lambda         | Serverless Compute       |
| ECS/EKS        | Container Orchestration  |
| CloudFront     | CDN                      |
| API Gateway    | API Management           |

## Common Commands (CLI)

* `aws s3 ls` - List S3 objects.
* `aws ec2 describe-instances` - Describe EC2 instances.
* `aws cloudformation create-stack` - Create CloudFormation stack.

## Important Comparisons

| Feature | RDS | DynamoDB |
|---|---|---|
| Data Model | Relational | NoSQL |
| Scalability | Vertical | Horizontal |
| Use Cases | Transactions, Complex Queries | High Throughput, Low Latency |

| Feature | SQS | SNS |
|---|---|---|
| Messaging Type | Queue | Pub/Sub |
| Use Cases | Asynchronous Processing | Notifications |

## Frequently Forgotten Concepts

* **Security Groups are stateful.** NACLs are stateless.
* **IAM Roles are used by Lambda, ECS/EKS.**
* **VPC peering connects two VPCs.**

## Service Limits

* Check AWS Service Limits documentation. Exceeding them can impact functionality.

## Security Reminders

* **Principle of Least Privilege:** Grant only necessary permissions.
* **Enable MFA for all IAM users.**

## Architecture Tips

* **Use Infrastructure as Code (CloudFormation).**
* **Automate everything.**

## Cost Optimization Reminders

* **Right-size EC2 instances.**
* **Utilize S3 storage tiers.**
* **Delete unused resources.**

**Interview Tips:** Be prepared to explain your reasoning behind architectural choices.

**Common Mistakes:** Not understanding service boundaries.

**Key Takeaways:** This is a quick reference; deeper understanding is essential for production environments.

# AWS Interview Questions - Revision Notes (Senior Engineer Focus)

## Beginner

1. **What is the difference between S3 and EBS?**  S3 is object storage, EBS is block storage for EC2.
    * **Answer:** "S3 is designed for storing unstructured data as objects – think images, videos, documents. It's incredibly scalable and durable, ideal for static content. EBS, on the other hand, provides block-level storage volumes directly attached to EC2 instances, much like a hard drive. EBS is well-suited for operating systems, databases (where you need consistent I/O), and applications requiring persistent, local storage. The key difference is that S3 data is accessed via HTTP/HTTPS, while EBS volumes are directly accessed by the EC2 instance."
2. **Explain VPCs.** Isolated network in AWS.
    * **Answer:** "A VPC (Virtual Private Cloud) allows me to define a logically isolated section of the AWS cloud where I can launch AWS resources in a virtual network that I define. I can control the IP address range, subnets, route tables, network gateways, and security settings. It’s like having my own data center within AWS. Crucially, it provides isolation and control over network access, ensuring security and compliance."
3. **What is IAM?** Manages access to AWS resources.
    * **Answer:**  "IAM (Identity and Access Management) is the foundation of AWS security. It allows me to control who (users, groups, roles) has access to which AWS resources. I define granular permissions using policies—JSON documents that specify allowed actions.  IAM roles are particularly useful for services and applications, as they provide temporary credentials without needing to hardcode access keys."

## Intermediate

4. **Describe the difference between Security Groups and NACLs.** (See aws-core-services.md)
    * **Answer:** "Both Security Groups and NACLs control network traffic, but they operate differently. Security Groups are stateful – they track connections and allow return traffic automatically. They operate at the instance level. NACLs are stateless – you must explicitly allow inbound and outbound traffic. They operate at the subnet level and provide an additional layer of security. Because NACLs are stateless, they’re more complex to configure but offer more granular control over network traffic."
5. **When would you use DynamoDB instead of RDS?** High-throughput, low-latency, schema-less data.
    * **Answer:** “I’d choose DynamoDB over RDS when I need extremely high read/write throughput with very low latency.  DynamoDB’s horizontal scalability makes it ideal for applications with unpredictable or rapidly changing workloads.  Also, when I don’t need the structure and relational constraints of a traditional database and the schema is likely to evolve.  Considerations include the lack of complex joins and the potential need for eventual consistency."
6. **Explain how Auto Scaling works.** Monitors metrics, launches/terminates instances.
    * **Answer:** “Auto Scaling Groups (ASGs) dynamically adjust the number of EC2 instances in response to demand.  I define a launch configuration/template specifying the instance type, AMI, and security group.  The ASG monitors CloudWatch metrics (like CPU utilization or queue length) and launches new instances when thresholds are exceeded and terminates instances when demand decreases.  I can define scaling policies based on these metrics, schedules, or even predicted traffic.”

## Advanced

7. **Design a highly available web application using AWS services.** (Includes VPC, Load Balancer, ASG, RDS/DynamoDB)
    * **Answer:** "For high availability, I’d structure the application within a VPC with multiple Availability Zones (AZs).  An Elastic Load Balancer (ELB) would distribute traffic across an Auto Scaling Group (ASG) of EC2 instances.  The ASG would use a Launch Template to configure its instances. I'd choose RDS (potentially with read replicas in different AZs) or DynamoDB for the database layer, depending on the data model requirements.  All components would be configured with health checks to ensure only healthy instances receive traffic.  Multi-AZ RDS or DynamoDB Global Tables would provide database resilience.  DNS failover with Route53 could be implemented as a last resort for regional outages."
8. **How would you troubleshoot a Lambda function that is timing out?** Review code, increase memory allocation, optimize dependencies.
    * **Answer:** "First, I’d examine the CloudWatch Logs for the Lambda function to identify any errors or bottlenecks. If the function is exceeding the timeout value, I'd look at the code for inefficient logic or long-running operations. Increasing the Lambda’s memory allocation often improves performance because it also increases CPU proportionally. Optimizing dependencies (reducing their size, using efficient libraries) and reviewing the execution environment can also help. I'd also enable X-Ray tracing to gain deeper insights into function performance."
9. **What are the tradeoffs between Microservices and a Monolithic Architecture on AWS?**
    * **Answer:** "A monolithic architecture is simpler to develop and deploy initially but can become difficult to manage and scale as it grows. Deployments are often all-or-nothing, and a single failure can impact the entire application. Microservices offer greater agility, independent scaling, and resilience. However, they introduce complexity – inter-service communication, distributed tracing, eventual consistency, and increased operational overhead.  Properly implemented microservices using ECS/EKS and API gateways are flexible but require greater skills and resource investment.”

## Scenario-Based

10. **You need to store user-uploaded images. What S3 features would you use to optimize costs and ensure durability?** S3 Lifecycle policies, Intelligent Tiering, Versioning.
    * **Answer:** "I’d use S3 Lifecycle policies to automatically move older, less frequently accessed images to cheaper storage tiers like S3 Standard-IA or Glacier. Intelligent Tiering automatically moves objects between access tiers to optimize costs. I’d also enable Versioning to protect against accidental deletions or overwrites. Server-Side Encryption (SSE) would be used to encrypt data at rest for security."
11. **You have an application experiencing high latency. How would you use AWS services to diagnose and mitigate the problem?** CloudWatch, X-Ray, CloudFront.
    * **Answer:** "I'd start with CloudWatch to monitor key metrics like CPU utilization, network I/O, and database query times. X-Ray would provide distributed tracing across the application, helping me pinpoint slow components. CloudFront would cache static content closer to users, reducing latency for those requests. Further analysis might involve RDS performance insights, or DynamoDB Acceleration for reads depending on the affected components."
12. **How would you implement a Blue/Green deployment strategy for a web application in AWS?** Utilize CodeDeploy, Load Balancer.
    * **Answer:** "I’d create two identical environments – Blue (live) and Green (new).  I’d deploy the new version to the Green environment.  Once testing is complete and all systems are healthy, I’d use a Load Balancer to switch traffic from the Blue environment to the Green environment.  The Blue environment remains as a rollback option. CodeDeploy can automate the deployments within each environment."

**Common Mistakes:**  Not considering security, over-engineering solutions.

**Key Takeaways:** Understand core concepts, be able to apply them to real-world scenarios. Practice system design.
