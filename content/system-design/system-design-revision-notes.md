- [[#📚 Core Topics Overview|📚 Core Topics Overview]]
- [[#1. Scaling Patterns|1. Scaling Patterns]]
- [[#2. DB Comparisons|2. DB Comparisons]]
- [[#3. Cache Patterns|3. Cache Patterns]]
- [[#4. API Best Practices|4. API Best Practices]]
- [[#5. Distributed Systems Concepts|5. Distributed Systems Concepts]]
- [[#6. Load Balancing|6. Load Balancing]]
- [[#7. Messaging Patterns|7. Messaging Patterns]]
- [[#8. Reliability Patterns|8. Reliability Patterns]]
- [[#9. Security Reminders|9. Security Reminders]]
- [[#10. Performance Optimization|10. Performance Optimization]]
- [[#📊 Comparison Tables|📊 Comparison Tables]]
- [[#🎯 Key Takeaways|🎯 Key Takeaways]]

## 📚 Core Topics Overview  

| Category              | Key Focus                          |
|-----------------------|------------------------------------|
| **Scaling Patterns**  | Horizontal, sharding, read replicas |
| **DB Comparisons**    | SQL vs NoSQL, NewSQL, cloud DBs    |
| **Cache Patterns**    | Invalidation, write strategies     |
| **API Best Practices**| Versioning, idioms, security       |
| **Distributed Systems**| Consensus, service discovery       |
| **CAP Theorem**       | Tradeoffs in practice              |
| **Load Balancing**    | L4 vs L7, algorithms               |
| **Architecture Pts**  | Microservices, serverless, events  |
| **Messaging**         | Kafka vs RabbitMQ, patterns        |
| **Reliability**       | Retries, circuit breakers, chaos   |
| **Security**          | OAuth2, mTLS, secrets              |
| **Performance**       | Indexing, CDN, pooling             |

---

## 1. Scaling Patterns  

### Strategies  
| Pattern          | Use Case                         | Tradeoffs                          |
|------------------|----------------------------------|------------------------------------|
| **Horizontal**   | Web traffic spikes               | Complexity, data sharding needed   |
| **Vertical**     | Small, monolithic services       | Diminishing returns, cost          |
| **Sharding**     | >10M rows, write-heavy           | Cross-shard queries, rebalancing   |
| **Read Replicas**| Read-heavy (e.g., analytics)     | Replication lag, eventual consistency |
| **Citus (PostgreSQL)** | Distributed SQL      | Query planning complexity          |

### Interview Tips  
- **Sharding key**: Choose immutable, high-cardinality fields (user_id).  
- **Replicas vs Sharding**: Replicas for reads, sharding for writes.  

### Common Mistakes  
- **Sharding too early**: Premature optimization → operational overhead.  
- **Ignoring hotspot keys**: E.g., time-based shards causing skew.  

### Production Insights  
- **Amazon Aurora**: Serverless auto-scaling read replicas.  
- **MongoDB Sharding**: Use hashed shard keys for uniform distribution.  

### Key Takeaways  
✅ **Sharding > Replicas** for write scalability.  
✅ **Monitor replication lag** to avoid stale reads.  

---

## 2. DB Comparisons  

### SQL vs NoSQL  
| Feature          | SQL (PostgreSQL)         | NoSQL (MongoDB/Cassandra)         |
|------------------|--------------------------|-----------------------------------|
| **Consistency**  | Strong (ACID)            | Eventual (Cassandra) / Optional   |
| **Scaling**      | Vertical + read replicas | Horizontal (sharding built-in)    |
| **Queries**      | Complex joins            | Aggregation pipelines, limited joins |
| **Use Cases**    | Financial, inventory     | Logging, IoT, flexible schemas     |

### NewSQL Examples  
- **CockroachDB**: Geo-distributed, ACID.  
- **TiDB**: HTAP, MySQL-compatible.  

### Interview Tips  
- **Choose based on data relationships**: Joins → SQL; Schema flexibility → NoSQL.  

### Common Mistakes  
- **Forcing SQL onto NoSQL**: Trying joins in MongoDB.  
- **Ignoring write amplification**: Cassandra’s lightweight transactions.  

### Production Insights  
- **DynamoDB**: On-demand capacity for unpredictable loads.  
- **PostgreSQL Partial Indexes**: Optimize query performance.  

### Key Takeaways  
✅ **SQL for transactions**, **NoSQL for scale/flexibility**.  
✅ **NewSQL bridges the gap** for distributed ACID.  

---

## 3. Cache Patterns  

### Strategies  
| Pattern         | Flow                                | When to Use               |
|-----------------|-------------------------------------|---------------------------|
| **Cache-Aside** | Read → Cache miss? DB → Store cache | Most common, decoupled    |
| **Write-Through**| Write → DB + Cache sync             | Low write latency needed  |
| **Write-Behind**| Write → Queue → Async DB + Cache    | Batch writes, reduce DB load |
| **Cache-Aside Invalidate**| Explicit invalidation after writes | Avoid stale data          |

### Interview Tips  
- **Probabilistic expiration**: Use peterson’s algorithm to reduce thundering herds.  

### Common Mistakes  
- **Cache stampede**: Not using atomic locks or probabilistic delays.  
- **Over-cache**: Caching user PII without TTL → compliance issues.  

### Production Insights  
- **Redis Module**: Replicate cache logic (e.g., leaderboards).  
- **CDN Edge Cache**: Cache static assets + dynamic fragments (stale-while-revalidate).  

### Key Takeaways  
✅ **Cache-Aside** is safest for consistency.  
✅ **TTL + absolute expiration** for compliance.  

---

## 4. API Best Practices  

### Comparison: REST vs GraphQL vs gRPC  
| Aspect           | REST               | GraphQL            | gRPC               |
|------------------|--------------------|--------------------|--------------------|
| **Payload**      | JSON               | JSON (over HTTP)   | Protobuf (binary)  |
| **Over-fetch**   | High               | Low (specific fields) | Low              |
| **Streaming**    | Polling            | Subscriptions      | Built-in           |
| **Use Case**     | Public APIs        | Mobile/web clients | Internal services  |

### Key Patterns  
- **Versioning**: URL (`/v1/users`) or Header (`Accept: application/vnd.versioned+json`).  
- **Idempotency**: `Idempotency-Key` header for retries.  
- **Pagination**: Cursor-based > offset-based for large datasets.  

### Interview Tips  
- **GraphQL vs REST**: Defend choice based on client needs (mobile bandwidth vs server complexity).  

### Common Mistakes  
- **Versioning in headers**: Hard to enforce, use URL.  
- **Offset pagination**: Slow for deletions/insertions in middle.  

### Production Insights  
- **Stripe API**: Idempotency keys + exponential backoff.  
- **Google APIs**: gRPC + proto files for schema evolution.  

### Key Takeaways  
✅ **Cursor pagination** for infinite scrolls.  
✅ **Idempotency keys** for payment retries.  

---

## 5. Distributed Systems Concepts  

### CAP Theorem Summary  
```mermaid
graph TD
    C[Consistency] -->|Sacrifice| A[Availability]
    A -->|Sacrifice| P[Partition Tolerance]
    P -->|Sacrifice| C
````

- **Real-world**: **AP** (Cassandra) for analytics, **CP** (PostgreSQL) for payments.

### Consensus Algorithms

|Algorithm|Use Case|Tradeoffs|
|---|---|---|
|**Raft**|Config store (etcd)|Simpler than Paxos|
|**Paxos**|Legacy systems|Complex implementation|

### Service Discovery

- **Kubernetes Services**: DNS-based.
- **Consul**: Intent-based routing + health checks.

### Interview Tips

- **PACELC**: “If no partition, how to trade consistency vs latency?”

### Common Mistakes

- **Assuming network is reliable**: Design for partitions.
- **Over-using leader election**: Use CRDTs for low-latency updates.

### Production Insights

- **etcd**: Kubernetes bootstrap storage.
- **DynamoDB Global Tables**: Multi-region with conflict resolution.

### Key Takeaways

✅ **Design for partition tolerance first**.  
✅ **Use CRDTs** for distributed counters/merges.

---

## 6. Load Balancing

### Types

|Layer|Algorithm|Use Case|
|---|---|---|
|L4|Round-robin|Simple TCP load|
|L7|Least connections|HTTP, session-aware|
||Consistent hashing|Cache clusters, sharded DBs|

### Interview Tips

- **Sticky sessions**: Avoid unless necessary (breaks scalability).

### Common Mistakes

- **Using round-robin for stateful services**: Session loss.

### Production Insights

- **Envoy**: L7 proxy with circuit breaking.
- **AWS ALB**: Host-based routing + websockets.

### Key Takeaways

✅ **L7 for HTTP**, **L4 for TCP/UDP**.  
✅ **Health checks > 5s** to avoid false positives.

---

## 7. Messaging Patterns

### Kafka vs RabbitMQ

|Feature|Kafka|RabbitMQ|
|---|---|---|
|**Durability**|Log-based, persistent|Disk/ram queues|
|**Throughput**|100K+ msgs/sec|10K msgs/sec (depends)|
|**Use Case**|Event streaming, logs|Workqueues, RPC|
|**Ordering**|Per-partition|Per-queue|

### Patterns

- **Event Sourcing**: Store state changes as immutable events.
- **Request-Reply**: Correlation IDs + timeouts.

### Interview Tips

- **Kafka consumer lag**: Monitor with `consumer-offset - last-offset`.

### Common Mistakes

- **Using Kafka as a queue**: Not designed for strict message order across topics.

### Production Insights

- **Confluent Cloud**: Managed Kafka with geo-replication.
- **RabbitMQ Quorum Queues**: Durable, fast failover.

### Key Takeaways

✅ **Kafka for high-throughput**, **RabbitMQ for complex routing**.  
✅ **Dead-letter queues** for poison pills.

---

## 8. Reliability Patterns

### Core Techniques

|Pattern|Implementation|Tradeoffs|
|---|---|---|
|**Circuit Breaker**|Hystrix/Resilience4j|Latency overhead|
|**Bulkhead**|Thread pools per service|Resource underutilization|
|**Chaos Engineering**|Gremlin, Chaos Mesh|Uncovered bugs|

### Interview Tips

- **Fallback strategies**: “What if payment gateway is down?” → Use cached quotes.

### Common Mistakes

- **No timeout + retry**: Deadlock risk.

### Production Insights

- **Netflix Hystrix**: Fallback to cached content.
- **Google SRE**: Error budgets + SLIs.

### Key Takeaways

✅ **Circuit breakers > retries** for flaky dependencies.  
✅ **Error budgets** drive reliability decisions.

---

## 9. Security Reminders

|Layer|Practice|
|---|---|
|**Transport**|TLS 1.3, mutual TLS (mTLS)|
|**Auth**|OAuth2 + OIDC, JWT claims|
|**Secrets**|Vault, AWS Secrets Manager|
|**Auditing**|Immutable logs, SIEM integration|

### Interview Tips

- **Explain JWT vs OAuth2**: JWT is a token format; OAuth2 is an auth framework.

### Common Mistakes

- **Storing passwords in plaintext**: Always hash (bcrypt/scrypt).

### Production Insights

- **Hashicorp Vault**: Dynamic secrets for DB credentials.
- **Zero-Trust**: Verify every request (SPA, microservices).

### Key Takeaways

✅ **mTLS for service mesh**.  
✅ **Rotate secrets quarterly**.

---

## 10. Performance Optimization

### Techniques

|Area|Optimization|
|---|---|
|**DB**|Covering indexes, query hints|
|**APIs**|HTTP/2, gzip, edge caching|
|**Infra**|Connection pooling, kernel tuning|

### Interview Tips

- **Explain N+1 problem**: How to fix with eager loading or joins.

### Common Mistakes

- **Over-indexing**: Slows writes.

### Production Insights

- **Amazon Aurora Global Write Forwarding**: Low-latency cross-region writes.
- **pghero**: PostgreSQL query bottleneck detector.

### Key Takeaways

✅ **Profile before optimizing**.  
✅ **Cache warm-up** during deployments.

---

## 📊 Comparison Tables

### SQL vs NoSQL

|**Criteria**|**SQL**|**NoSQL**|
|---|---|---|
|**ACID**|Yes|Varies (e.g., DynamoDB transactional)|
|**Scaling**|Vertical + read replicas|Horizontal (sharding)|
|**Schema**|Fixed|Flexible|
|**Best For**|Financial, ERP|Analytics, IoT, flexible data|

### REST vs GraphQL vs gRPC

|**Metric**|**REST**|**GraphQL**|**gRPC**|
|---|---|---|---|
|**Payload**|JSON|JSON (client-specified)|Protobuf (binary)|
|**Efficiency**|Lower|Medium|High|
|**Streaming**|Polling|Subscriptions|Built-in|
|**Complexity**|Low|Medium (client/server)|High (proto definitions)|

### Kafka vs RabbitMQ

|**Aspect**|**Kafka**|**RabbitMQ**|
|---|---|---|
|**Model**|Publish-Subscribe|Point-to-Point / Pub-Sub|
|**Durability**|Log-based, persistent|Disk/ram queues|
|**Throughput**|100K+ msgs/sec|10K msgs/sec|
|**Use Case**|Event logging, streams|Work queues, RPC|

### Monolith vs Microservices

|**Factor**|**Monolith**|**Microservices**|
|---|---|---|
|**Deployment**|Single unit|Independent services|
|**Scaling**|Entire app|Per-service|
|**Complexity**|Lower|Higher (network, ops)|
|**Team Autonomy**|Low|High|

### Synchronous vs Asynchronous

|**When to Use**|**Synchronous**|**Asynchronous**|
|---|---|---|
|**Latency**|Critical (e.g., payments)|Non-critical (e.g., emails)|
|**Error Handling**|Immediate|Retry queues, DLQ|
|**Complexity**|Lower|Higher (messaging, idempotency)|

### Polling vs WebSockets

|**Aspect**|**Polling**|**WebSockets**|
|---|---|---|
|**Overhead**|HTTP request/response per poll|Persistent connection|
|**Latency**|Poll interval|Near real-time|
|**Use Case**|Infrequent updates|Chat, live dashboards|

---

## 🎯 Key Takeaways

1. **Tradeoffs > Theory**: Always map decisions to latency, cost, complexity.
2. **Observability First**: Metrics → Logs → Traces before scaling.
3. **Security by Default**: TLS, auth, secrets management non-negotiable.
4. **Chaos Ready**: Inject failures early in design.
5. **Cost-Aware**: Read replicas cheap; Kafka clusters expensive.

_🔍 **Interview Grand Slam**: Start with a simple diagram, quantify, discuss tradeoffs, end with observability._