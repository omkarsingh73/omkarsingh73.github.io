- [[#📚 Core Concepts Overview|📚 Core Concepts Overview]]
- [[#1. Thread Basics|1. Thread Basics]]
- [[#2. Synchronization|2. Synchronization]]
- [[#3. Thread Communication|3. Thread Communication]]
- [[#4. Thread Safety|4. Thread Safety]]
- [[#5. Executors Framework|5. Executors Framework]]
- [[#6. Concurrent Collections|6. Concurrent Collections]]
- [[#7. Locks|7. Locks]]
- [[#8. ThreadLocal|8. ThreadLocal]]
- [[#📊 Comparison Tables|📊 Comparison Tables]]
- [[#🔥 Production Troubleshooting|🔥 Production Troubleshooting]]
- [[#🎯 Interview Preparation Checklist|🎯 Interview Preparation Checklist]]
- [[#📌 Key Takeaways|📌 Key Takeaways]]

## 📚 Core Concepts Overview  

| Category             | Key Focus                          |
|----------------------|------------------------------------|
| **Thread Basics**    | Lifecycle, creation, state mgmt    |
| **Synchronization**  | Locks, monitors, visibility        |
| **Thread Comm**      | Wait/notify, producer-consumer     |
| **Thread Safety**    | Immutability, confinement, atomic |
| **Executors**        | Pool mgmt, scheduling, futures    |
| **Concurrent Coll**  | CHM, BlockingQueue, CopyOnWrite    |
| **Advanced Locks**   | Reentrant, ReadWrite, StampedLock |
| **ThreadLocal**      | Context, tracing, pitfalls         |

---

## 1. Thread Basics  

### Thread Lifecycle  
```mermaid
graph LR
    A[New] --> B[Runnable]
    B --> C[Blocked]
    C --> D[Waiting]
    D --> E[Timed Waiting]
    E --> F[Running]
    F --> G[Dead]
```

| State         | When Occurs                     | Pitfall                          |
|---------------|---------------------------------|-----------------------------------|
| **New**       | Thread object created           | Never started → resource leak    |
| **Runnable**  | Ready to run                    | Starvation if priority inverted  |
| **Blocked**   | Waiting for I/O or lock        | Long blocks → thread pool exhaustion |
| **Waiting**   | `wait()` called                | Forgetting `notify` → deadlock  |
| **Timed Waiting**| `sleep()`, I/O timeout      | Timeout miscalculations          |
| **Dead**      | Terminated                      | N/A                              |

### Runnable vs Callable  
| Feature          | Runnable                     | Callable                        |
|------------------|------------------------------|---------------------------------|
| **Return Value** | `void`                      | `V` (generic)                  |
| **Exception**    | No propagation               | `Future.get()` throws `ExecException` |
| **Use Case**     | Simple tasks                | Tasks needing result/exceptions|

### Thread Creation Approaches  
| Approach        | When to Use                  | Performance | Tradeoffs                     |
|-----------------|------------------------------|-------------|-------------------------------|
| **Extending Thread** | Quick PoCs                 | ❌          | Tight coupling, inheritance overhead |
| **Implementing Runnable** | Most production code    | ✅          | Single task, no state via `this` |
| **ExecutorService** | Scalable systems          | ✅✅         | Preferred: decouples algo from thread mgmt |

**Interview Tip**: *"Why not extend `Thread`?"* → Avoids thread management encapsulation.

**Common Mistake**: Creating threads manually in web apps → thread leak → OOM.

**Production Insight**: Use `ThreadPoolExecutor` with bounded queue to prevent uncontrolled thread creation.

**Key Takeaway**: **Always prefer `ExecutorService` over manual threads.**

---

## 2. Synchronization  

### `synchronized` Keyword  
| Aspect             | Details                                                                 |
|--------------------|-------------------------------------------------------------------------|
| **Intrinsic Lock** | Each object has one; `synchronized` acquires it implicitly             |
| **Monitor Lock**   | JVM-level monitor; prevents multiple threads entering same monitor     |
| **Object-Level**   | `synchronized(this)` → locks entire object                           |
| **Class-Level**    | `synchronized(MyClass.class)` → locks class loader instance           |

**Performance Bottleneck**: Fine-grained locking needed for high contention (e.g., per-user accounts).

**Real-World Example**:  
```java
class Account {
    private final ReentrantLock lock = new ReentrantLock();
    void transfer(Account dest, int amount) {
        lock.lock();                // Stage 1: lock source
        try {
            dest.lock();            // Stage 2: lock dest (deadlock risk!)
            // ...
        } finally {
            lock.unlock();
        }
    }
}
```

**Tradeoffs**:  
| Coarse-Grained (object-level) | Fine-Grained (per-field) |
|-------------------------------|--------------------------|
| ✅ Simple                     | ✅ Scalable              |
| ❌ Contention high           | ❌ Complexity           |

**Interview Tip**: *"How to avoid deadlock with two locks?"* → Always acquire locks in **consistent global order**.

**Production Insight**: Use `StampedLock` for optimistic reads in read-heavy systems.

---

## 3. Thread Communication  

### `wait()` / `notify()` / `notifyAll()`  

| Method        | Purpose                         | Common Pitfall                  |
|---------------|---------------------------------|----------------------------------|
| `wait()`      | Release lock, block thread     | Must be in synchronized block   |
| `notify()`    | Wake one waiting thread        | May wake thread that exits first |
| `notifyAll()` | Wake all waiting threads       | Unnecessary wakeups → wasted CPU |

**Producer-Consumer Pattern**  
```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>();

// Producer
queue.put(produceItem());

// Consumer
String item = queue.take(); // Blocks until item available
```

**Why It Matters**: Enables cooperative multitasking without busy-waiting.

**When NOT to Use**: Prefer `BlockingQueue` in modern code (handles waits/notify internally).

**Key Takeaway**: **`wait()` must be in loop to handle spurious wakeups.**

---

## 4. Thread Safety  

| Technique       | Use Case                          | When NOT to Use                  |
|-----------------|-----------------------------------|----------------------------------|
| **Immutable**   | Value objects (e.g., `record`)   | Frequently updated state         |
| **Stateless**   | Service layers, filters          | State required for business logic |
| **Thread Confinement** | Thread-local tasks (e.g., async processing) | Shared state access needed |
| **Volatile**    | Simple flags (e.g., stop signal) | Complex invariants (double-checked locking) |
| **Atomic**      | Single variable updates (counters, flags) | Compound actions (e.g., check-then-act) |

**Real-World Example**:  
```java
class RateLimiter {
    private final AtomicLong allowedRequests = new AtomicLong(100);

    boolean allowRequest() {
        long current = allowedRequests.get();
        return current > 0 && allowedRequests.compareAndSet(current, current - 1);
    }
}
```

**Performance**: `AtomicLong` uses CAS → fast with low contention.

**Common Mistake**: Assuming `volatile` solves all visibility issues → doesn’t prevent reordering for complex ops.

**Interview Tip**: *"Why not use `synchronized` everywhere?"* → Overhead kills scalability under contention.

---

## 5. Executors Framework  

### Core Classes  

| Class               | When to Use                          | Pitfall                           |
|---------------------|--------------------------------------|-----------------------------------|
| `ExecutorService`   | Generic task execution              | Shutting down improperly → task loss |
| `ThreadPoolExecutor`| Custom pool sizing, queue control  | ` bounded` queue → rejection policy |
| `ScheduledExecutorService`| Delayed/periodic tasks         | Scheduled tasks lost on shutdown |
| `Future<T>`         | Retrieve result, cancel tasks       | `get()` blocks → deadlocks if pool exhausted |

**Tune Thread Pools**:  
```java
int corePoolSize = Runtime.getRuntime().availableProcessors(); 
ThreadPoolExecutor exec = new ThreadPoolExecutor(
    corePoolSize, 
    corePoolSize * 2, 
    60L, TimeUnit.SECONDS, 
    new LinkedBlockingQueue<>(1000)
);
```

**Tradeoffs**:  
| Fixed Pool |Cached Pool|
|------------|------------|
| Predictable resource usage | Potentially unlimited threads → OOM |

**Production Insight**: Use `ThreadPoolExecutor.CallerRunsPolicy` in web apps to protect against malicious requests.

**Key Takeaway**: **Always shutdown executors gracefully in `try-with-resources` or `finally`.**

---

## 6. Concurrent Collections  

| Class                  | Best For                           | Thread Safety Mechanism          |
|------------------------|------------------------------------|----------------------------------|
| `ConcurrentHashMap`    | High-read, moderate-write maps    | Segment-level locking (since JDK 8: lock striping) |
| `CopyOnWriteArrayList`| Read-heavy, infrequent writes    | Iteration snapshot (no concurrent mods) |
| `BlockingQueue`        | Producer-consumer pipelines        | `put()`/`take()` block when full/empty |
| `ConcurrentLinkedQueue`| Unbounded FIFO, weak consistency  | Lock-free (CAS)                  |

**Performance Bottleneck**: `ConcurrentHashMap` resizing → locks all segments temporarily.

**Real-World Example**: Cache with expiration using `ConcurrentHashMap` + `ScheduledExecutorService`.

**Interview Tip**: *"Why `CopyOnWriteArrayList` for broadcast events?"* → Safe iteration, no lost updates.

**Common Mistake**: Assuming `ConcurrentHashMap` allows concurrent iteration + modification without locks.

---

## 7. Locks  

### ReentrantLock vs `synchronized`  
| Feature          | `synchronized`     | `ReentrantLock`                 |
|------------------|---------------------|----------------------------------|
| **Reentrancy**   | Yes                 | Yes                              |
| **Fairness**     | No (daemonic)       | Configurable                     |
| **TryLock**      | No                  | Yes (`lock.tryLock()`)          |
| **ConditionVars**| Limited             | Full support                     |

### ReadWriteLock  
| Scenario         | Implementation                 |
|------------------|---------------------------------|
| **Read-heavy**   | `readLock.lock(); ... readUnlock();` |
| **Write-heavy**  | `writeLock.lock(); ... writeUnlock();` |

**StampedLock** (JDK 8+)  
- **Optimistic Reads**: `validate(stamp)` avoids locks for reads.  
- **Use Case**: UI rendering, caching layers.

**Tradeoffs**:  
| `ReentrantLock` | `StampedLock` |
|-----------------|---------------|
| Structured locking | Low-latency optimistic reads |

**Production Insight**: Prefer `ReentrantLock` over `synchronized` when needing conditions or fairness.

---

## 8. ThreadLocal  

### Use Cases  
| Scenario               | Example                              |
|------------------------|--------------------------------------|
| **MDC Logging**        | `ThreadLocal<Logger>`               |
| **User Session**       | Request-scoped user context          |
| **Resource Pool**      | Assign one resource per thread      |

**Memory Leak Risks**:  
- **WeakReference** not used by default → GC won't collect if thread lives long.  
- **Solution**: `ThreadLocal.withInitial()`, explicit `remove()` in `finally`.

**Real-World Pitfall**: Web app threads reused → leaked user data across requests.

**Interview Tip**: *"How to avoid `ThreadLocal` leaks in thread pools?"* → `ThreadPoolExecutor` with `ThreadFactory` that clears `ThreadLocal`.

**Key Takeaway**: **Use `ThreadLocal` sparingly; prefer MDC via frameworks.**

---

## 📊 Comparison Tables  

### Locking Mechanisms  
| Type                | Granularity       | Reentrancy | Fairness | Use Case               |
|---------------------|-------------------|------------|----------|------------------------|
| **Intrinsic**       | Object monitor    | Yes        | No       | Simple sync            |
| **ReentrantLock**   | Explicit object   | Yes        | Config   | Complex sync           |
| **ReadWriteLock**   | Read/write scopes | No         | No       | Read-mostly            |
| **StampedLock**     | Optimistic reads  | Yes        | No       | Low-latency reads      |

### Concurrency Tools  
| Tool                | Throughput | Latency | Complexity |
|---------------------|------------|---------|-------------|
| **BlockingQueue**   | High       | Low     | Medium      |
| **CompletableFuture**| High      | Low     | High        |
| **ForkJoinPool**    | High       | Medium  | Medium      |
| **ExecutorService** | Medium     | Low     | Low         |

---

## 🔥 Production Troubleshooting  

**Thread Dump Analysis**  
```bash
kill -3 <pid> # Generates thread dump to stdout/log
```
- **Look For**: `BLOCKED` threads holding locks, long `WAITING` periods.  
- **Tool**: `jstack`, `jcmd <pid> Thread.print`.

**Common Issues**:  
| Symptom                | Root Cause                     | Fix                          |
|------------------------|---------------------------------|------------------------------|
| High CPU, low throughput| Lock contention                | Reduce lock scope, try `LockStriping` |
| OOM Old Gen            | Thread-local leaks             | Clear `ThreadLocal` in pools |
| Deadlocks              | Circular lock acquisition      | Consistent lock order        |
| Frequent GC pauses     | Excessive object creation in threads | Pool sizing, object reuse |

---

## 🎯 Interview Preparation Checklist  

1. **Explain CAP implications** for concurrent `HashMap` vs `ConcurrentHashMap`.  
2. **Discuss tradeoffs** of `synchronized` vs `ReentrantLock`.  
3. **Solve**: *"How to make singleton thread-safe with lazy initialization?"* → Double-checked locking or `static final`.  
4. **Debug**: Given a thread dump, identify bottlenecks.  
5. **Design**: Rate limiter using `AtomicLong` and `ScheduledExecutorService`.

---

## 📌 Key Takeaways  

✅ **Never block thread-pool threads on I/O** – offload to async or separate pools.  
✅ **Prefer `ConcurrentHashMap` over `synchronized` HashMap** for scalability.  
✅ **Use time bounds** on locks (`tryLock(timeout)`) to avoid deadlocks.  
✅ **Profile contention** with `jstack` or async-profiler before optimizing.  
✅ **Shutdown executors** – never rely on JVM to close threads.  

*"In production, a well-tuned thread pool is worth more than a hundred clever algorithms."*  