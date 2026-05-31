- [[#📚 Core Topics Overview|📚 Core Topics Overview]]
- [[#1. JVM & Concurrency|1. JVM & Concurrency]]
- [[#2. Advanced Synchronization|2. Advanced Synchronization]]
- [[#3. ForkJoin Framework|3. ForkJoin Framework]]
- [[#4. CompletableFuture|4. CompletableFuture]]
- [[#5. Reactive & Async Concepts|5. Reactive & Async Concepts]]
- [[#6. Deadlocks|6. Deadlocks]]
- [[#7. Performance Optimization|7. Performance Optimization]]
- [[#8. Parallel Processing|8. Parallel Processing]]
- [[#9. Real-world Patterns|9. Real-world Patterns]]
- [[#10. Production Debugging|10. Production Debugging]]
- [[#🎯 Interview Preparation Checklist|🎯 Interview Preparation Checklist]]
- [[#📌 Key Takeaways|📌 Key Takeaways]]


---

## 📚 Core Topics Overview  

| Category                  | Key Focus                          |
|---------------------------|------------------------------------|
| **JVM & Concurrency**     | JMM, visibility, reordering        |
| **Advanced Sync**         | CAS, lock-free, ABA, spin locks   |
| **ForkJoin**              | Work stealing, recursive tasks    |
| **CompletableFuture**     | Async pipelines, chaining         |
| **Reactive/Async**        | Non-blocking, backpressure        |
| **Deadlocks**             | Detection, prevention             |
| **Performance**           | Contention, false sharing         |
| **Parallel Processing**   | Streams, batching                 |
| **Patterns**              | Bulkhead, circuit breaker         |
| **Debugging**             | Thread dumps, CPU spikes          |

---

## 1. JVM & Concurrency  

### Java Memory Model (JMM)  
```mermaid
graph LR
    A[JVM Heap] --> B[Main Memory]
    B --> C[CPU Cache 1]
    B --> D[CPU Cache 2]
````

|Concept|What It Is|Why It Matters|
|---|---|---|
|**Happens-Before**|Order guarantees (e.g., `x = y;` → `y` visible)|Prevents race conditions|
|**Visibility**|Changes to variables seen by threads|`volatile` ensures visibility|
|**Reordering**|Compiler/CPU may reorder instructions|Can break invariants (e.g., check-then-act)|
|**CPU Cache**|Local cache per core|False sharing, cache coherency|

**Interview Tip**: _"Why does `volatile` solve visibility but not reordering?"_ → `volatile` adds acquire/release semantics but not full happens-before.

**Common Mistake**: Assuming `final` fields guarantee visibility without `volatile`.

**Production Insight**: Use `Atomic*` classes for simple state changes; avoid `synchronized` for high-frequency flags.

**Key Takeaway**: **JMM ensures _minimum_ visibility; use `volatile`/`Atomic` for simple fields.**

---

## 2. Advanced Synchronization

### CAS & Lock-Free Programming

```java
class Counter {
    private final AtomicInteger count = new AtomicInteger(0);
    void increment() {
        while (true) {
            int current = count.get();
            int next = current + 1;
            if (count.compareAndSet(current, next)) return;
        }
    }
}
```

|Technique|Use Case|Pitfall|
|---|---|---|
|**CAS**|Lock-free counters, algorithms|**ABA Problem** (see below)|
|**Lock-Free**|No blocking, high throughput|Complex error handling|

### ABA Problem

|Scenario|Example|
|---|---|
|Thread A: A → B → A|Thread B reads A during transition|
|Result|CAS passes incorrectly|

**Fix**: Use `AtomicReferenceAndInt` or versioned objects.

### Spin Locks

```java
class SpinLock {
    private final AtomicBoolean locked = new AtomicBoolean(false);
    void lock() {
        while (locked.getAndSet(true)) Thread.onSpinWait();
    }
    void unlock() { locked.set(false); }
}
```

**When to Use**: Short critical sections on multi-core systems.

**Tradeoffs**:

|Spin Locks|Mutex Locks|
|---|---|
|✅ Low latency|✅ Prevents CPU starvation|
|❌ Wastes CPU|❌ Longer latency on contention|

---

## 3. ForkJoin Framework

### Work Stealing

```mermaid
graph LR
    A[Main Thread] --> B[Task Pool]
    B --> C[Fork]
    C --> D[Split Work]
    D --> E[Steal from Other Threads]
```

|Class|Purpose|
|---|---|
|`RecursiveTask`|Returns result (e.g., sum)|
|`RecursiveAction`|No result (e.g., update state)|

**Real-World Example**: Parallel file processing

```java
ForkJoinPool.commonPool().invoke(new RecursiveAction() {
    protected void compute() {
        if (files.size() <= 10) process(files);
        else splitAndFork();
    }
});
```

**Interview Tip**: _"When to prefer ForkJoin over `CompletableFuture`?"_ → ForkJoin excels at **divide-and-conquer** tasks.

**Common Mistake**: Unbounded recursion → `StackOverflowError`.

**Production Insight**: Use `commonPool()` for short-lived tasks; tune `parallelismLevel` for long-running jobs.

---

## 4. CompletableFuture

### Async Pipelines

```java
CompletableFuture.supplyAsync(() -> fetchUser())
    .thenApply(user -> fetchOrders(user))
    .thenAccept(orders -> logOrders(orders))
    .exceptionally(ex -> handleError(ex));
```

|Feature|Benefit|
|---|---|
|**Chaining**|Linear async flow|
|**Exception Handling**|`.exceptionally()` isolates failures|
|**Parallel**|`CompletableFuture.allOf()`|

**Performance**: Avoid excessive `.thenApply()` depth → use `join()` sparingly.

**Tradeoffs**:

|CompletableFuture|Future + Callback|
|---|---|
|✅ Composition|✅ Simple error handling|

---

## 5. Reactive & Async Concepts

### Backpressure

|System|Mechanism|
|---|---|
|**Project Reactor**|`flux.onBackpressureBuffer()`|
|**RxJava**|`flowable.onBackpressureDrop()`|

**When to Use**: High-throughput streams (e.g., IoT data).

**Interview Tip**: _"How does Reactor differ from CompletableFuture?"_ → Reactor = **streams**, CompletableFuture = **single values**.

---

## 6. Deadlocks

### Four Conditions

|Condition|Example|
|---|---|
|**Mutual Exclusion**|Two threads hold Lock A/B|
|**Hold and Wait**|Thread 1 holds A, waits for B|
|**No Preemption**|Locks can’t be forcibly taken|
|**Circular Wait**|A → B → A chain|

**Detection**:

```java
ThreadMXBean bean = ManagementFactory.getThreadMXBean();
long[] deadlocked = bean.findDeadlockedThreads(); 
```

**Prevention**:

- **Lock ordering**
- **TryLock with timeout**
- **Lock timeouts**

**Production Insight**: Use `Thread.dump()` during outages to identify deadlock patterns.

---

## 7. Performance Optimization

### False Sharing

```java
// Bad: Different fields on same cache line
class Throughput {
    volatile long counter1 = 0;
    volatile long counter2 = 0; // Same cache line!
}
```

**Fix**: Add padding (`long pad1-7`) between fields.

### Lock Granularity

|Approach|Throughput Impact|
|---|---|
|**Coarse**|Low contention, high latency|
|**Fine**|High contention, low latency|

**Rule of Thumb**: Start coarse; split only under load.

---

## 8. Parallel Processing

### Parallel Streams

```java
list.parallelStream()
    .map(Order::getPrice)
    .reduce(0, Integer::sum);
```

**When to Use**:

- **Embarrassingly parallel** tasks
- **Large collections** (>10k elements)

**Caution**: Avoid stateful intermediates (e.g., `BufferedWriter`).

---

## 9. Real-world Patterns

### Bulkhead Pattern

```java
ExecutorService bulkhead = new ThreadPoolExecutor(
    2, 2, 0, TimeUnit.MILLISECONDS, new LinkedBlockingQueue<>(10)
);
```

**Purpose**: Isolate failures in critical subsystems.

### Circuit Breaker

```java
CircuitBreaker cb = new CircuitBreaker(5, 1, TimeUnit.MINUTES);
if (cb.allow()) try {
    // Call external service
} catch (Exception e) {
    cb.fail();
}
```

---

## 10. Production Debugging

### Thread Dump Analysis

```bash
jstack <pid> > threads.dump
```

**Key Signals**:

- **BLOCKED**: Waiting for lock
- **WAITING**: `Object.wait()`
- **TIMED_WAITING**: `Thread.sleep()`

### CPU Spikes

|Symptom|Debug Steps|
|---|---|
|100% CPU, low throughput|1. Run `jstack` 2. Check for lock contention 3. Enable GC logs|

**Livelock Example**: Two threads keep yielding to each other → high CPU, no progress.

---

## 🎯 Interview Preparation Checklist

1. **Explain JMM** with happens-before examples.
2. **Solve ABA problem** with `AtomicReferenceAndInt`.
3. **Design a deadlock-safe** resource pool.
4. **Compare ForkJoin vs CompletableFuture** for file processing.
5. **Debug a thread dump** to identify blocked threads.

---

## 📌 Key Takeaways

✅ **CAS is powerful but fragile** – handle ABA with version stamps.  
✅ **Fine-grained locks > coarse locks** for scalability.  
✅ **Use ForkJoin for recursive tasks**, CompletableFuture for pipelines.  
✅ **Profile before optimizing** – false sharing is subtle.  
✅ **Always add timeouts** to locks in distributed systems.

_"In production, a well-designed backpressure strategy saves more systems than any algorithm."_