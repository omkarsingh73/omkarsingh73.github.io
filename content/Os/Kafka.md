# Table of Contents

- [[#1. Kafka Architecture Overview|1. Kafka Architecture Overview]]
- [[#2. Kafka Internals|2. Kafka Internals]]
- [[#3. Topics & Partitions|3. Topics & Partitions]]
- [[#4. Replication & Fault Tolerance|4. Replication & Fault Tolerance]]
- [[#5. Kafka Producer Deep Dive|5. Kafka Producer Deep Dive]]
- [[#6. Kafka Consumer Deep Dive|6. Kafka Consumer Deep Dive]]
- [[#7. Kafka Delivery Semantics|7. Kafka Delivery Semantics]]
- [[#8. Kafka Retention & Cleanup Policies|8. Kafka Retention & Cleanup Policies]]
- [[#9. Kafka Performance Tuning|9. Kafka Performance Tuning]]
- [[#10. Kafka Scaling Strategies|10. Kafka Scaling Strategies]]
- [[#11. Kafka Consumer Lag|11. Kafka Consumer Lag]]
- [[#12. Kafka with Spring Boot|12. Kafka with Spring Boot]]
- [[#13. Kafka Transactions|13. Kafka Transactions]]
- [[#14. Kafka Streams|14. Kafka Streams]]
- [[#15. Kafka Connect|15. Kafka Connect]]
- [[#16. Schema Registry & Avro|16. Schema Registry & Avro]]
- [[#17. Dead Letter Queue (DLQ)|17. Dead Letter Queue (DLQ)]]
- [[#18. Kafka Security|18. Kafka Security]]
- [[#19. Kafka Monitoring & Observability|19. Kafka Monitoring & Observability]]
- [[#20. Kafka Deployment & Operations|20. Kafka Deployment & Operations]]
- [[#21. Common Production Issues|21. Common Production Issues]]
- [[#22. Kafka Design Patterns|22. Kafka Design Patterns]]
- [[#23. Kafka Interview Questions|23. Kafka Interview Questions]]
- [[#24. Kafka Cheatsheet|24. Kafka Cheatsheet]]
- [[#25. Real-World Architecture Scenarios|25. Real-World Architecture Scenarios]]



---

## 1. Kafka Architecture Overview

### Core Components

```
Producers → [Broker Cluster] → Consumers
                 ↕
           ZooKeeper / KRaft
```

```
┌─────────────────────────────────────────────────────────┐
│                    Kafka Cluster                         │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐          │
│  │ Broker 1 │    │ Broker 2 │    │ Broker 3 │          │
│  │ (Leader  │    │ (Follow  │    │ (Follow  │          │
│  │  P0,P1)  │    │  er P0)  │    │  er P1)  │          │
│  └──────────┘    └──────────┘    └──────────┘          │
└─────────────────────────────────────────────────────────┘
       ↑                                    ↓
  Producer                          Consumer Group A
  (writes to leader)                 Consumer 1 → P0
                                     Consumer 2 → P1
```

### Component Roles

| Component | Role |
|---|---|
| **Broker** | Server storing and serving messages; one broker is controller |
| **Topic** | Logical category/feed of messages |
| **Partition** | Ordered, immutable log within a topic; unit of parallelism |
| **Offset** | Monotonically increasing ID per message within a partition |
| **Producer** | Writes records to topics |
| **Consumer** | Reads records from topics at its own pace |
| **Consumer Group** | Group of consumers sharing partition load |
| **ZooKeeper** | Metadata store, leader election (legacy, pre-3.x) |
| **KRaft** | Kafka's built-in consensus (Raft protocol), ZK replacement (Kafka 3.3+ stable) |

### Why Kafka is Fast — High Throughput Design

```
1. Sequential disk I/O (append-only log) — OS prefetches pages, avoids seek time
2. OS Page Cache — Linux caches log files; reads often served from memory
3. Zero-copy (sendfile syscall) — no user-space copy; kernel streams file → network
4. Batching — producers and consumers batch messages, reducing syscall overhead
5. Compression — snappy/lz4/zstd reduce bytes on wire and disk
6. Partition parallelism — multiple partitions = multiple consumers = horizontal scale
```

### ZooKeeper vs KRaft

| Aspect | ZooKeeper | KRaft |
|---|---|---|
| Metadata storage | External ZK cluster | Built-in `__cluster_metadata` topic |
| Controller election | ZK-based | Raft quorum |
| Startup complexity | Higher (2 systems) | Simpler (Kafka only) |
| Scalability ceiling | ~200K partitions | Millions of partitions |
| Status | Deprecated (Kafka 4.0 removes) | Production-ready (3.3+) |

---

## 2. Kafka Internals

### Log-Based Storage

```
Topic: orders, Partition 0
─────────────────────────────────────────────────────────
  Segment 0           Segment 1           Active Segment
  [00000000000.log]   [00000005000.log]   [00000009500.log]
  [00000000000.index] [00000005000.index] [00000009500.index]
  [00000000000.timeindex]
  (deleted after retention)
```

Each segment file contains:
- `.log` — actual message records
- `.index` — sparse offset → physical position index (binary search)
- `.timeindex` — timestamp → offset index (for time-based queries)

### Key Storage Configs

```properties
# Segment rolling
log.segment.bytes=1073741824       # 1GB per segment
log.roll.hours=168                 # 7 days max segment age

# Retention
log.retention.hours=168            # 7 days
log.retention.bytes=107374182400   # 100GB per partition
log.retention.check.interval.ms=300000

# Performance
log.flush.interval.messages=10000  # Fsync after N messages (rely on replication instead)
log.flush.interval.ms=1000         # Or time-based fsync
```

### Zero-Copy Deep Dive

```
Without zero-copy (traditional):
  Disk → kernel buffer → user buffer → socket buffer → NIC
  (4 copies, 2 context switches)

With zero-copy (sendfile):
  Disk → kernel buffer → NIC  (via DMA)
  (2 copies, 0 context switches to user space)
```

> **Production Tip:** Zero-copy is disabled when SSL is enabled — SSL requires data in user space for encryption. Factor this into latency expectations for secured clusters.

### Page Cache

Kafka doesn't manage its own cache — it relies on the OS page cache. Benefits:
- Producers write to page cache → OS asynchronously flushes to disk
- Consumers usually read from page cache (no disk I/O for recent messages)
- After broker restart, page cache is rebuilt (warm-up period)

> **Production Tip:** Allocate only ~50% of RAM to JVM heap. Rest goes to page cache. A broker with 64GB RAM should run with `-Xms6g -Xmx6g` and leave 58GB for OS page cache.

---

## 3. Topics & Partitions

### Partitioning Strategy

```java
ProducerRecord<String, String> record = new ProducerRecord<>(
    "orders",      // topic
    "user-123",    // key → deterministic partition assignment via hash
    orderJson      // value
);

// Partition selection:
// 1. Key present → partition = hash(key) % numPartitions
// 2. No key → sticky partitioner (batches to one partition, rotates on batch fill)
// 3. Custom partitioner → implement Partitioner interface
```

### Ordering Guarantees

```
Per-partition: GUARANTEED (offset monotonically increases)
Cross-partition: NOT GUARANTEED

Pattern: Same key → same partition → ordered delivery
Example: All events for userId=123 → partition 5 → processed in order
```

### Custom Partitioner

```java
public class TenantPartitioner implements Partitioner {
    @Override
    public int partition(String topic, Object key, byte[] keyBytes,
                         Object value, byte[] valueBytes, Cluster cluster) {
        int numPartitions = cluster.partitionCountForTopic(topic);
        String tenantId = extractTenantId(key.toString());
        return Math.abs(tenantId.hashCode()) % numPartitions;
    }
}
```

### Partition Count Planning

```
Partitions = max(throughput_target / single_partition_throughput,
                 desired_consumer_parallelism)

Rules:
- More partitions → more parallelism → more file handles, more memory
- Can INCREASE partitions (but breaks key-ordering for existing messages)
- Cannot DECREASE partitions (must delete and recreate topic)
- Typical: start with 6-12, scale to 100s for high-throughput
- Avoid > 4000 partitions per broker (resource limits)
```

> **Interview Note:** Increasing partitions on an existing topic does not redistribute existing messages. New messages with keys may now land on different partitions, breaking ordering for affected keys during the transition period.

---

## 4. Replication & Fault Tolerance

### Leader, Follower, ISR

```
Topic: payments, RF=3
                     Leader     Follower    Follower
                     Broker 1   Broker 2   Broker 3
Partition 0:         [L]        [F]        [F]
Partition 1:         [F]        [L]        [F]
Partition 2:         [F]        [F]        [L]

ISR = {Broker1, Broker2, Broker3}  (all caught up)
If Broker2 falls behind → ISR = {Broker1, Broker3}
```

### Replication Flow

```
Producer → Leader (writes to local log)
         ↓
     Followers fetch from leader (like a consumer)
         ↓
     Follower ACKs → added back to ISR
         ↓
     Leader advances High Watermark (HW)
         ↓
     Consumer can only read up to HW
```

### Critical Durability Configs

```properties
# Broker
default.replication.factor=3
min.insync.replicas=2          # Producer gets error if < 2 ISR replicas

# Producer
acks=all                       # Wait for all ISR replicas to acknowledge
retries=Integer.MAX_VALUE
enable.idempotence=true        # Exactly-once on producer side
```

### Failure Scenarios

| Scenario | Outcome |
|---|---|
| Leader fails, follower in ISR | Automatic leader election from ISR |
| Leader fails, no ISR follower | `unclean.leader.election.enable=false` → partition unavailable; `=true` → data loss possible |
| All replicas fail | Topic partition unavailable until a broker recovers |
| Network partition → ISR shrinks below min.insync.replicas | Producers fail with `NotEnoughReplicasException` |

```properties
# Never enable in production (data loss risk)
unclean.leader.election.enable=false
```

> **Interview Note:** `acks=all` with `min.insync.replicas=2` and `replication.factor=3` is the gold standard. It tolerates 1 broker failure with no data loss. With `replication.factor=2`, you have no redundancy.

---

## 5. Kafka Producer Deep Dive

### Producer Lifecycle & Batching

```
Application calls producer.send()
  ↓
RecordAccumulator (in-memory buffer, per partition)
  ↓ [linger.ms elapsed OR batch.size reached]
Sender thread (batches per broker)
  ↓
Network → Broker Leader
  ↓
ACK → release from buffer
```

### Production Producer Config

```properties
# Reliability
acks=all
retries=Integer.MAX_VALUE
enable.idempotence=true         # Prevents duplicate on retry (sequence numbers)
max.in.flight.requests.per.connection=5  # Max with idempotence; 1 for strict ordering

# Throughput
linger.ms=5                     # Wait up to 5ms to fill batch
batch.size=65536                 # 64KB batch (default 16KB)
buffer.memory=67108864           # 64MB total buffer (increase if producers back up)
compression.type=snappy          # snappy: fast; lz4: faster; zstd: best ratio

# Timeouts
request.timeout.ms=30000
delivery.timeout.ms=120000       # Total time including retries
```

### Idempotent Producer Internals

```
Each producer gets: ProducerID (PID) + sequence numbers per partition
Broker deduplicates using (PID, partition, sequence)
If retry arrives with same sequence → broker ignores duplicate
Guarantees exactly-once within a single producer session
```

### Transactional Producer

```java
Properties props = new Properties();
props.put("transactional.id", "order-service-tx-1");
props.put("enable.idempotence", "true");

KafkaProducer<String, String> producer = new KafkaProducer<>(props);
producer.initTransactions();

try {
    producer.beginTransaction();
    producer.send(new ProducerRecord<>("orders", key, orderJson));
    producer.send(new ProducerRecord<>("audit-log", key, auditJson));
    producer.commitTransaction();
} catch (ProducerFencedException | OutOfOrderSequenceException e) {
    producer.close(); // Fatal — cannot recover
} catch (KafkaException e) {
    producer.abortTransaction(); // Recoverable
}
```

### Delivery Semantics Summary

| Semantic | Config | Duplicate Risk | Data Loss Risk |
|---|---|---|---|
| At-most-once | `acks=0`, no retry | None | High |
| At-least-once | `acks=all`, retries > 0 | Yes | None |
| Exactly-once | `enable.idempotence=true` + transactions | None | None |

---

## 6. Kafka Consumer Deep Dive

### Consumer Group Architecture

```
Topic: orders (6 partitions)

Consumer Group A (3 consumers):
  Consumer 1 → P0, P1
  Consumer 2 → P2, P3
  Consumer 3 → P4, P5

Consumer Group B (6 consumers):
  Consumer 1 → P0
  Consumer 2 → P1  ... (1 partition per consumer)

Consumer Group C (7 consumers):
  1 consumer is IDLE (more consumers than partitions = waste)
```

### Poll Mechanism

```java
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    // poll() drives: heartbeats, rebalance detection, offset fetching

    for (ConsumerRecord<String, String> record : records) {
        processRecord(record);
    }

    // Commit after processing
    consumer.commitSync(); // Blocks until broker ACKs
    // OR
    consumer.commitAsync((offsets, ex) -> {
        if (ex != null) log.error("Commit failed", ex);
    });
}
```

### Critical Consumer Configs

```properties
enable.auto.commit=false              # Manual commit for reliability
auto.offset.reset=earliest            # earliest | latest | none
max.poll.records=500                  # Records per poll()
max.poll.interval.ms=300000           # Max processing time before rebalance triggered
session.timeout.ms=45000              # Heartbeat timeout
heartbeat.interval.ms=15000           # Send heartbeat every 15s (< session.timeout/3)
fetch.min.bytes=1048576               # 1MB minimum fetch (reduces requests)
fetch.max.wait.ms=500                 # Max wait if fetch.min.bytes not met
```

### Offset Management Strategies

```java
// 1. Commit after each record (low throughput, safest)
for (ConsumerRecord<String, String> record : records) {
    process(record);
    consumer.commitSync(Collections.singletonMap(
        new TopicPartition(record.topic(), record.partition()),
        new OffsetAndMetadata(record.offset() + 1)));
}

// 2. Commit after each batch (balanced)
processAll(records);
consumer.commitSync();

// 3. Async commit with sync on shutdown (production pattern)
consumer.commitAsync();
// On shutdown:
try { consumer.commitSync(); } finally { consumer.close(); }
```

### Rebalancing

**Triggers:** Consumer joins/leaves group, partition count changes, `max.poll.interval.ms` exceeded, heartbeat missed.

**Eager Rebalance (default pre-2.4):**
```
All consumers release ALL partitions → pause consumption
Coordinator assigns partitions → consumers resume
Stop-the-world for the group
```

**Cooperative/Incremental Rebalance (recommended):**
```
Only affected partitions are revoked/reassigned
Most consumers continue processing uninterrupted
Configure: partition.assignment.strategy=CooperativeStickyAssignor
```

```properties
partition.assignment.strategy=org.apache.kafka.clients.consumer.CooperativeStickyAssignor
group.instance.id=consumer-pod-1    # Static membership — prevents rebalance on restart
```

> **Production Tip:** Static membership (`group.instance.id`) prevents rebalance when a consumer restarts within `session.timeout.ms`. Critical for rolling deployments.

---

## 7. Kafka Delivery Semantics

### Comparison Table

| Semantic | Producer Config | Consumer Config | Duplicate Risk | Data Loss Risk | Use Case |
|---|---|---|---|---|---|
| **At-most-once** | `acks=0` | Auto commit before processing | None | High | Metrics, logs (loss OK) |
| **At-least-once** | `acks=all`, retries | Commit after processing | Yes (on failure) | None | Most business events |
| **Exactly-once** | Idempotence + transactions | `isolation.level=read_committed` | None | None | Payments, financial |

### Exactly-Once Semantics (EOS) End-to-End

```
Producer (idempotent + transactional)
  → Kafka (atomic multi-topic write)
  → Consumer (read_committed isolation)
  → Consumer offset commit inside transaction
```

```java
// Consumer side for EOS
props.put("isolation.level", "read_committed"); // Skip uncommitted/aborted messages
```

> **Interview Note:** EOS in Kafka means "exactly-once within Kafka." If your consumer writes to a database, you need idempotent consumers on the DB side too (e.g., upsert by event ID), or use Kafka Streams which handles EOS internally via state stores.

---

## 8. Kafka Retention & Cleanup Policies

### Retention Policies

```properties
# Time-based (default 7 days)
log.retention.hours=168
log.retention.minutes=      # More granular
log.retention.ms=604800000  # Milliseconds

# Size-based (per partition)
log.retention.bytes=10737418240  # 10GB

# Both: whichever is hit first triggers deletion
```

### Log Compaction

```
Cleanup policy = compact:
  Retains the LATEST value for each key
  Old messages for the same key are removed
  Tombstone (null value) → key is deleted

Use cases:
  - User profiles (keep latest state)
  - Configuration (keep latest setting)
  - Change Data Capture output topics

Timeline:
  offset 0: key=user1, val=Alice
  offset 5: key=user1, val=Alice Smith   ← retained
  offset 3: key=user2, val=Bob           ← retained
  After compaction: only offset 5 and 3 remain
```

```properties
cleanup.policy=compact
min.cleanable.dirty.ratio=0.5     # Compact when 50% is dirty
segment.ms=86400000               # Compact at most daily
delete.retention.ms=86400000      # Tombstones retained 24h for consumer catch-up
```

### Mixed Policy

```properties
cleanup.policy=compact,delete     # Compact AND enforce size/time retention
```

> **Interview Note:** Log compaction does NOT guarantee all historical records — it only guarantees the latest value per key. For full history, use time-based retention. Compaction happens in the background and is not instantaneous.

---

## 9. Kafka Performance Tuning

### Producer Tuning

```properties
# Throughput (sacrifice some latency)
linger.ms=20                    # Larger batches
batch.size=131072               # 128KB
compression.type=lz4            # Best speed/ratio balance
buffer.memory=134217728         # 128MB buffer

# Latency (sacrifice some throughput)
linger.ms=0
batch.size=16384
acks=1                          # Only leader ACK (not all ISR)
```

### Consumer Tuning

```properties
fetch.min.bytes=1048576         # 1MB — fewer fetch requests, higher throughput
fetch.max.wait.ms=500
max.poll.records=1000           # Larger batches per poll
max.partition.fetch.bytes=10485760  # 10MB per partition per fetch

# Parallelism
# Add consumers up to partition count
# Use multi-threaded processing with executor pool
```

### Broker Tuning

```properties
# Network
num.network.threads=8           # = num CPU cores
num.io.threads=16               # 2x CPU cores
socket.send.buffer.bytes=1048576    # 1MB
socket.receive.buffer.bytes=1048576
socket.request.max.bytes=104857600  # 100MB

# I/O
num.replica.fetchers=4          # Replication throughput
log.flush.interval.messages=10000   # Rely on replication; avoid frequent fsync

# Producer request handling
queued.max.requests=500
```

### Backpressure Handling

```java
// Producer: buffer.memory exceeded → blocks or throws
props.put("max.block.ms", "60000"); // Block 60s before throwing

// Consumer: slow consumer → offset lag grows
// Solutions:
// 1. Increase consumer instances (up to partition count)
// 2. Increase max.poll.records + optimize processing
// 3. Async processing with bounded queue
// 4. Scale out the service

ExecutorService executor = Executors.newFixedThreadPool(20);
consumer.poll(Duration.ofMillis(100)).forEach(record ->
    executor.submit(() -> processRecord(record)));
// CAUTION: Must track futures and commit offsets only after processing
```

> **Production Tip:** Never do blocking I/O (DB calls, HTTP) inside the Kafka poll loop without threading. A slow DB call can exceed `max.poll.interval.ms`, triggering a rebalance.

---

## 10. Kafka Scaling Strategies

### Horizontal Scaling Approach

```
Step 1: Add brokers to cluster
Step 2: Reassign partition leaders to new brokers
Step 3: Rebalance replicas using kafka-reassign-partitions.sh
Step 4: Monitor ISR during reassignment (network-intensive)

Partition Leaders distribute automatically after reassignment.
```

### Increasing Partitions

```bash
# Increase partitions (irreversible for ordering)
kafka-topics.sh --alter --topic orders --partitions 12 \
  --bootstrap-server broker:9092

# WARNING: Keys that previously went to partition X may now go to partition Y
# Ordering for existing keys is broken during transition
```

### Partition Count Planning Framework

```
Target throughput: 1 GB/s
Single partition throughput (conservative): 100 MB/s
  → Need 10 partitions minimum

Consumer processing rate: 50 messages/s per instance
Target: 500 messages/s total
  → Need 10 consumers → 10 partitions minimum

Recommendation: max(10, 10) = 10 partitions, create with buffer → 12 partitions
```

---

## 11. Kafka Consumer Lag

### Lag Formula

```
Consumer Lag (per partition) = Log End Offset - Current Consumer Offset

Group Lag = Σ partition lags across all partitions in group
```

### Monitoring Tools

| Tool | Description |
|---|---|
| `kafka-consumer-groups.sh` | Built-in CLI lag inspection |
| **Burrow** (LinkedIn) | Dedicated lag monitor with sliding window analysis |
| **Kafka Exporter** | Prometheus metrics for consumer lag |
| **Confluent Control Center** | Enterprise UI with lag visualization |
| **Grafana + Prometheus** | Dashboard with alerts on lag thresholds |

```bash
kafka-consumer-groups.sh --bootstrap-server broker:9092 \
  --group order-processor --describe

# Output:
# GROUP           TOPIC     PARTITION  CURRENT-OFFSET  LOG-END-OFFSET  LAG
# order-processor orders    0          15000           15100           100
```

### Lag Causes & Remediation

| Cause | Diagnosis | Fix |
|---|---|---|
| Slow message processing | CPU/DB profiling | Optimize processing, async I/O |
| Too few consumers | Lag growing linearly | Add consumers (up to partition count) |
| Upstream traffic spike | Sudden lag jump | Scale out consumers, increase max.poll.records |
| Rebalancing too frequent | Lag spikes during rebalance | Static membership, cooperative rebalance |
| GC pauses on consumer | JVM GC logs, poll interval exceeded | GC tuning, reduce max.poll.records |

---

## 12. Kafka with Spring Boot

### KafkaTemplate (Producer)

```java
@Service
public class OrderEventPublisher {
    private final KafkaTemplate<String, OrderEvent> kafkaTemplate;

    public void publish(OrderEvent event) {
        kafkaTemplate.send("orders", event.getOrderId(), event)
            .whenComplete((result, ex) -> {
                if (ex == null) {
                    log.info("Published offset={}", result.getRecordMetadata().offset());
                } else {
                    log.error("Publish failed for orderId={}", event.getOrderId(), ex);
                    // Alert / retry / fallback
                }
            });
    }
}
```

### @KafkaListener (Consumer)

```java
@Component
@Slf4j
public class OrderEventConsumer {

    @KafkaListener(
        topics = "orders",
        groupId = "order-processor",
        concurrency = "3",              // 3 consumer threads (≤ partition count)
        containerFactory = "kafkaListenerContainerFactory"
    )
    public void consume(
            ConsumerRecord<String, OrderEvent> record,
            Acknowledgment ack) {
        try {
            log.info("Processing order={} partition={} offset={}",
                record.key(), record.partition(), record.offset());
            processOrder(record.value());
            ack.acknowledge();          // Manual commit after success
        } catch (RecoverableException e) {
            throw e;                    // Let Spring retry
        } catch (NonRecoverableException e) {
            log.error("Poison message, sending to DLT", e);
            ack.acknowledge();          // Commit to skip; DLT handled by DeadLetterPublishingRecoverer
        }
    }
}
```

### Spring Kafka Configuration

```java
@Configuration
public class KafkaConfig {

    @Bean
    public ConcurrentKafkaListenerContainerFactory<String, OrderEvent>
           kafkaListenerContainerFactory(ConsumerFactory<String, OrderEvent> cf) {

        ConcurrentKafkaListenerContainerFactory<String, OrderEvent> factory =
            new ConcurrentKafkaListenerContainerFactory<>();
        factory.setConsumerFactory(cf);
        factory.getContainerProperties().setAckMode(AckMode.MANUAL_IMMEDIATE);
        factory.setCommonErrorHandler(errorHandler());
        return factory;
    }

    @Bean
    public DefaultErrorHandler errorHandler(KafkaTemplate<String, OrderEvent> template) {
        // Retry 3 times with exponential backoff, then send to DLT
        ExponentialBackOffWithMaxRetries backOff = new ExponentialBackOffWithMaxRetries(3);
        backOff.setInitialInterval(1000L);
        backOff.setMultiplier(2.0);

        DeadLetterPublishingRecoverer recoverer =
            new DeadLetterPublishingRecoverer(template,
                (r, e) -> new TopicPartition(r.topic() + ".DLT", r.partition()));

        return new DefaultErrorHandler(recoverer, backOff);
    }
}
```

### Retry Topics Pattern (Spring Kafka 2.7+)

```java
@RetryableTopic(
    attempts = "4",
    backoff = @Backoff(delay = 1000, multiplier = 2.0),
    dltTopicSuffix = ".DLT",
    retryTopicSuffix = "-retry"
)
@KafkaListener(topics = "orders")
public void consume(OrderEvent event) {
    // Automatically retried via: orders-retry-1, orders-retry-2, orders-retry-3
    // Then to: orders.DLT
    processOrder(event);
}
```

---

## 13. Kafka Transactions

### Transactional Guarantees

```
Transactional producer ensures:
1. Atomic multi-partition write (all or nothing)
2. Exactly-once semantics (with idempotence)
3. Consumers with read_committed see only committed messages
```

### Configuration

```properties
# Producer
transactional.id=order-service-instance-1    # Unique per producer instance
enable.idempotence=true                       # Required for transactions
acks=all                                      # Required

# Consumer
isolation.level=read_committed                # Skip aborted transactions
```

### Producer Fencing

When a new instance starts with the same `transactional.id`, the broker fences the old producer:
```
Old producer → ProducerFencedException (must close)
New producer → Takes over, can abort old pending transaction
```

> **Production Tip:** Set `transactional.id` to include instance/pod identity. For K8s: `order-service-${POD_NAME}`. Ensures each pod has a unique transaction ID but can recover across restarts.

### Read-Process-Write Pattern (EOS with consumer offset)

```java
producer.beginTransaction();
try {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    // Process and produce results
    for (ConsumerRecord<String, String> r : records) {
        producer.send(new ProducerRecord<>("output", transform(r)));
    }
    // Commit consumer offsets atomically with the produced messages
    producer.sendOffsetsToTransaction(getOffsets(records), consumer.groupMetadata());
    producer.commitTransaction();
} catch (Exception e) {
    producer.abortTransaction();
}
```

---

## 14. Kafka Streams

### Core Concepts

```
KStream  → Unbounded stream of records (each record = independent event)
KTable   → Changelog stream (latest value per key = current state)
GlobalKTable → Replicated to all instances (for lookups)

Stream → KStream.groupByKey().aggregate() → KTable
KTable → KTable.toStream() → KStream
```

### Simple Topology Example

```java
StreamsBuilder builder = new StreamsBuilder();

KStream<String, OrderEvent> orders = builder.stream("orders");

// Filter and transform
KStream<String, String> highValueOrders = orders
    .filter((key, order) -> order.getAmount() > 1000)
    .mapValues(order -> "HIGH_VALUE:" + order.getOrderId());

highValueOrders.to("high-value-orders");

// Aggregation with windowing
orders
    .groupBy((key, order) -> order.getRegion())
    .windowedBy(TimeWindows.ofSizeWithNoGrace(Duration.ofHours(1)))
    .aggregate(
        OrderStats::new,
        (key, order, stats) -> stats.addOrder(order),
        Materialized.<String, OrderStats, WindowStore<Bytes, byte[]>>as("order-stats-store")
            .withValueSerde(orderStatsSerde)
    )
    .toStream()
    .to("order-stats-hourly");

KafkaStreams streams = new KafkaStreams(builder.build(), streamsConfig);
streams.start();
```

### Windowing Types

| Window Type | Description | Use Case |
|---|---|---|
| Tumbling | Fixed, non-overlapping | Hourly aggregations |
| Hopping | Fixed size, overlapping (slide < size) | Rolling 1h window every 15min |
| Session | Activity-based, gap-defined | User session analytics |
| Sliding | Events within a time range | Real-time moving average |

### Kafka Streams vs Flink vs Spark Streaming

| Aspect | Kafka Streams | Apache Flink | Spark Streaming |
|---|---|---|---|
| Deployment | Library (no cluster) | Separate cluster | Spark cluster |
| Latency | Low (ms) | Very low (ms) | Higher (micro-batch) |
| State management | RocksDB | Managed | External |
| Learning curve | Low | High | Medium |
| Use case | Microservice-embedded | Complex stateful | Batch + streaming |

---

## 15. Kafka Connect

### Architecture

```
Source System (DB, API) → Source Connector → Kafka Topic
Kafka Topic → Sink Connector → Target System (ES, DB, S3)

Connectors run in:
  Standalone: single process (dev/testing)
  Distributed: multiple workers, fault-tolerant, horizontally scalable
```

### Common Connectors

| Connector | Direction | Use Case |
|---|---|---|
| Debezium (JDBC/MySQL/PG) | Source | CDC — capture DB changes |
| JDBC Sink | Sink | Write Kafka records to relational DB |
| Elasticsearch Sink | Sink | Index records in Elasticsearch |
| S3 Sink | Sink | Archive data to S3 (partitioned by time) |
| MongoDB Connector | Both | Sync with MongoDB |

### CDC with Debezium

```json
{
  "name": "orders-cdc",
  "config": {
    "connector.class": "io.debezium.connector.postgresql.PostgresConnector",
    "database.hostname": "postgres",
    "database.port": "5432",
    "database.user": "debezium",
    "database.password": "secret",
    "database.dbname": "orders_db",
    "table.include.list": "public.orders",
    "topic.prefix": "cdc",
    "plugin.name": "pgoutput",
    "slot.name": "debezium_slot"
  }
}
```

> **Production Tip:** Debezium reads from the DB's WAL/binlog. Ensure the replication slot doesn't grow unbounded — monitor `pg_replication_slots` in PostgreSQL. A stopped Debezium connector that doesn't consume will cause WAL to accumulate and fill disk.

---

## 16. Schema Registry & Avro

### Why Schema Registry

```
Problem: Producer changes message schema → Consumer breaks
Solution: Centralize schema management + enforce compatibility

Flow:
Producer → serialize with Avro → register schema → send (schemaId + bytes)
Consumer → read schemaId → fetch schema from registry → deserialize
```

### Compatibility Modes

| Mode | Description | Safe Change |
|---|---|---|
| `BACKWARD` | New schema reads old data | Add optional fields |
| `FORWARD` | Old schema reads new data | Remove optional fields |
| `FULL` | Both directions | Add/remove optional fields only |
| `NONE` | No compatibility check | Anything (dangerous) |

### Serialization Format Comparison

| Format | Schema Required | Human Readable | Size | Speed |
|---|---|---|---|---|
| JSON | No | Yes | Large | Slow |
| Avro | Yes (registry) | No | Small | Fast |
| Protobuf | Yes (.proto file) | No | Smallest | Fastest |
| Thrift | Yes | No | Small | Fast |

```java
// Avro producer with Schema Registry
props.put("key.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("value.serializer", "io.confluent.kafka.serializers.KafkaAvroSerializer");
props.put("schema.registry.url", "http://schema-registry:8081");
```

---

## 17. Dead Letter Queue (DLQ)

### DLQ Architecture

```
orders → Consumer → [processing failure]
                         ↓
                   Retry (1s, 2s, 4s exponential)
                         ↓ [max retries exceeded]
                   orders.DLT
                         ↓
                   DLT Consumer → Alert + manual review/replay
```

### Production DLQ Strategy

```java
@Bean
public DeadLetterPublishingRecoverer recoverer(KafkaTemplate<String, Object> template) {
    return new DeadLetterPublishingRecoverer(template,
        (record, exception) -> {
            // Route to different DLTs based on exception type
            if (exception.getCause() instanceof SerializationException) {
                return new TopicPartition("orders.DLT.serialization", record.partition());
            }
            return new TopicPartition("orders.DLT", record.partition());
        });
}
```

### DLT Headers (Spring adds automatically)

```
kafka_dlt-exception-cause-fqcn: com.example.OrderProcessingException
kafka_dlt-exception-message: Failed to process order
kafka_dlt-exception-stacktrace: ...
kafka_dlt-original-topic: orders
kafka_dlt-original-partition: 2
kafka_dlt-original-offset: 15432
kafka_dlt-original-timestamp: 1701234567890
```

### Poison Message Pattern

```java
// DLT consumer for replay / investigation
@KafkaListener(topics = "orders.DLT", groupId = "orders-dlt-handler")
public void handleDlt(ConsumerRecord<String, OrderEvent> record,
                      @Header("kafka_dlt-exception-message") String errorMsg) {
    log.error("DLT message: key={}, error={}", record.key(), errorMsg);
    alertingService.sendAlert(record, errorMsg);
    // Optionally: store in DB for ops team review and replay
}
```

---

## 18. Kafka Security

### Security Layers

```
1. Encryption (in-transit): SSL/TLS
2. Authentication: SASL (PLAIN, SCRAM, GSSAPI/Kerberos, OAUTHBEARER)
3. Authorization: ACLs (Access Control Lists)
4. Encryption at-rest: OS/disk level (not built into Kafka)
```

### Producer/Consumer Security Config

```properties
# Encryption + Authentication
security.protocol=SASL_SSL
sasl.mechanism=SCRAM-SHA-512

sasl.jaas.config=org.apache.kafka.common.security.scram.ScramLoginModule required \
  username="app-user" \
  password="secret";

ssl.truststore.location=/certs/kafka.client.truststore.jks
ssl.truststore.password=changeit
```

### ACL Management

```bash
# Grant producer access
kafka-acls.sh --bootstrap-server broker:9093 \
  --add --allow-principal User:order-service \
  --operation Write --topic orders

# Grant consumer group access
kafka-acls.sh --bootstrap-server broker:9093 \
  --add --allow-principal User:order-processor \
  --operation Read --topic orders \
  --operation Read --group order-processor-group

# List ACLs
kafka-acls.sh --bootstrap-server broker:9093 --list
```

### Security Best Practices

- Use `SASL_SSL` (not `SASL_PLAINTEXT`) in production
- Rotate credentials with SCRAM (no broker restart needed)
- Principle of least privilege: producers only Write; consumers only Read
- Use separate service accounts per microservice
- Enable audit logging via third-party (Confluent Audit, LinkedIn Cruise Control)

---

## 19. Kafka Monitoring & Observability

### Critical JMX Metrics

| Metric | Alert Threshold | Meaning |
|---|---|---|
| `UnderReplicatedPartitions` | > 0 | Replicas falling behind |
| `ActiveControllerCount` | ≠ 1 | Controller failure |
| `OfflinePartitionsCount` | > 0 | Unavailable partitions |
| `RequestHandlerAvgIdlePercent` | < 0.2 (20%) | Broker overloaded |
| `NetworkProcessorAvgIdlePercent` | < 0.3 (30%) | Network threads saturated |
| Consumer `records-lag-max` | Per SLA | Consumer falling behind |
| `BytesInPerSec` / `BytesOutPerSec` | Per capacity | Throughput monitoring |

### Prometheus + Grafana Setup

```yaml
# kafka-exporter deployment
- name: kafka-exporter
  image: danielqsj/kafka-exporter:latest
  args:
    - --kafka.server=broker1:9092
    - --kafka.server=broker2:9092
  ports:
    - containerPort: 9308  # Prometheus scrape port
```

### Essential Grafana Dashboards

- Grafana Dashboard ID **7589** — Kafka Overview
- Grafana Dashboard ID **12483** — Consumer Lag

### Log Analysis with ELK

```
Kafka Broker Logs → Filebeat → Logstash → Elasticsearch → Kibana
Search for: "LeaderEpochCache" (leader election), "ERROR", "WARN", rebalance events
```

---

## 20. Kafka Deployment & Operations

### Docker Compose (Dev)

```yaml
version: '3.8'
services:
  broker:
    image: confluentinc/cp-kafka:7.5.0
    environment:
      KAFKA_NODE_ID: 1
      KAFKA_PROCESS_ROLES: broker,controller          # KRaft mode
      KAFKA_LISTENERS: PLAINTEXT://:9092,CONTROLLER://:9093
      KAFKA_ADVERTISED_LISTENERS: PLAINTEXT://broker:9092
      KAFKA_CONTROLLER_QUORUM_VOTERS: 1@broker:9093
      KAFKA_OFFSETS_TOPIC_REPLICATION_FACTOR: 1
      CLUSTER_ID: "MkU3OEVBNTcwNTJENDM2Qg"
    ports:
      - "9092:9092"
```

### Kubernetes with Strimzi

```yaml
apiVersion: kafka.strimzi.io/v1beta2
kind: Kafka
metadata:
  name: production-kafka
spec:
  kafka:
    replicas: 3
    storage:
      type: persistent-claim
      size: 500Gi
      class: fast-ssd
    config:
      default.replication.factor: 3
      min.insync.replicas: 2
      log.retention.hours: 168
    resources:
      requests:
        memory: 8Gi
        cpu: "2"
      limits:
        memory: 16Gi
        cpu: "4"
  zookeeper:    # Or use KRaft
    replicas: 3
    storage:
      type: persistent-claim
      size: 50Gi
```

### StatefulSet Considerations

- Use `StatefulSets` for stable pod identities (broker-0, broker-1, broker-2)
- Use `PersistentVolumeClaims` with `Retain` policy — data survives pod deletion
- Use `anti-affinity` rules — brokers on separate nodes
- Use `PodDisruptionBudgets` — prevent simultaneous eviction

### Strimzi vs Confluent Platform

| Aspect | Strimzi | Confluent Platform |
|---|---|---|
| License | Open-source (Apache 2.0) | Commercial |
| Schema Registry | External (Apicurio) | Included |
| RBAC / ACL | Manual | Enterprise management |
| Support | Community | Enterprise SLA |
| Cost | Free | Expensive |

---

## 21. Common Production Issues

### Rebalance Storms

```
Symptom: Constant consumer rebalancing, high lag spikes
Causes:
  - Processing time > max.poll.interval.ms
  - Frequent consumer restarts (rolling deploy without static membership)
  - Too many consumers for partition count

Fix:
  - group.instance.id for static membership
  - Reduce max.poll.records
  - Switch to CooperativeStickyAssignor
  - Async processing to keep poll loop fast
```

### Disk Full

```
Symptom: Broker fails, logs stop, producers get errors
Causes:
  - Retention too long + high throughput
  - Compaction falling behind
  - Log directory permissions

Fix:
  - Monitor disk with >80% alert threshold
  - Reduce log.retention.hours or log.retention.bytes
  - Add brokers and reassign partitions
  - kafka-log-dirs.sh to identify large topics
```

### Message Duplication

```
Causes:
  1. Producer retry without idempotence → duplicate on broker
  2. Consumer commit before processing fails → re-reads after crash
  3. Consumer commit after processing fails → reprocesses after crash

Fix:
  1. enable.idempotence=true (Producer)
  2. Commit AFTER processing (Consumer)
  3. Idempotent consumers (DB upsert by event ID)
```

### Troubleshooting Checklist

```
1. Under-replicated partitions? → kafka-topics.sh --describe, check broker health
2. Consumer lag growing? → kafka-consumer-groups.sh --describe
3. Producer failures? → Check acks, buffer.memory, connection to broker
4. Rebalancing? → logs for "Rebalancing" events, check max.poll.interval.ms
5. High latency? → JMX RequestHandlerIdlePercent, num.io.threads, disk I/O
6. OOM on broker? → Heap dump, reduce log.cleaner.dedupe.buffer.size
7. Network bottleneck? → BytesInPerSec vs NIC capacity
```

---

## 22. Kafka Design Patterns

### Event Sourcing

```
State = Replay of all events from the beginning (or snapshot)
Kafka as event store: infinite retention, compaction for snapshots

Topic: account-events (key=accountId)
  offset 0: AccountOpened {id:123, owner: Alice}
  offset 1: MoneyDeposited {id:123, amount:500}
  offset 5: MoneyWithdrawn {id:123, amount:100}
Current state = replay → balance: 400
```

### CQRS with Kafka

```
Write side:
  API → Command → Domain Model → Event → Kafka topic

Read side:
  Kafka Consumer → Project event → Read Model (Elasticsearch / Redis)
  Query API → Read Model (optimized for reads)
```

### Saga Pattern (Choreography)

```
Order Service → OrderCreated → Kafka
                                  ↓
                          Payment Service → PaymentProcessed → Kafka
                                                                  ↓
                                                       Inventory Service → Reserved
                                                                  ↓
                                                         Order Service → OrderConfirmed

On failure:
  PaymentFailed → Kafka
                    ↓
           Order Service → OrderCancelled (compensating transaction)
```

### Outbox Pattern

```
Problem: Write to DB and Kafka atomically
Solution: Write to DB "outbox" table in same transaction → separate poller publishes to Kafka

DB Transaction:
  INSERT INTO orders VALUES (...)
  INSERT INTO outbox VALUES (topic='orders', key=orderId, payload=orderJson)

Outbox Poller (Debezium CDC or scheduled job):
  SELECT FROM outbox WHERE published=false
  → kafka.send(record)
  → UPDATE outbox SET published=true

Guarantees: No message loss, no dual-write inconsistency
```

### Retry Pattern

```
Immediate Retry → Short delay retry (1s) → Longer delay retry (60s) → DLT

Implementation with Spring Kafka RetryableTopic:
  orders → orders-retry-1 (delay 1s) → orders-retry-2 (delay 60s) → orders.DLT
```

---

## 23. Kafka Interview Questions

### Intermediate Level

**Q: How does Kafka ensure message ordering?**
A: Ordering is guaranteed within a partition only. Messages with the same key always go to the same partition (via `hash(key) % partitions`), ensuring ordered delivery for that key. Cross-partition ordering is not guaranteed. If total ordering is needed → use 1 partition (sacrifices parallelism).

**Q: What is the High Watermark and why does it matter for consumers?**
A: HW is the offset up to which all ISR replicas have acknowledged. Consumers can only read messages up to the HW, ensuring they never read data that might be lost (not yet replicated). This prevents reading uncommitted data after a leader failover.

**Q: Explain `auto.offset.reset` behavior.**
A: `earliest` → start from beginning if no committed offset. `latest` → start from newest if no committed offset. `none` → throw exception if no committed offset. Applies only when a consumer group has no committed offset for a partition (new group, or offset expired).

**Q: What happens when a consumer's `max.poll.interval.ms` is exceeded?**
A: The broker considers the consumer dead and triggers a group rebalance. The partition is reassigned to another consumer. The original consumer gets a `CommitFailedException` when it tries to commit, since it's no longer part of the group.

**Q: Difference between `commitSync()` and `commitAsync()`?**
A: `commitSync()` blocks until the broker confirms, retries on retriable failures, safe for shutdown. `commitAsync()` non-blocking, no retry on failure (retrying could commit stale offsets due to out-of-order async callbacks). Production pattern: use `commitAsync()` in the poll loop, `commitSync()` on graceful shutdown.

### Advanced Level

**Q: How does the idempotent producer prevent duplicates?**
A: On init, the broker assigns a `ProducerID (PID)`. Each message gets a monotonically increasing sequence number per partition. On retry, the broker checks (PID, partition, sequence) — if it already exists, it deduplicates and returns success. If the sequence is out of order, it throws an `OutOfOrderSequenceException`.

**Q: Explain Kafka's replication protocol and what happens during leader election.**
A: Followers fetch messages from the leader like consumers. When caught up (within `replica.lag.time.max.ms`), they're added to ISR. On leader failure: ZooKeeper/KRaft detects loss, the controller selects the first ISR member as new leader, updates metadata, brokers are notified. Non-ISR followers can be elected only if `unclean.leader.election.enable=true` (data loss risk).

**Q: How does log compaction work internally?**
A: The log cleaner thread reads old segments, builds a map of key → latest offset, then copies only latest-value messages to a new cleaned segment. Tombstones (null values) are retained for `delete.retention.ms` then removed. Compaction runs in the background and doesn't block reads. Active (head) segment is never compacted.

**Q: What is the difference between `isolation.level=read_uncommitted` and `read_committed`?**
A: `read_uncommitted` (default) reads all messages including those from aborted transactions. `read_committed` skips messages from aborted transactions and buffers messages from open transactions until they commit. Necessary for EOS consumer side.

**Q: How does Kafka handle a split-brain scenario?**
A: Kafka uses ZooKeeper/KRaft for single controller election — prevents split-brain for cluster metadata. For partitions, ISR + min.insync.replicas ensures consistency: if two network-partitioned sets try to write, only the side with the leader and enough ISR can successfully write (`NotEnoughReplicasException` on the minority side).

### Architect Level

**Q: Design a payment event system with exactly-once guarantees.**
A: Producer: `enable.idempotence=true`, `transactional.id=payment-service-${instance}`, `acks=all`. Topic: RF=3, min.insync.replicas=2. Consumer: `isolation.level=read_committed`, manual offset commit, idempotent processing (upsert by payment event ID in DB). For consume-process-produce (e.g., payment → ledger update → notification): use transactional read-process-write pattern with `sendOffsetsToTransaction`.

**Q: How would you handle a 10x traffic spike in a Kafka-based system?**
A: Pre-spike: partition count should already be sized for peak (cannot decrease). During spike: add broker nodes, reassign partitions to new brokers, scale consumer instances horizontally. Producer side: increase `buffer.memory`, `linger.ms` for better batching. Consumer side: increase `max.poll.records`, optimize processing (async DB writes, connection pooling). Monitor: consumer lag and broker `RequestHandlerIdlePercent`.

**Q: Compare Kafka to RabbitMQ for a high-throughput event sourcing system.**
A: Kafka wins for: replay (consumers can re-read), retention (days/weeks), throughput (millions/sec), partitioned ordering, Streams processing, log compaction for state. RabbitMQ wins for: complex routing (exchanges, bindings), per-message TTL/priority, traditional RPC patterns, lower operational complexity for small scale. For event sourcing → Kafka is the clear choice (durable log, replay, compaction).

---

## 24. Kafka Cheatsheet

### Topic Management

```bash
# Create topic
kafka-topics.sh --create --topic orders \
  --partitions 12 --replication-factor 3 \
  --config retention.ms=604800000 \
  --bootstrap-server broker:9092

# List topics
kafka-topics.sh --list --bootstrap-server broker:9092

# Describe topic (partition leaders, ISR, replicas)
kafka-topics.sh --describe --topic orders --bootstrap-server broker:9092

# Increase partitions
kafka-topics.sh --alter --topic orders --partitions 24 --bootstrap-server broker:9092

# Delete topic
kafka-topics.sh --delete --topic orders --bootstrap-server broker:9092

# Change retention
kafka-configs.sh --alter --entity-type topics --entity-name orders \
  --add-config retention.ms=86400000 --bootstrap-server broker:9092
```

### Producer / Consumer CLI

```bash
# Produce messages
kafka-console-producer.sh --topic orders --bootstrap-server broker:9092 \
  --property "key.separator=:" --property "parse.key=true"

# Consume from beginning
kafka-console-consumer.sh --topic orders --from-beginning \
  --group test-group --bootstrap-server broker:9092

# Consume with key display
kafka-console-consumer.sh --topic orders \
  --property "print.key=true" --property "key.separator=:" \
  --bootstrap-server broker:9092
```

### Consumer Group Operations

```bash
# List consumer groups
kafka-consumer-groups.sh --list --bootstrap-server broker:9092

# Describe group (LAG!)
kafka-consumer-groups.sh --describe --group order-processor \
  --bootstrap-server broker:9092

# Reset offsets (to beginning)
kafka-consumer-groups.sh --reset-offsets --group order-processor \
  --topic orders --to-earliest --execute \
  --bootstrap-server broker:9092

# Reset to specific offset
kafka-consumer-groups.sh --reset-offsets --group order-processor \
  --topic orders:2 --to-offset 5000 --execute \
  --bootstrap-server broker:9092

# Reset to timestamp
kafka-consumer-groups.sh --reset-offsets --group order-processor \
  --topic orders --to-datetime 2024-01-01T00:00:00.000 --execute \
  --bootstrap-server broker:9092
```

### Broker Debugging

```bash
# Log dirs and sizes
kafka-log-dirs.sh --describe --topic-list orders \
  --bootstrap-server broker:9092

# Broker configs
kafka-configs.sh --describe --entity-type brokers --entity-name 1 \
  --bootstrap-server broker:9092

# Reassign partitions
kafka-reassign-partitions.sh --execute \
  --reassignment-json-file reassignment.json \
  --bootstrap-server broker:9092

# Verify reassignment
kafka-reassign-partitions.sh --verify \
  --reassignment-json-file reassignment.json \
  --bootstrap-server broker:9092
```

### Performance Checklist

- [ ] `acks=all` + `min.insync.replicas=2` + `replication.factor=3`
- [ ] `enable.idempotence=true` on producers
- [ ] `linger.ms=5-20` + `batch.size=65536+` for throughput
- [ ] `compression.type=lz4` or `snappy`
- [ ] `enable.auto.commit=false` on consumers
- [ ] `max.poll.interval.ms` > actual processing time
- [ ] `CooperativeStickyAssignor` for rebalance strategy
- [ ] `group.instance.id` for static membership
- [ ] HikariCP or connection pool for DB-writing consumers
- [ ] Separate executor for processing (keep poll loop fast)
- [ ] Monitor: consumer lag, under-replicated partitions, disk usage

### Consumer Lag Checklist

- [ ] `kafka-consumer-groups.sh --describe` — is lag growing?
- [ ] Is `max.poll.interval.ms` being exceeded? (check for rebalances)
- [ ] Is processing blocked on I/O? (profile the consumer)
- [ ] Is partition count equal to or greater than consumer count?
- [ ] Is the consumer healthy? (GC pauses, OOM?)
- [ ] Can parallelism increase? (add consumers, increase concurrency)
- [ ] Is traffic spike temporary? (backlog will drain)

---

## 25. Real-World Architecture Scenarios

### Order Processing System

```
Customer API → order-events (topic, 12 partitions, key=orderId)
                    ↓
         ┌──────────────────────┐
         │  Order Processor     │ (3 instances × 4 threads = 12 consumers)
         │  - Validate order    │
         │  - Reserve inventory │
         │  - Publish result    │
         └──────────────────────┘
                    ↓
    order-confirmed / order-failed (topics)
                    ↓
    Notification Service (email/SMS)

Partitioning: key=orderId → all events for same order → same consumer → in-order processing
Reliability: acks=all, idempotent producer, manual ack consumer
Scaling: increase partitions + consumers for peak season
```

### CDC Pipeline (Change Data Capture)

```
PostgreSQL → Debezium Connector → cdc.orders topic
                                        ↓
                              Stream Processor (Kafka Streams)
                              - Transform schema
                              - Enrich with lookup data
                                        ↓
                    ┌───────────────────┴──────────────────┐
              Elasticsearch                              Data Warehouse
              (search/query)                       (BigQuery via S3 Sink)

Key decisions:
- Debezium topic per table (cdc.orders, cdc.users)
- Log compaction on CDC topics (keep latest state)
- Schema Registry for Avro schemas
- At-least-once + idempotent sink connectors
```

### Payment Events — High Reliability

```
Payment API
  ↓ (Outbox pattern — DB + outbox table in same TX)
Outbox Poller / Debezium
  ↓
payment-events (RF=3, min.insync.replicas=2, acks=all)
  ↓
Payment Processor (isolation.level=read_committed, EOS)
  ↓
  ├── Ledger DB (idempotent upsert by payment_event_id)
  ├── fraud-signals (topic)
  └── payment-confirmed (topic → Notification Service)

Failure handling:
  - payment-events.DLT → ops alert → manual review → replay
```

### Log Aggregation Pipeline

```
Services (K8s pods)
  ↓ (Filebeat / Fluent Bit)
application-logs (topic, cleanup.policy=delete, retention=3 days)
  ↓
  ├── Logstash → Elasticsearch → Kibana (search/alert)
  └── Stream Processor → error-logs (filtered topic) → PagerDuty
```

### Analytics Pipeline

```
Events (user-actions topic, retention=30 days)
  ↓
Kafka Streams (sessionize, aggregate, enrich)
  ↓
enriched-events (topic)
  ↓
  ├── S3 Sink Connector (partitioned by date/hour)
  │     ↓ Athena / Spark (batch analytics)
  └── Flink (real-time analytics → dashboard)
```

### Notification System — Fan-out

```
order-confirmed (topic)
  ↓
Notification Router (KStreams branch/filter)
  ↓
  ├── email-notifications (topic) → Email Consumer (SendGrid)
  ├── sms-notifications (topic)   → SMS Consumer (Twilio)
  └── push-notifications (topic)  → Push Consumer (FCM/APNs)

Each channel: independent consumer group, DLT, retry, rate limiting
Partitioning: key=userId → ordered notifications per user
```

---

*Apache Kafka Advanced Notes — Updated for Kafka 3.x, KRaft mode, Spring Kafka 3.x / Spring Boot 3.x*
