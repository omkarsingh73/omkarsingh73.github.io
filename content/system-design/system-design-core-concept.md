- [[#Scalability|Scalability]]
- [[#Distributed Systems|Distributed Systems]]
- [[#Database Design|Database Design]]
- [[#Caching|Caching]]
- [[#Messaging/Event Systems|Messaging/Event Systems]]
- [[#Reliability|Reliability]]
- [[#API Design|API Design]]
- [[#Security|Security]]

## Scalability

### Horizontal vs Vertical Scaling
| **Horizontal Scaling**              | **Vertical Scaling**               |
|-------------------------------------|------------------------------------|
| Add more instances                  | Upgrade instance size              |
| **Use Case**: Web services, stateless APIs | **Use Case**: Single-node databases, legacy apps |
| **Bottlenecks**: Database locks, stateful services | **Bottlenecks**: Instance limits, cost |
| **Tradeoffs**: Complexity vs elasticity | **Tradeoffs**: Diminishing returns, downtime |

**Stateless Services**  
- **Why**: Enables horizontal scaling, simplifies load balancing  
- **Examples**: REST APIs, Lambda, Fargate  
- **Mistakes**: Storing session in memory, local file system  

**Throughput vs Latency**  
- **Throughput**: Total requests/sec (scale out)  
- **Latency**: Response time (optimize code/db indexes)  

**CAP Theorem**  
| **Consistency** | **Availability** | **Partition Tolerance** |
|-----------------|-------------------|-------------------------|
| All nodes see same data | Service always responds | Network failures handled |
| **Tradeoff**: Choose 2 of 3 | **Real-World**: DynamoDB (AP), PostgreSQL (CP) |

**Interview Tips**  
- Discuss tradeoffs between AP vs CP systems  
- Explain how to handle eventual consistency in user-facing apps  

**Key Takeaway**  
> Horizontal scaling is king for web-scale systems. Make services stateless first.

---

## Distributed Systems

### Load Balancing
| **Type**          | **Use Case**                     | **Example**       |
|-------------------|----------------------------------|-------------------|
| Round-Robin       | Even distribution                | Nginx default     |
| Least Connections | Long-lived requests              | API Gateway       |
| IP Hash           | Session persistence              | ALB sticky sessions |

**Reverse Proxies**  
- **Why**: SSL termination, rate limiting, caching  
- **Tools**: Nginx, HAProxy, CloudFront, ALB  
- **Mistakes**: Overloading proxy with complex logic  

**Service Discovery**  
- **Patterns**:  
  - **Static**: Hardcoded IPs (small teams)  
  - **Dynamic**: Consul, Kubernetes Services, AWS Cloud Map  
- **Production Insight**: Combine with health checks to avoid dead nodes  

**Distributed Locking**  
- **Tools**: Redis `SET NX`, Zookeeper, etcd  
- **Use Case**: Inventory management, rate limiting  
- **Bottleneck**: Lock contention → use optimistic concurrency  

**Consensus Basics**  
- **Raft vs Paxos**: Raft is easier to understand (leader-based)  
- **When**: Config management, distributed databases  

**Interview Tip**  
> Explain tradeoffs between centralized (Zookeeper) vs decentralized (Raft) consensus.

---

## Database Design

### SQL vs NoSQL
| **SQL**                | **NoSQL**                      |
|------------------------|--------------------------------|
| ACID, joins, relations | Schema-less, vertical scaling  |
| **Use**: Financial data, reporting | **Use**: IoT, catalogs, sessions |
| **Bottleneck**: Write scalability | **Bottleneck**: Complex queries |

**Sharding**  
- **Key Selection**: Hash-based (even distro), range-based (time series)  
- **Example**: User ID modulo shard count  
- **Mistake**: Sharding by non-unique field  

**Replication**  
- **Master-Slave**: Read scaling, write bottleneck  
- **Multi-Master**: Conflict resolution needed (Cassandra, DynamoDB)  

**CQRS**  
- **Why**: Decouple reads/writes, optimize for different workloads  
- **Example**: Order processing → write model, analytics → read model  

**Index Pitfalls**  
- Over-indexing → write slowdown  
- Missing indexes → query latency spikes  

**Production Insight**  
> Use read replicas for analytics, but monitor replication lag (e.g., <5s for critical data)

---

## Caching

### Patterns
| **Pattern**       | **Flow**                          | **Use Case**              |
|-------------------|-----------------------------------|---------------------------|
| Cache-Aside       | App → Cache → DB                 | Dynamic data              |
| Write-Through     | App → Cache → DB (sync)           | Frequent updates          |
| Write-Behind      | App → Cache → DB (async)          | Bulk updates              |

**Redis vs Memcached**  
- Redis: Persistence, data structures, pub/sub  
- Memcached: Simpler, pure cache  

**Cache Invalidation**  
- **Strategies**:  
  - **TTL**: Simple but stale data risk  
  - **Write-Through**: Immediate sync  
  - **Event-Driven**: Kafka → cache purge  

**CDN Caching**  
- **Tradeoff**: Stale content vs reduced origin load  
- **Example**: CloudFront with `Cache-Control` headers  

**Interview Tip**  
> Discuss cache stampede mitigation (e.g., request coalescing in Redis)

---

## Messaging/Event Systems

### Queues vs Pub/Sub
| **Queues (SQS, RabbitMQ)** | **Pub/Sub (Kafka, SNS)**        |
|-----------------------------|---------------------------------|
| Single consumer             | Fan-out to multiple consumers   |
| DLQ support                 | High throughput (>1M msg/sec)   |

**Kafka Basics**  
- **Partitions**: Parallel processing units  
- **Consumer Groups**: Independent readers  
- **Use Case**: Event sourcing, log aggregation  

**SQS Visibility Timeout**  
- **Pitfall**: Incorrect timeout → duplicate processing  
- **Fix**: Set > max processing time + buffer  

**Event-Driven Architecture**  
- **Benefits**: Loose coupling, scalability  
- **Mistake**: Not handling event idempotency  

**Production Insight**  
> Use dead-letter queues to analyze failures without losing messages

---

## Reliability

### Retry & Circuit Breakers
| **Pattern**       | **Implementation**               | **Example**              |
|-------------------|-----------------------------------|--------------------------|
| Exponential Backoff | `retry-after` header, jitter    | Payment gateway retries  |
| Circuit Breaker   | Hystrix, Resilience4j            | API rate limiting        |

**Idempotency**  
- **Key**: Same request → same effect  
- **Implementation**: Request ID + server-side dedupe  

**Rate Limiting**  
- **Algorithms**:  
  - Token Bucket: Bursts allowed  
  - Leaky Bucket: Smooth outflow  

**Failover**  
- **DB**: Multi-AZ RDS automatic failover  
- **Services**: Active-passive vs active-active  

**Interview Tip**  
> Explain how to design for partial failures (e.g., DB down but API still responds)

---

## API Design

### REST vs GraphQL vs gRPC
| **REST**          | **GraphQL**         | **gRPC**              |
|-------------------|---------------------|-----------------------|
| Stateless, cacheable | Client-defined queries | Binary protocol, streaming |
| **Use**: Public APIs | **Use**: Mobile apps | **Use**: Internal microservices |

**Pagination**  
- **Keyset**: `WHERE id > last_id` (efficient)  
- **Offset**: Simple but slow for large datasets  

**Versioning**  
- **URI Version**: `/v1/users`  
- **Header Version**: `Accept: application/vnd.myapi.v2+json`  

**Tradeoff**: URI versioning is cache-friendly but harder to change  

---

## Security

### JWT vs OAuth2
| **JWT**             | **OAuth2**                  |
|---------------------|----------------------------|
| Stateless tokens    | Authorization framework    |
| **Use**: API auth   | **Use**: Third-party login |

**DDoS Protection**  
- **Layer 4**: CloudFront rate limiting  
- **Layer 7**: AWS WAF, custom rules  

**Encryption**  
- **In Transit**: TLS 1.3  
- **At Rest**: AES-256, KMS  

**Common Mistake**  
> Exposing internal APIs without auth (e.g., `0.0.0.0/0` in security groups)