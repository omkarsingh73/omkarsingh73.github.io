
> 🔧 **Covers:** JVM Architecture · Heap/Stack · GC · Memory Leaks · OOM Errors · JVM Tuning

---

# 📋 Table of Contents

- [[#1. JVM Architecture Overview|1. JVM Architecture Overview]]
- [[#2. Java Memory Areas|2. Java Memory Areas]]
- [[#3. Stack vs Heap|3. Stack vs Heap]]
- [[#4. PermGen vs Metaspace|4. PermGen vs Metaspace]]
- [[#5. Young Generation Memory|5. Young Generation Memory]]
- [[#6. Garbage Collection (GC)|6. Garbage Collection (GC)]]
- [[#7. Object Lifecycle in Java|7. Object Lifecycle in Java]]
- [[#8. Strong, Weak, Soft & Phantom References|8. Strong, Weak, Soft & Phantom References]]
- [[#9. Memory Leaks in Java|9. Memory Leaks in Java]]
- [[#10. OutOfMemoryError Types|10. OutOfMemoryError Types]]
- [[#11. StackOverflowError|11. StackOverflowError]]
- [[#12. JVM Tuning Basics|12. JVM Tuning Basics]]
- [[#13. JVM Memory Diagram|13. JVM Memory Diagram]]
- [[#14. Important Interview Questions|14. Important Interview Questions]]
- [[#15. Quick Revision Cheatsheet|15. Quick Revision Cheatsheet]]


---

## 1. JVM Architecture Overview

> **One-liner:** JVM is an abstract machine that executes Java bytecode and manages memory at runtime.

```
┌─────────────────────────────────────────────────────────────┐
│                        JVM Architecture                     │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   ┌──────────────────────────────────────────────────┐      │
│   │              CLASS LOADER SUBSYSTEM              │      │
│   │   Bootstrap → Extension → Application Loader     │      │
│   └──────────────────────────┬───────────────────────┘      │
│                               ▼                             │
│   ┌──────────────────────────────────────────────────┐      │
│   │             RUNTIME DATA AREAS                   │      │
│   │  ┌──────┐ ┌──────┐ ┌────────┐ ┌────┐ ┌───────┐   │      │
│   │  │ Heap │ │Stack │ │Method  │ │ PC │ │Native │   │      │
│   │  │      │ │      │ │ Area   │ │Reg.│ │Stack  │   │      │
│   │  └──────┘ └──────┘ └────────┘ └────┘ └───────┘   │      │
│   └──────────────────────────┬───────────────────────┘      │
│                               ▼                             │
│   ┌──────────────────────────────────────────────────┐      │
│   │              EXECUTION ENGINE                    │      │
│   │   Interpreter → JIT Compiler → GC                │      │
│   └──────────────────────────────────────────────────┘      │
└─────────────────────────────────────────────────────────────┘
```

### JVM Components Summary

| Component | Role |
|-----------|------|
| **Class Loader** | Loads `.class` files into JVM (Bootstrap → Extension → App) |
| **Heap** | Stores all objects and arrays |
| **Stack** | Stores method frames, local variables per thread |
| **Method Area** | Stores class metadata, static variables, constant pool |
| **PC Register** | Points to current executing instruction per thread |
| **Native Method Stack** | Manages native (C/C++) method calls |
| **Execution Engine** | Executes bytecode (Interpreter + JIT) |
| **JIT Compiler** | Compiles hot bytecode to native machine code at runtime |
| **GC** | Automatically reclaims memory from unreachable objects |

### Class Loading Phases

```
Loading → Linking (Verify → Prepare → Resolve) → Initialization
```

> 💡 **Interview Tip:** JVM is platform-independent; the JRE (JVM + libraries) is platform-dependent.

---

## 2. Java Memory Areas

### 2.1 Heap Memory

- Stores all **objects** and **arrays**
- Shared across all threads
- Managed by **Garbage Collector**
- Divided into **Young Gen** + **Old Gen** (+ Metaspace in Java 8+)

```
HEAP
┌─────────────────────────────────────────────────────┐
│  Young Generation                │  Old Generation  │
│  ┌──────────┬───────┬───────┐    │  (Tenured Space) │
│  │  Eden    │  S0   │  S1   │    │                  │
│  │  Space   │       │       │    │  Long-lived      │
│  │ (new obj)│Surv.0 │Surv.1 │    │  objects         │
│  └──────────┴───────┴───────┘    │                  │
└─────────────────────────────────────────────────────┘
```

---

### 2.2 Stack Memory

- One stack **per thread** — thread-safe by design
- Stores **stack frames** (one per method call)
- Each frame contains: local variables, operand stack, frame data
- LIFO order — method returns → frame popped
- Fixed size (default ~512KB–1MB per thread)

```
STACK (Thread-1)
┌─────────────────────┐ ← top
│  main() frame       │   - args
│  ─────────────────  │
│  calculate() frame  │   - local vars (a, b, result)
│  ─────────────────  │
│  helper() frame     │   - local vars
└─────────────────────┘ ← bottom
```

---

### 2.3 Method Area (Metaspace in Java 8+)

- Stores **class-level** data: bytecode, field names, method signatures
- Stores **static variables** and the **Runtime Constant Pool**
- Shared across all threads
- In Java 8+: lives in **native memory** as Metaspace (no fixed size by default)

---

### 2.4 PC Register (Program Counter)

- One per thread
- Holds address of **currently executing JVM instruction**
- `undefined` for native methods

---

### 2.5 Native Method Stack

- Supports native methods written in C/C++
- One per thread
- Used when `native` keyword methods are invoked

---

## 3. Stack vs Heap

### Code Example

```java
class Test {
    int x = 10;  // instance variable → stored in HEAP (inside object)

    public static void main(String[] args) {
        int a = 5;       // local variable → STACK
        Test t = new Test();  // reference t → STACK, object → HEAP
    }
}
```

```
STACK (main thread)          HEAP
┌──────────────────┐        ┌──────────────────────┐
│  main() frame    │        │  Test object         │
│  ┌────────────┐  │        │  ┌────────────────┐  │
│  │ a = 5      │  │        │  │ x = 10         │  │
│  │ t ─────────┼──┼───────►│  └────────────────┘  │
│  └────────────┘  │        └──────────────────────┘
└──────────────────┘
```

### Stack vs Heap Comparison Table

| Feature | Stack | Heap |
|---------|-------|------|
| **What's stored** | Local vars, method calls, references | Objects, arrays, instance vars |
| **Scope** | Method/block scope | Until GC collects |
| **Thread safety** | ✅ Thread-safe (private per thread) | ❌ Shared — needs synchronization |
| **Lifetime** | Auto-freed on method return | Managed by GC |
| **Size** | Small (~512KB–1MB default) | Large (configured by `-Xmx`) |
| **Allocation speed** | ⚡ Fast (pointer move) | 🐢 Slower (GC overhead) |
| **Error** | `StackOverflowError` | `OutOfMemoryError` |
| **Order** | LIFO | No order |
| **Access** | Very fast | Slower |

> 💡 **Interview Tip:** Local variables are **always** on the stack. Primitive local variables never leave the stack. Only **references** to objects are on the stack — the actual objects live on the heap.

---

## 4. PermGen vs Metaspace

### Before Java 8 — PermGen (Permanent Generation)

- Part of the **heap**
- Fixed maximum size (default 64MB–256MB)
- Stored: class metadata, static variables, interned strings, method bytecode
- Common error: `java.lang.OutOfMemoryError: PermGen space`
- Caused by: deploying too many classes, hot-reloading in app servers

```bash
# PermGen JVM flags (Java 7 and earlier)
-XX:PermSize=64m          # initial PermGen size
-XX:MaxPermSize=256m      # maximum PermGen size
```

---

### After Java 8 — Metaspace

- **Removed from heap** — lives in **native OS memory**
- **No fixed upper limit** by default (grows as needed)
- Only stores **class metadata** (interned strings moved to heap)
- Rarely causes OOM — but can still grow unbounded if unchecked

```bash
# Metaspace JVM flags (Java 8+)
-XX:MetaspaceSize=64m         # initial Metaspace size
-XX:MaxMetaspaceSize=256m     # cap it (recommended in production)
```

### PermGen vs Metaspace Table

| Feature | PermGen | Metaspace |
|---------|---------|-----------|
| **Java version** | Java 7 and earlier | Java 8+ |
| **Location** | Heap (fixed part) | Native OS memory |
| **Default max size** | 64MB–256MB (fixed) | Unlimited (grows dynamically) |
| **OOM error** | `PermGen space` | `Metaspace` (if capped) |
| **Interned Strings** | Stored here | Moved to heap (Java 8+) |
| **Resizing** | ❌ Needs restart | ✅ Dynamic |
| **GC tuning needed** | Yes (common pain point) | Less frequent |
| **Monitoring** | `-verbose:gc` | `jstat`, `jcmd` |

### Why PermGen Was Removed

- Fixed size caused frequent `OutOfMemoryError` in enterprise apps
- Difficult to predict how much class metadata would be needed
- Hot deployment (e.g. Tomcat, JBoss) kept reloading classes, filling PermGen
- Native memory is larger and more flexible

> ⚠️ **Warning:** Even with Metaspace, class loader leaks (e.g., repeated hot deployments) can cause `OutOfMemoryError: Metaspace`. Always set `-XX:MaxMetaspaceSize` in production.

---

## 5. Young Generation Memory

> Young Gen is where **all new objects are born**. Most objects die young (short-lived).

### Structure

```
Young Generation
┌─────────────────────────────────────────────────────┐
│                                                     │
│  ┌─────────────────────┐  ┌──────────┐ ┌──────────┐│
│  │     Eden Space       │  │   S0     │ │   S1     ││
│  │                      │  │(Survivor)│ │(Survivor)││
│  │  New objects born    │  │          │ │          ││
│  │  here first          │  │ Active   │ │ Inactive ││
│  │  (~80% of Young Gen) │  │ (10%)    │ │  (10%)   ││
│  └─────────────────────┘  └──────────┘ └──────────┘│
└─────────────────────────────────────────────────────┘
```

### Object Movement Lifecycle

```
Step 1: New object → Eden Space
Step 2: Eden full → Minor GC triggered
Step 3: Surviving objects → S0 (age = 1)
Step 4: Next Minor GC → Eden survivors + S0 survivors → S1 (age++)
Step 5: S0 and S1 alternate each GC cycle
Step 6: Object age > threshold (default 15) → promoted to Old Gen
Step 7: Old Gen full → Major GC / Full GC triggered
```

### Key Facts

| Space | Default Size | Trigger |
|-------|-------------|---------|
| Eden | ~80% of Young Gen | Fills up quickly |
| S0 / S1 | ~10% each | Alternating survivors |
| Old Gen | Separate (larger) | Long-lived objects |

- **Minor GC** = cleans Young Gen only (fast, frequent)
- **Aging threshold** = `-XX:MaxTenuringThreshold` (default 15)
- Objects too large for Eden go directly to Old Gen

> 💡 **Interview Tip:** Only ONE survivor space is active at a time — the other is always empty. This is by design (copying GC algorithm).

---

## 6. Garbage Collection (GC)

> **One-liner:** GC automatically reclaims memory by removing objects no longer reachable from any live reference.

### Types of GC Events

| Type | Scope | Frequency | Pause |
|------|-------|-----------|-------|
| **Minor GC** | Young Generation only | Very frequent | Short (ms) |
| **Major GC** | Old Generation | Less frequent | Longer (seconds) |
| **Full GC** | Entire heap + Metaspace | Least frequent | Longest (worst case) |

### GC Algorithms

| Algorithm | Flag | Best For | Notes |
|-----------|------|----------|-------|
| **Serial GC** | `-XX:+UseSerialGC` | Small apps, single-core | Single-threaded, stop-the-world |
| **Parallel GC** | `-XX:+UseParallelGC` | Batch processing, throughput | Multi-threaded, default Java 8 |
| **CMS GC** | `-XX:+UseConcMarkSweepGC` | Low-latency apps | Mostly concurrent, deprecated Java 9+ |
| **G1 GC** | `-XX:+UseG1GC` | Large heaps, balanced | Default Java 9+, region-based |
| **ZGC** | `-XX:+UseZGC` | Ultra-low latency | Java 11+, sub-millisecond pauses |
| **Shenandoah** | `-XX:+UseShenandoahGC` | Low pause time | Java 12+, concurrent compaction |

### Stop-the-World (STW)

```
Normal execution:  Thread1 ──────────────────────────►
                   Thread2 ──────────────────────────►
                   Thread3 ──────────────────────────►

During STW GC:     Thread1 ────┤         ├────────────►
                   Thread2 ────┤  PAUSED  ├────────────►
                   Thread3 ────┤  (GC)    ├────────────►
                                └─────────┘
                                STW pause (ms to seconds)
```

- All application threads halt during STW phases
- G1 GC minimizes STW by doing most work concurrently

### GC Reachability

```java
String s = new String("hello"); // object reachable via 's'
s = null;                        // object now unreachable → eligible for GC
System.gc();                     // hint to JVM (not guaranteed)
```

> 💡 **Interview Tip:** `System.gc()` is just a **hint** — JVM may ignore it. Calling it in production code is bad practice.

---

## 7. Object Lifecycle in Java

```
┌─────────────────────────────────────────────────────────────────┐
│                     OBJECT LIFECYCLE                            │
│                                                                 │
│  1. CREATED       new MyClass()  → allocated in Eden Space      │
│         │                                                       │
│         ▼                                                       │
│  2. IN USE        Has at least one strong reference             │
│         │                                                       │
│         ▼                                                       │
│  3. INVISIBLE     Reference out of scope but not yet null       │
│         │                                                       │
│         ▼                                                       │
│  4. UNREACHABLE   No references point to it → GC eligible       │
│         │                                                       │
│         ▼                                                       │
│  5. COLLECTED     GC reclaims memory                            │
└─────────────────────────────────────────────────────────────────┘
```

### finalize() Method

```java
class MyClass {
    @Override
    protected void finalize() throws Throwable {
        System.out.println("Object being garbage collected");
        super.finalize();
    }
}
```

- Called by GC **before** collecting object (not guaranteed, not timely)
- **Deprecated in Java 9**, removed in Java 18
- ❌ Never rely on `finalize()` for resource cleanup
- ✅ Use `try-with-resources` or `Cleaner` (Java 9+) instead

> ⚠️ **Warning:** `finalize()` can **resurrect** objects (by creating a new reference to `this`). This is a serious anti-pattern.

---

## 8. Strong, Weak, Soft & Phantom References

> Java provides 4 reference types in `java.lang.ref` to give you control over GC eligibility.

| Reference Type | GC Behavior | Cleared When | Real-World Use |
|---------------|-------------|--------------|----------------|
| **Strong** | Never collected if referenced | Manually set to `null` | Regular object usage |
| **Soft** | Collected only when OOM approaching | JVM under memory pressure | Memory-sensitive caches |
| **Weak** | Collected at next GC | Next GC cycle | `WeakHashMap`, event listeners |
| **Phantom** | Already dead when enqueued | After `finalize()` | Cleanup actions, off-heap resources |

### Code Examples

```java
import java.lang.ref.*;

// Strong Reference (default)
String strong = new String("hello"); // not collected while 'strong' is in scope

// Soft Reference — good for caches
SoftReference<byte[]> cache = new SoftReference<>(new byte[1024 * 1024]);
byte[] data = cache.get(); // returns null if GC cleared it
if (data == null) {
    data = loadDataFromDisk(); // reload
}

// Weak Reference — good for metadata/listeners
WeakReference<User> weakUser = new WeakReference<>(new User("Alice"));
User user = weakUser.get(); // may return null after GC
if (user != null) {
    process(user);
}

// WeakHashMap — keys collected when no strong ref exists
Map<User, SessionData> sessions = new WeakHashMap<>();
// When User object has no other strong references → entry auto-removed

// Phantom Reference — for post-GC cleanup
ReferenceQueue<Object> queue = new ReferenceQueue<>();
PhantomReference<Object> phantom = new PhantomReference<>(new Object(), queue);
// phantom.get() always returns null
// Useful for knowing when object is about to be collected
```

> 💡 **Interview Tip:** `SoftReference` → collected on OOM (great for caches). `WeakReference` → collected on next GC (great for canonicalized maps). `PhantomReference.get()` **always returns null**.

---

## 9. Memory Leaks in Java

> **One-liner:** Memory leak = objects still referenced but never used again → GC cannot collect them.

### Common Causes & Fixes

---

#### ❌ 1. Static Collections

```java
// BAD — static map grows forever, objects never removed
class Cache {
    private static Map<String, byte[]> cache = new HashMap<>();

    public static void store(String key, byte[] data) {
        cache.put(key, data); // LEAK — never cleaned up!
    }
}

// GOOD — use WeakHashMap or bounded cache with eviction
private static Map<String, byte[]> cache = new WeakHashMap<>();
// or use Guava Cache / Caffeine with size/time limits
```

---

#### ❌ 2. Unclosed Resources

```java
// BAD — connection never closed → resource leak
public void readData() throws Exception {
    Connection conn = DriverManager.getConnection(url);
    // ... if exception thrown, conn never closed
    conn.close();
}

// GOOD — try-with-resources ensures close() always called
public void readData() throws Exception {
    try (Connection conn = DriverManager.getConnection(url);
         Statement stmt = conn.createStatement()) {
        // auto-closed even on exception
    }
}
```

---

#### ❌ 3. Event Listener Leaks

```java
// BAD — listener registered but never removed
button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent e) { /* uses outer class */ }
});
// Outer class held alive by listener → can't be GC'd

// GOOD — store and remove listener when done
ActionListener listener = e -> handleClick();
button.addActionListener(listener);
// ... later:
button.removeActionListener(listener);
```

---

#### ❌ 4. ThreadLocal Misuse

```java
// BAD — ThreadLocal value not removed after use
private static ThreadLocal<HeavyObject> threadLocal = new ThreadLocal<>();

public void process() {
    threadLocal.set(new HeavyObject()); // set value
    // ... do work
    // FORGET to remove → object stays in thread's memory (especially bad with thread pools)
}

// GOOD — always remove in finally block
public void process() {
    try {
        threadLocal.set(new HeavyObject());
        // ... do work
    } finally {
        threadLocal.remove(); // critical in thread pool environments
    }
}
```

---

#### ❌ 5. Inner Class Holding Outer Reference

```java
// BAD — non-static inner class holds implicit reference to outer class
class Outer {
    byte[] largeData = new byte[1024 * 1024];

    class Inner { } // holds reference to Outer → Outer can't be GC'd while Inner lives
}

// GOOD — use static inner class
class Outer {
    byte[] largeData = new byte[1024 * 1024];

    static class Inner { } // no implicit reference to Outer
}
```

> ⚠️ **Warning:** In Java, you can't always see memory leaks with code review alone. Use tools: **VisualVM**, **Eclipse MAT**, **JProfiler**, or `jmap`/`jstat`.

---

## 10. OutOfMemoryError Types

### Summary Table

| OOM Type | Cause | Solution |
|----------|-------|----------|
| `Java heap space` | Heap full, objects not GC'd | Increase `-Xmx`, fix leaks |
| `GC overhead limit exceeded` | GC spending >98% time recovering <2% heap | Increase heap, reduce object creation |
| `Metaspace` | Too many class definitions loaded | Set `-XX:MaxMetaspaceSize`, fix classloader leaks |
| `Unable to create new native thread` | Too many threads created | Reduce thread count, increase OS limit |
| `Direct buffer memory` | NIO ByteBuffer.allocateDirect() overuse | Increase `-XX:MaxDirectMemorySize` |
| `StackOverflowError` | Infinite/deep recursion | Fix recursion, increase `-Xss` |

---

### OOM Examples

```java
// 1. Java heap space
List<byte[]> list = new ArrayList<>();
while (true) {
    list.add(new byte[1024 * 1024]); // 1MB per iteration → OOM
}
// Fix: -Xmx512m → -Xmx2g  or  fix the logic

// 2. GC overhead limit exceeded
Map<String, String> map = new HashMap<>();
int i = 0;
while (true) {
    map.put("key" + i, "value" + i++); // fills heap, GC spins
}

// 3. Metaspace (e.g., using Javassist/CGLib to generate endless classes)
while (true) {
    ClassPool pool = ClassPool.getDefault();
    CtClass cc = pool.makeClass("DynamicClass" + i++);
    Class<?> clazz = cc.toClass(); // classloader leak → Metaspace OOM
}

// 4. Unable to create native thread
while (true) {
    new Thread(() -> {
        try { Thread.sleep(100000); } catch (Exception e) {}
    }).start(); // OS thread limit reached
}
```

---

## 11. StackOverflowError

> Caused by **infinitely deep** method call chains — stack frames exceed stack size.

```java
// Direct recursion — classic cause
public void test() {
    test(); // calls itself indefinitely
}
// java.lang.StackOverflowError

// Indirect recursion
public void methodA() { methodB(); }
public void methodB() { methodA(); } // ping-pong → StackOverflow
```

### Fix: Use Iteration or Increase Stack Size

```java
// BAD — recursive fibonacci
public int fib(int n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2); // deep recursion for large n
}

// GOOD — iterative fibonacci
public int fib(int n) {
    if (n <= 1) return n;
    int a = 0, b = 1;
    for (int i = 2; i <= n; i++) {
        int temp = a + b;
        a = b;
        b = temp;
    }
    return b;
}

// Or increase stack size (last resort):
// java -Xss4m MyApp
```

> ⚠️ **Warning:** `StackOverflowError` extends `Error` (not `Exception`) — catching it is possible but almost always wrong. Fix the recursion instead.

---

## 12. JVM Tuning Basics

### Core JVM Memory Flags

```bash
# Heap sizing
-Xms512m              # Initial heap size (start size)
-Xmx2g                # Maximum heap size
-Xmn256m              # Young generation size

# Stack sizing
-Xss512k              # Stack size per thread (default ~512KB–1MB)

# Metaspace
-XX:MetaspaceSize=128m      # Initial Metaspace size
-XX:MaxMetaspaceSize=256m   # Max Metaspace (always set in production!)

# GC selection
-XX:+UseG1GC              # G1 GC (recommended default)
-XX:+UseParallelGC         # Throughput-optimized GC
-XX:+UseZGC               # Ultra-low latency (Java 11+)

# GC logging
-Xlog:gc*                  # Java 9+ unified GC logging
-XX:+PrintGCDetails        # Java 8 GC details (deprecated in 9+)

# GC pause tuning (G1)
-XX:MaxGCPauseMillis=200   # Target max pause time (soft goal)
-XX:G1HeapRegionSize=16m   # G1 region size
```

### Heap Sizing Best Practices

| Rule | Recommendation |
|------|----------------|
| **Initial = Max** | Set `-Xms` = `-Xmx` to avoid heap resizing pauses |
| **Young Gen size** | ~25–33% of total heap |
| **Leave OS headroom** | Heap should not exceed 75–80% of total RAM |
| **Production minimum** | `-Xmx` at least 512m for any non-trivial app |

### Quick Diagnostic Commands

```bash
# Heap usage stats
jstat -gcutil <pid> 1000          # GC stats every 1 second

# Heap dump (for analysis in Eclipse MAT)
jmap -dump:format=b,file=heap.hprof <pid>

# Thread dump
jstack <pid>

# JVM info
jcmd <pid> VM.info
jcmd <pid> GC.heap_info

# Live JVM monitoring
jconsole                           # GUI monitoring
jvisualvm                          # Profiling + heap analysis
```

> 💡 **Interview Tip:** Setting `-Xms` equal to `-Xmx` eliminates heap resizing overhead — recommended for production servers with predictable workloads.

---

## 13. JVM Memory Diagram

```
┌──────────────────────────────────────────────────────────────────────────┐
│                          JVM MEMORY LAYOUT                               │
│                                                                          │
│  ┌───────────────────────────────────────────────────────────────────┐   │
│  │                         H E A P                                   │   │
│  │   ┌─────────────────────────────────┐  ┌───────────────────────┐ │   │
│  │   │      Young Generation           │  │   Old Generation      │ │   │
│  │   │  ┌──────────┬──────┬──────┐     │  │   (Tenured Space)     │ │   │
│  │   │  │  Eden    │  S0  │  S1  │     │  │                       │ │   │
│  │   │  │  (~80%)  │(~10%)│(~10%)│─────┼─►│  Long-lived objects   │ │   │
│  │   │  └──────────┴──────┴──────┘     │  │  (age > threshold)   │ │   │
│  │   └─────────────────────────────────┘  └───────────────────────┘ │   │
│  └───────────────────────────────────────────────────────────────────┘   │
│                                                                          │
│  ┌────────────────────────┐  ┌────────────────────────────────────────┐  │
│  │      M E T A S P A C E │  │         S T A C K S  (per thread)      │  │
│  │   (Native Memory)      │  │   ┌──────────┐ ┌──────────┐           │  │
│  │  - Class metadata      │  │   │ Thread-1 │ │ Thread-2 │   ...     │  │
│  │  - Method bytecode     │  │   │ ┌──────┐ │ │ ┌──────┐ │           │  │
│  │  - Field names         │  │   │ │frame3│ │ │ │frame2│ │           │  │
│  │  - Constant pool       │  │   │ │frame2│ │ │ │frame1│ │           │  │
│  │  - Static variables    │  │   │ │frame1│ │ │ └──────┘ │           │  │
│  └────────────────────────┘  │   │ └──────┘ │ └──────────┘           │  │
│                              │   └──────────┘                         │  │
│  ┌────────────────────────┐  └────────────────────────────────────────┘  │
│  │    Code Cache (JIT)    │                                              │
│  │  Compiled native code  │  ┌────────┐  ┌───────────────────────────┐  │
│  └────────────────────────┘  │   PC   │  │  Native Method Stacks     │  │
│                              │  Regs  │  └───────────────────────────┘  │
│                              └────────┘                                  │
└──────────────────────────────────────────────────────────────────────────┘
```

---

## 14. Important Interview Questions

### ❓ Why is String stored in the heap?

> All objects (including `String`) are on the heap. Java maintains a **String Pool** (in heap since Java 7) for interned strings to allow reuse. `new String("hello")` always creates a new heap object. `"hello"` (literal) reuses from the pool.

```java
String s1 = "hello";         // String pool (heap)
String s2 = "hello";         // same pool reference
String s3 = new String("hello"); // new heap object

System.out.println(s1 == s2);  // true  (same pool reference)
System.out.println(s1 == s3);  // false (different object)
System.out.println(s1.equals(s3)); // true (same content)
```

---

### ❓ Why are local variables thread-safe?

> Each thread has its **own stack**. Local variables exist within a method's stack frame, which is private to that thread. No other thread can access another thread's stack.

---

### ❓ What is the difference between heap and stack?

> Stack = thread-private, method scope, fast, auto-managed, stores primitives/references.
> Heap = shared, object scope, GC-managed, stores objects/arrays.

---

### ❓ Why was PermGen removed?

> PermGen was a fixed-size region inside the heap that frequently caused `OutOfMemoryError` in enterprise apps with many classes or hot redeployments. Metaspace replaced it, using native (OS) memory which grows dynamically — far less prone to OOM.

---

### ❓ What causes memory leaks in Java?

> GC can't collect objects still referenced. Common causes: static collections that grow unboundedly, unclosed streams/connections, event listeners never removed, `ThreadLocal` values not cleaned up after use.

---

### ❓ Explain the GC process.

> 1. Objects created in **Eden**. 2. Minor GC: Eden full → live objects moved to Survivor (S0 or S1) with age incremented. 3. Objects reaching age threshold promoted to **Old Gen**. 4. Old Gen full → Major/Full GC. 5. GC traces reachability from **GC roots** (static vars, stack vars, JNI refs) — anything unreachable is collected.

---

### ❓ What is a GC Root?

```
GC Roots:
- Local variables in active stack frames
- Active Java threads
- Static variables (class-level)
- JNI (native) references
```

---

### ❓ What is the difference between Minor, Major, and Full GC?

| GC Type | Area Cleaned | Frequency | Pause Impact |
|---------|-------------|-----------|--------------|
| Minor | Young Gen only | Very frequent | Low |
| Major | Old Gen | Less frequent | High |
| Full | Young + Old + Metaspace | Rare | Very high (avoid!) |

---

## 15. Quick Revision Cheatsheet

### One-Line Definitions

```
JVM           → Executes Java bytecode; manages memory, GC, threads
Heap          → Shared memory; stores all objects; GC-managed
Stack         → Per-thread; stores method frames & local vars; auto-freed
Metaspace     → Native memory; stores class metadata (Java 8+, replaced PermGen)
Eden Space    → Where new objects are created first
Survivor (S0/S1) → Intermediate space between Eden and Old Gen
Old Gen       → Long-lived objects that survived multiple GCs
Minor GC      → Cleans Young Gen (fast, frequent)
Full GC       → Cleans entire heap + Metaspace (slow, avoid)
GC Root       → Entry point for reachability analysis
Memory Leak   → Object still referenced but never used again → GC can't collect
Strong Ref    → Default; prevents GC
Soft Ref      → Cleared on OOM; good for caches
Weak Ref      → Cleared on next GC; good for metadata maps
Phantom Ref   → Post-finalize cleanup; get() always null
finalize()    → Deprecated; unreliable cleanup; never rely on it
OOM Error     → Heap/Metaspace exhausted; or too many threads
StackOverflow → Infinite recursion depth exceeded stack limit
```

### Key JVM Flags Reference

```bash
# Heap
-Xms<size>             Initial heap (e.g. -Xms512m)
-Xmx<size>             Max heap    (e.g. -Xmx4g)
-Xmn<size>             Young gen size

# Stack
-Xss<size>             Stack size per thread (e.g. -Xss512k)

# Metaspace
-XX:MetaspaceSize=<n>
-XX:MaxMetaspaceSize=<n>

# GC selection
-XX:+UseG1GC           G1 (default Java 9+, recommended)
-XX:+UseParallelGC     Throughput GC (default Java 8)
-XX:+UseZGC            Low-latency GC (Java 11+)

# GC tuning
-XX:MaxGCPauseMillis=200
-XX:MaxTenuringThreshold=15

# Diagnostics
-Xlog:gc*              GC logging (Java 9+)
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/tmp/heap.hprof

# Code cache (JIT)
-XX:ReservedCodeCacheSize=256m
```

### Memory Hierarchy Summary

```
Speed:  Registers > Code Cache > Stack > Heap > Metaspace > Disk
Size:   Registers < Code Cache < Stack < Heap < Metaspace (native)

Thread-safe areas: Stack, PC Register, Native Method Stack
Shared areas:      Heap, Metaspace, Code Cache
```

### GC Algorithm Decision Guide

```
Small app / dev machine      → Serial GC       (-XX:+UseSerialGC)
High throughput batch jobs   → Parallel GC     (-XX:+UseParallelGC)
Large heap, balanced         → G1 GC           (-XX:+UseG1GC)       ← recommended
Ultra-low latency (<1ms)     → ZGC / Shenandoah (-XX:+UseZGC)
```

### Object Lifecycle Quick Reference

```
new Object()  →  Eden  →  [Minor GC]  →  Survivor (age++)
              →  [age > 15]  →  Old Gen  →  [Full GC]  →  Collected
```

---

*📌 Java 8–21 compatible | ☕ JVM Internals Ready | Last Updated: 2024*
