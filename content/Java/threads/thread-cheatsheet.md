- [[#📚 Quick Concepts|📚 Quick Concepts]]
- [[#🔍 Quick Comparisons|🔍 Quick Comparisons]]
- [[#⚠️ Common Problems|⚠️ Common Problems]]
- [[#🛠️ Best Practices|🛠️ Best Practices]]
- [[#📊 Production Tips|📊 Production Tips]]
- [[#🎯 Key Takeaways|🎯 Key Takeaways]]

## 📚 Quick Concepts  

### Thread Lifecycle  
| State         | Description                | When Occurs               |
|---------------|---------------------------|---------------------------|
| **New**       | Thread object created     | `new Thread()`            |
| **Runnable**  | Ready to run              | After `start()`           |
| **Blocked**   | Waiting for I/O/lock      | `wait()`, I/O operations  |
| **Waiting**   | Explicit wait             | `wait()`                  |
| **Timed Waiting** | Waiting with timeout  | `sleep()`, `join(timeout)`|
| **Terminated**| Execution completed       | Exit from `run()`         |

**Interview Tip**: *"BLOCKED threads in dumps indicate lock contention."*  
**Common Mistake**: *Assuming `Runnable` = actively running.*  
**Production Insight**: *Monitor thread counts; excessive `New` states signal leaks.*  
**Tradeoff**: `*start()` vs `submit()`: direct control vs pool management.*  
**Key Takeaway**: *Use `interrupt()` for cancellation, never `stop()`.**  

### Locking Hierarchy  
```mermaid
graph LR
    A[UserLock] --> B[AccountLock]
    B --> C[TransactionLock]
```
**Order**: User → Account → Transaction  

**Interview Tip**: *"Always acquire locks in consistent global order."*  
**Common Mistake**: *Reverse order in different methods → deadlock.*  
**Production Insight**: *Use `tryLock(timeout)` to avoid deadlocks.*  
**Tradeoff**: *Hierarchical locking reduces flexibility but prevents deadlocks.*  
**Key Takeaway**: *Document lock order; enforce via code reviews.*  

### Memory Visibility  
| Keyword/Type   | Visibility Guarantee      | Atomicity          |
|----------------|---------------------------|--------------------|
| `volatile`     | Flushes changes to main mem | No                 |
| `Atomic*`      | CAS-based, visible        | Yes (single var)   |
| `synchronized`| Happens-before barrier    | Yes (block scope)  |
| `final`        | Safe publication          | N/A                |

**Interview Tip**: *"`volatile` ensures visibility but not atomicity for compound actions."*  
**Common Mistake**: *"Using `volatile` for counters without atomic ops."*  
**Production Insight**: *`AtomicLong` preferred for high-contention counters.*  
**Tradeoff**: *`volatile` is lighter; `Atomic` has CAS overhead.*  
**Key Takeaway**: *Use `volatile` for flags; `Atomic` for counters.*  

### Thread Pool Types  
| Pool Type               | Use Case                          | Core Size        | Task Queue        |
|-------------------------|-----------------------------------|------------------|-------------------|
| `FixedThreadPool`       | CPU-bound tasks                   | Fixed N          | `LinkedBlockingQueue` |
| `CachedThreadPool`      | Short-lived tasks                 | 0                 | `SynchronousQueue` |
| `ScheduledExecutorService`| Delayed/periodic tasks          | Core + keepalive | `DelayedQueue`    |
| `ForkJoinPool`          | Recursive divide-and-conquer     | `commonPool()`   | Work-stealing     |

**Interview Tip**: *"`ForkJoinPool` excels at recursive tasks."*  
**Common Mistake**: *"Using cached pool for long-running tasks → thread exhaust."*  
**Production Insight**: *Tune `corePoolSize` = `#cores` for CPU-bound work.*  
**Tradeoff**: *Fixed pools limit scalability; cached pools risk OOM.*  
**Key Takeaway**: *Prefer `ExecutorService` over raw threads.*  

### Concurrent Collections  
| Collection              | Thread Safety                      | Use Case                          |
|-------------------------|------------------------------------|-----------------------------------|
| `ConcurrentHashMap`     | Lock striping, concurrent reads    | High-read caches                  |
| `CopyOnWriteArrayList`  | Safe iteration, slow writes        | Read-mostly lists (e.g., logs)   |
| `BlockingQueue`         | Producer-consumer handshake        | Task queues, rate limiting       |
| `ConcurrentLinkedQueue` | Lock-free FIFO                     | Unbounded queues                  |

**Interview Tip**: *"`CopyOnWriteArrayList` avoids iteration failures but has write overhead."*  
**Common Mistake**: *"Using `CHM` for high write contention → use `StampedLock`."*  
**Production Insight**: *`BlockingQueue` with rejection handler prevents overload.*  
**Tradeoff**: *Lock striping vs full sync performance.*  
**Key Takeaway**: *Choose based on read/write ratio.*  

---

## 🔍 Quick Comparisons  

### `synchronized` vs `ReentrantLock`  
| Feature            | `synchronized`                | `ReentrantLock`               |
|--------------------|-------------------------------|-------------------------------|
| Reentrancy         | Yes                           | Yes                           |
| Fairness           | No                            | Configurable                  |
| Try-lock           | No                            | Yes (`tryLock()`)             |
| Condition Support  | Limited (`wait/notify`)       | Full (`Condition`)            |
| Performance        | Lower overhead                | Higher control, tuning       |

**Interview Tip**: *"Use `ReentrantLock` for fairness or complex locking."*  
**Common Mistake**: *"Mixing `synchronized` and `ReentrantLock` on same object."*  
**Tradeoff**: *Flexibility vs complexity.*  
**Key Takeaway**: *`synchronized` for simplicity; `ReentrantLock` for control.*  

### `HashMap` vs `ConcurrentHashMap`  
| Feature            | `HashMap`                     | `ConcurrentHashMap`           |
|--------------------|-------------------------------|-------------------------------|
| Thread Safety      | Not thread-safe               | Thread-safe                   |
| Concurrency        | Single-threaded               | High concurrency              |
| Locking            | Coarse-grained                | Lock striping                 |
| Iteration          | Fail-fast                     | Weakly consistent             |
| Use Case           | Single-threaded               | Multi-threaded read-heavy     |

**Interview Tip**: *"`CHM` allows concurrent reads without locking."*  
**Common Mistake**: *"Assuming `CHM` solves all concurrency issues."*  
**Tradeoff**: *Striping vs full sync performance.*  
**Key Takeaway**: *Use `CHM` for shared caches; `HashMap` for single-threaded.*  

### `Runnable` vs `Callable`  
| Feature            | `Runnable`                    | `Callable`                     |
|--------------------|-------------------------------|-------------------------------|
| Return Value       | `void`                        | Generic `V`                   |
| Exception Handling | None                          | Propagates via `Future`       |
| Use Case           | Simple tasks                  | Tasks needing result/exceptions |

**Interview Tip**: *"Use `Callable` when tasks return values or throw checked exceptions."*  
**Common Mistake**: *"Ignoring `Future.get()` timeouts → thread deadlock."*  
**Key Takeaway**: *Prefer `Callable` for async results.*  

### `Future` vs `CompletableFuture`  
| Feature            | `Future`                      | `CompletableFuture`           |
|--------------------|-------------------------------|-------------------------------|
| Synchronization    | Blocking `get()`              | Async chaining (`thenApply`)  |
| Error Handling     | Manual inspection             | Built-in `.exceptionally()`   |
| Use Case           | Simple async tasks            | Pipelines, composed actions  |

**Interview Tip**: *"`CompletableFuture` avoids callback hell via fluent API."*  
**Common Mistake**: *"Deep chaining → stack overflow."*  
**Key Takeaway**: *Use for async pipelines; handle errors with `exceptionally()`.*  

### `wait()` vs `sleep()`  
| Feature            | `wait()`                      | `sleep()`                      |
|--------------------|-------------------------------|-------------------------------|
| Lock Release       | Releases monitor lock         | Does **not** release lock    |
| Use Case           | Coordination in sync blocks  | Simple delays                 |
| Interruptible?      | Yes (`InterruptedException`)  | Yes                           |

**Interview Tip**: *"Never call `wait()` outside synchronized block."*  
**Common Mistake**: *Forgetting to re-acquire lock after `wait()`.*  
**Key Takeaway**: *Use `wait()` for coordination; `sleep()` for delays.*  

### `notify()` vs `notifyAll()`  
| Feature            | `notify()`                    | `notifyAll()`                 |
|--------------------|-------------------------------|-------------------------------|
| Wake-ups           | One arbitrary waiting thread | All waiting threads          |
| Use Case           | Optimize when safe           | General purpose              |

**Interview Tip**: *"`notifyAll()` safer but may waste CPU."*  
**Common Mistake**: *"Using `notify()` and assuming all threads wake."*  
**Key Takeaway**: *Prefer `notifyAll()` unless optimization is proven.*  

### `Atomic` vs `volatile`  
| Feature            | `volatile`                    | `Atomic`                      |
|--------------------|-------------------------------|-------------------------------|
| Visibility         | Yes                           | Yes                           |
| Atomicity          | No                            | Yes (single var operations)   |
| Use Case           | Simple flags                  | Counters, flags with CAS      |

**Interview Tip**: *"`volatile` ≠ atomic; use `Atomic` for counters."*  
**Common Mistake**: *"Using `volatile` for `incrementAndGet()`."*  
**Key Takeaway**: *`Atomic` for concurrency-safe primitives.*  

---

## ⚠️ Common Problems  

| Problem          | Cause                          | Fix                            | Production Symptom            |
|------------------|--------------------------------|--------------------------------|-------------------------------|
| **Deadlock**     | Circular lock wait             | Consistent lock order          | Threads stuck `BLOCKED`       |
| **Starvation**   | High-priority threads hogging  | Fair locks, priority inheritance| Low-priority threads idle     |
| **Race Condition**| Unsynced shared state updates | Use `synchronized`, `Atomic`  | Inconsistent data             |
| **Memory Leak**  | `ThreadLocal` not cleared     | `remove()` in `finally`       | `OutOfMemoryError`           |
| **Thread Exhaustion**| Unbounded pool creation    | Bounded queues, reject policies| `RejectedExecutionException` |

**Interview Tip**: *"Deadlock detection via `ThreadMXBean.findDeadlockedThreads()`."*  
**Common Mistake**: *"Assuming starvation only affects low-priority threads."*  
**Key Takeaway**: *Monitor thread states; use timeouts to avoid deadlocks.*  

---

## 🛠️ Best Practices  

1. **Avoid Shared Mutable State**  
   - Prefer immutables (`record`), stateless services.  
2. **Prefer Executors Over Raw Threads**  
   - Pools manage resources; avoid thread leaks.  
3. **Tune Thread Pools**  
   - CPU-bound: `coreSize = #cores`; IO-bound: `coreSize = 2*x` + keepalive.  
4. **Minimize Lock Scope**  
   - Lock only critical sections; avoid class-level sync.  
5. **Use `Concurrent` Collections**  
   - `ConcurrentHashMap` > synchronized Map for reads.  

**Interview Tip**: *"Explain why `CopyOnWriteArrayList` is slow for writes."*  
**Common Mistake**: *"Using `synchronized` on public methods for entire class."*  

---

## 📊 Production Tips  

### Thread Dump Analysis  
```bash
jcmd <pid> Thread.print > dump.txt
```
- **Key Signals**:  
  - `BLOCKED`: Waiting for lock → check lock owner.  
  - `WAITING`: `wait()` called → check timeout.  
  - `TIMED_WAITING`: `sleep()`/`join()` → adjust timeouts.  

**Interview Tip**: *"Correlate dumps with GC logs to find pauses."*  
**Production Insight**: *Automate dump collection on CPU spikes.*  

### Pool Sizing Formula  
```text
CPU-bound: corePoolSize = Runtime.getRuntime().availableProcessors()
IO-bound: corePoolSize = (Thread.maxPriority() - Thread.MIN_PRIORITY) * 2
```

### CPU-bound vs IO-bound Threads  
| Type             | Characteristics               | Pool Choice                   |
|------------------|-------------------------------|-------------------------------|
| **CPU-bound**    | High CPU, low I/O             | Fixed pool, size = cores     |
| **IO-bound**     | Blocking I/O, waiting         | Cached pool, keepalive = 60s |

**Interview Tip**: *"Explain why `ForkJoinPool` suits CPU-bound recursive tasks."*  

### Async API Design  
- **Use `CompletionStage<T>`** for reactive flows.  
- **Avoid blocking calls** in async contexts → thread starvation.  

### Monitoring  
- **Metrics**: `Thread.activeCount()`, `LockSupport.parkCount`.  
- **Tools**: `jvisualvm`, `async-profiler`, Prometheus exporters.  

**Key Takeaway**: *"Profile before tuning; assume 80% of gains come from 20% of bottlenecks."*  

---

## 🎯 Key Takeaways  

✅ **Lock Order > Lock Type** – Consistent hierarchy prevents deadlocks.  
✅ **`Atomic` > `volatile`** for counters; `volatile` only for simple flags.  
✅ **Pool Sizing** – CPU-bound = cores; IO-bound = larger + keepalive.  
✅ **Thread Dumps** – First step in diagnosing production deadlocks.  
✅ **Prefer Immutables** – Reduce bugs, simplify concurrency.  
