
## Table of Contents

- [[#1. Collection Hierarchy|1. Collection Hierarchy]]
- [[#2. List|2. List]]
	- [[#2. List#ArrayList|ArrayList]]
	- [[#2. List#LinkedList|LinkedList]]
- [[#3. Set|3. Set]]
	- [[#3. Set#HashSet|HashSet]]
	- [[#3. Set#LinkedHashSet|LinkedHashSet]]
	- [[#3. Set#TreeSet|TreeSet]]
- [[#4. Queue & Deque|4. Queue & Deque]]
	- [[#4. Queue & Deque#PriorityQueue|PriorityQueue]]
	- [[#4. Queue & Deque#ArrayDeque|ArrayDeque]]
	- [[#4. Queue & Deque#Stack|Stack]]
- [[#5. Map|5. Map]]
	- [[#5. Map#HashMap|HashMap]]
	- [[#5. Map#LinkedHashMap|LinkedHashMap]]
	- [[#5. Map#TreeMap|TreeMap]]
- [[#6. Comparable vs Comparator|6. Comparable vs Comparator]]
	- [[#6. Comparable vs Comparator#Comparable `[core]`|Comparable `[core]`]]
	- [[#6. Comparable vs Comparator#Comparator `[core]`|Comparator `[core]`]]
- [[#7. Fail-Fast vs Fail-Safe|7. Fail-Fast vs Fail-Safe]]
	- [[#7. Fail-Fast vs Fail-Safe#Fail-Fast `[⚠ trap]`|Fail-Fast `[⚠ trap]`]]
	- [[#7. Fail-Fast vs Fail-Safe#Fail-Safe `[core]`|Fail-Safe `[core]`]]
- [[#8. Thread-Safe Collections|8. Thread-Safe Collections]]
	- [[#8. Thread-Safe Collections#ConcurrentHashMap `[core]`|ConcurrentHashMap `[core]`]]
	- [[#8. Thread-Safe Collections#CopyOnWriteArrayList `[tip]`|CopyOnWriteArrayList `[tip]`]]
	- [[#8. Thread-Safe Collections#Other Thread-Safe Options|Other Thread-Safe Options]]
- [[#9. Immutable Collections|9. Immutable Collections]]
- [[#10. Java 8 — Streams with Collections|10. Java 8 — Streams with Collections]]
- [[#11. Time Complexity Master Table|11. Time Complexity Master Table]]
	- [[#11. Time Complexity Master Table#List|List]]
	- [[#11. Time Complexity Master Table#Set|Set]]
	- [[#11. Time Complexity Master Table#Map|Map]]
	- [[#11. Time Complexity Master Table#Queue|Queue]]
- [[#12. Common Interview Questions|12. Common Interview Questions]]
- [[#13. Quick Cheat Sheet|13. Quick Cheat Sheet]]
	- [[#13. Quick Cheat Sheet#When to Use Which Collection?|When to Use Which Collection?]]
	- [[#13. Quick Cheat Sheet#Key Null Handling|Key Null Handling]]
	- [[#13. Quick Cheat Sheet#Memory Tricks|Memory Tricks]]



---

##  1. Collection Hierarchy

```
java.lang.Iterable
    └── java.util.Collection
            ├── List (ordered, index-based, duplicates allowed)
            │     ├── ArrayList
            │     ├── LinkedList
            │     └── Vector → Stack
            ├── Set (no duplicates)
            │     ├── HashSet
            │     │     └── LinkedHashSet
            │     └── SortedSet → NavigableSet → TreeSet
            └── Queue (FIFO)
                  ├── PriorityQueue
                  └── Deque (double-ended)
                        ├── ArrayDeque
                        └── LinkedList

java.util.Map (key-value, NOT part of Collection)
    ├── HashMap
    │     └── LinkedHashMap
    ├── SortedMap → NavigableMap → TreeMap
    ├── Hashtable (legacy)
    └── ConcurrentHashMap
```

> **Memory trick:** `List` = sequence · `Set` = uniqueness · `Queue` = processing order · `Map` = lookup

---

## 2. List

### ArrayList

**Internal working:** Backed by a `Object[]` array. Default capacity = **10**. When full, grows by **50%** (`newCapacity = oldCapacity * 3/2 + 1`). Elements are contiguous in memory.

```java
List<String> list = new ArrayList<>();         // capacity 10
List<String> list = new ArrayList<>(50);       // pre-size — avoids resizing

list.add("A");                    // O(1) amortized
list.add(1, "B");                 // O(n) — shifts elements right
list.get(2);                      // O(1) — random access
list.remove(Integer.valueOf("A")); // O(n) — search + shift
list.contains("A");               // O(n)
list.size();                      // O(1)

// Iterate (preferred)
for (String s : list) { }
list.forEach(System.out::println);

// Sort
Collections.sort(list);
list.sort(Comparator.reverseOrder());

// Convert array ↔ list
String[] arr = list.toArray(new String[0]);
List<String> fromArr = new ArrayList<>(Arrays.asList(arr));
List<String> fixed    = Arrays.asList("a","b","c"); // fixed size!
```

**Interview points:**
- `Arrays.asList()` returns a **fixed-size** list backed by the array — no add/remove.
- `ArrayList` is **not thread-safe** — use `Collections.synchronizedList()` or `CopyOnWriteArrayList`.
- Pre-size with `new ArrayList<>(n)` when size is known — avoids repeated resizing.

---

### LinkedList

**Internal working:** Doubly-linked list. Each node holds `prev`, `next`, `data`. Also implements `Deque` — can be used as stack or queue.

```java
LinkedList<String> ll = new LinkedList<>();

// List operations
ll.add("A");           // O(1) — add to tail
ll.add(1, "B");        // O(n) — traverse to index, then O(1) insert
ll.get(2);             // O(n) — no random access
ll.remove("A");        // O(n) — search + O(1) remove

// Deque operations
ll.addFirst("Z");      // O(1)
ll.addLast("X");       // O(1)
ll.peekFirst();        // O(1) — null if empty
ll.pollFirst();        // O(1) — removes head, null if empty
ll.push("Q");          // same as addFirst (stack)
ll.pop();              // same as removeFirst (stack)
```

**ArrayList vs LinkedList:**

| | ArrayList | LinkedList |
|---|---|---|
| Random access | O(1) ✅ | O(n) ❌ |
| Insert/delete at middle | O(n) | O(n) traverse + O(1) |
| Insert/delete at head | O(n) | O(1) ✅ |
| Memory | Compact array | Extra node pointers (~40B/node) |
| Cache performance | Better (contiguous) | Worse (pointer chasing) |
| Use when | Read-heavy | Frequent head insert/delete |

---

## 3. Set

### HashSet

**Internal working:** Backed by a `HashMap<E, PRESENT>` internally. Element stored as key; value is a dummy constant object. Relies on `hashCode()` + `equals()`.

```java
Set<String> set = new HashSet<>();
set.add("A");      // O(1) avg
set.contains("A"); // O(1) avg
set.remove("A");   // O(1) avg
set.size();        // O(1)

// No guaranteed order
Set<Integer> nums = new HashSet<>(Arrays.asList(3, 1, 2, 1)); // {1, 2, 3} — order varies

// Set operations (manual)
Set<Integer> a = new HashSet<>(Arrays.asList(1, 2, 3));
Set<Integer> b = new HashSet<>(Arrays.asList(2, 3, 4));

Set<Integer> union        = new HashSet<>(a); union.addAll(b);       // {1,2,3,4}
Set<Integer> intersection = new HashSet<>(a); intersection.retainAll(b); // {2,3}
Set<Integer> difference   = new HashSet<>(a); difference.removeAll(b);   // {1}
```

**⚠ Trap:** If you override `equals()` but not `hashCode()`, two "equal" objects can end up in two different buckets — HashSet won't detect them as duplicates.

---

### LinkedHashSet

**Internal working:** Extends `HashSet` but maintains a **doubly-linked list** across all entries — preserves **insertion order**.

```java
Set<String> lhs = new LinkedHashSet<>();
lhs.add("Banana"); lhs.add("Apple"); lhs.add("Cherry");
System.out.println(lhs); // [Banana, Apple, Cherry] — insertion order preserved
```

**Use when:** You need uniqueness + predictable iteration order.

---

### TreeSet

**Internal working:** Backed by a `TreeMap` (Red-Black tree). Elements sorted in **natural order** (or custom `Comparator`). All ops O(log n).

```java
TreeSet<Integer> ts = new TreeSet<>();
ts.add(5); ts.add(2); ts.add(8); ts.add(1);
// Stored sorted: [1, 2, 5, 8]

ts.first();           // 1
ts.last();            // 8
ts.floor(4);          // 2 — largest element ≤ 4
ts.ceiling(4);        // 5 — smallest element ≥ 4
ts.lower(5);          // 2 — strictly less than 5
ts.higher(5);         // 8 — strictly greater than 5
ts.headSet(5);        // [1, 2]   — exclusive of 5
ts.tailSet(5);        // [5, 8]   — inclusive of 5
ts.subSet(2, 8);      // [2, 5]   — [2..8)

// Custom order
TreeSet<String> byLen = new TreeSet<>(Comparator.comparingInt(String::length));
```

**⚠ Trap:** `TreeSet` uses `compareTo()` (not `equals()`) to determine duplicates. If `compareTo()` returns 0, elements are considered equal and won't be inserted — even if `equals()` says otherwise.

---

## 4. Queue & Deque

### PriorityQueue

**Internal working:** **Min-heap** (binary heap backed by array). Smallest element always at head. NOT thread-safe.

```java
PriorityQueue<Integer> pq = new PriorityQueue<>(); // natural order (min-heap)
pq.offer(5); pq.offer(1); pq.offer(3);

pq.peek();   // 1 — view head, O(1)
pq.poll();   // 1 — remove head, O(log n)
pq.size();   // 2

// Max-heap
PriorityQueue<Integer> maxPq = new PriorityQueue<>(Comparator.reverseOrder());

// Custom object
PriorityQueue<Employee> bysal = new PriorityQueue<>(
    Comparator.comparingInt(Employee::getSalary));
```

**⚠ Trap:** Iterating a `PriorityQueue` does NOT give sorted order — only `poll()` gives sorted output.

---

### ArrayDeque

**Internal working:** Resizable circular array. Faster than `LinkedList` as a stack/queue (no node allocation). Preferred over `Stack` class.

```java
ArrayDeque<String> deque = new ArrayDeque<>();

// As Queue (FIFO)
deque.offer("A");   // add to tail
deque.offer("B");
deque.poll();        // remove from head — "A"
deque.peek();        // view head — "B"

// As Stack (LIFO)
deque.push("X");    // add to head
deque.push("Y");
deque.pop();         // remove from head — "Y"
```

---

### Stack

**Legacy class** — extends `Vector` (synchronized). Prefer `ArrayDeque` over `Stack`.

```java
Stack<Integer> stack = new Stack<>();
stack.push(1); stack.push(2); stack.push(3);
stack.peek();   // 3
stack.pop();    // 3
stack.isEmpty(); // false

// PREFER:
Deque<Integer> stack2 = new ArrayDeque<>();
stack2.push(1); stack2.pop();
```

---

## 5. Map

### HashMap

**Internal working:** Array of `Node<K,V>[]` buckets (default 16). Key → `hashCode()` → index. Collision handled by **chaining** (linked list). Java 8+: chain becomes a **Red-Black tree** when bucket size ≥ 8 (treeification). Load factor = **0.75** by default. Resizes (doubles) when `size > capacity * loadFactor`.

```java
Map<String, Integer> map = new HashMap<>();   // default: 16 buckets, LF 0.75
Map<String, Integer> map = new HashMap<>(64, 0.5f); // custom capacity + LF

map.put("a", 1);                        // O(1) avg
map.get("a");                           // O(1) avg — null if missing
map.getOrDefault("b", 0);              // 0
map.containsKey("a");                   // O(1)
map.containsValue(1);                   // O(n)
map.remove("a");                        // O(1) avg
map.size();                             // O(1)

// Compute methods (Java 8)
map.putIfAbsent("b", 2);               // insert only if absent
map.computeIfAbsent("c", k -> 3);      // compute & insert if absent
map.computeIfPresent("b", (k,v)->v*2); // update only if present
map.compute("a", (k,v)-> v==null?1:v+1); // always compute
map.merge("a", 1, Integer::sum);        // insert or merge with BiFunction

// Iterate
map.forEach((k, v) -> System.out.println(k + "=" + v));
for (Map.Entry<String, Integer> e : map.entrySet()) { }
```

**Internal hashing:**
```java
// HashMap computes hash as:
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// bucket index = (n-1) & hash
```

**Interview points:**
- Allows **one null key** and **multiple null values**.
- Not thread-safe — use `ConcurrentHashMap` for concurrent access.
- Worst case O(n) (all keys same bucket) → O(log n) after treeification.

---

### LinkedHashMap

**Internal working:** Extends `HashMap` + maintains a **doubly-linked list** across all entries. Preserves **insertion order** by default, or **access order** (LRU) if configured.

```java
// Insertion order
Map<String, Integer> lhm = new LinkedHashMap<>();
lhm.put("B", 2); lhm.put("A", 1); lhm.put("C", 3);
System.out.println(lhm.keySet()); // [B, A, C]

// LRU Cache pattern (access order)
Map<Integer, String> lru = new LinkedHashMap<>(16, 0.75f, true) {
    @Override
    protected boolean removeEldestEntry(Map.Entry<Integer, String> eldest) {
        return size() > 3; // max 3 entries
    }
};
lru.put(1, "A"); lru.put(2, "B"); lru.put(3, "C");
lru.get(1);      // access 1 → moves to tail
lru.put(4, "D"); // evicts eldest (2)
System.out.println(lru.keySet()); // [3, 1, 4]
```

---

### TreeMap

**Internal working:** Red-Black tree. Keys sorted in natural order or by `Comparator`. All ops O(log n).

```java
TreeMap<String, Integer> tm = new TreeMap<>();
tm.put("banana", 2); tm.put("apple", 1); tm.put("cherry", 3);
// keys sorted: apple < banana < cherry

tm.firstKey();           // "apple"
tm.lastKey();            // "cherry"
tm.floorKey("b");        // "banana" — largest key ≤ "b"
tm.ceilingKey("b");      // "banana" — smallest key ≥ "b"
tm.lowerKey("banana");   // "apple"
tm.higherKey("banana");  // "cherry"
tm.headMap("cherry");    // {apple=1, banana=2}
tm.tailMap("banana");    // {banana=2, cherry=3}
tm.subMap("apple","cherry"); // {apple=1, banana=2}

// Descending
NavigableMap<String, Integer> desc = tm.descendingMap();
```

---

## 6. Comparable vs Comparator

### Comparable `[core]`
- Defines **natural ordering** — implemented on the class itself.
- `compareTo()` returns negative (less), 0 (equal), positive (greater).
- Used by `Collections.sort()`, `TreeSet`, `TreeMap` by default.

```java
class Employee implements Comparable<Employee> {
    String name;
    int salary;

    @Override
    public int compareTo(Employee other) {
        return Integer.compare(this.salary, other.salary); // ascending salary
    }
}

List<Employee> emps = getEmployees();
Collections.sort(emps); // uses compareTo()
```

---

### Comparator `[core]`
- Defines **external/custom ordering** — passed at sort time.
- Multiple comparators possible for the same class.

```java
// Old way
Comparator<Employee> byName = new Comparator<Employee>() {
    public int compare(Employee a, Employee b) {
        return a.name.compareTo(b.name);
    }
};

// Lambda (Java 8)
Comparator<Employee> byName    = (a, b) -> a.name.compareTo(b.name);
Comparator<Employee> bySalary  = Comparator.comparingInt(Employee::getSalary);
Comparator<Employee> byNameRev = Comparator.comparing(Employee::getName).reversed();

// Chaining
Comparator<Employee> compound = Comparator
    .comparing(Employee::getDept)
    .thenComparingInt(Employee::getSalary)
    .thenComparing(Employee::getName);

emps.sort(compound);
```

| | Comparable | Comparator |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `compareTo(T o)` | `compare(T o1, T o2)` |
| Location | Inside class | Outside class |
| Single ordering | ✅ (natural) | ✅ (multiple possible) |
| Modifying class needed | ✅ yes | ❌ no |
| Use when | You own the class | Third-party or multiple orders |

---

## 7. Fail-Fast vs Fail-Safe

### Fail-Fast `[⚠ trap]`
Throws `ConcurrentModificationException` if collection is structurally modified during iteration (via iterator). Detects via `modCount` internal counter.

```java
List<String> list = new ArrayList<>(Arrays.asList("A","B","C"));
Iterator<String> it = list.iterator();
while (it.hasNext()) {
    String s = it.next();
    if (s.equals("B")) {
        list.remove(s);   // ⚠ ConcurrentModificationException!
    }
}

// Safe removal during iteration
while (it.hasNext()) {
    if (it.next().equals("B")) it.remove(); // ✅ use iterator's remove
}

// Or Java 8
list.removeIf(s -> s.equals("B")); // ✅
```

---

### Fail-Safe `[core]`
Iterates over a **copy** of the collection — no `ConcurrentModificationException`. Stale reads possible. Used in `java.util.concurrent` collections.

```java
CopyOnWriteArrayList<String> cowList = new CopyOnWriteArrayList<>(Arrays.asList("A","B","C"));
for (String s : cowList) {
    cowList.add("D"); // ✅ no exception — iterates over snapshot copy
}

ConcurrentHashMap<String, Integer> chm = new ConcurrentHashMap<>();
chm.put("a", 1);
for (String k : chm.keySet()) {
    chm.put("b", 2); // ✅ no exception
}
```

| | Fail-Fast | Fail-Safe |
|---|---|---|
| Throws CME? | ✅ Yes | ❌ No |
| Iterates over | Original | Snapshot/copy |
| Memory | Less | More (copy overhead) |
| Examples | `ArrayList`, `HashMap`, `HashSet` | `CopyOnWriteArrayList`, `ConcurrentHashMap` |
| Stale reads? | No | Possible |

---

## 8. Thread-Safe Collections

### ConcurrentHashMap `[core]`
- Java 7: segment-level locking (16 segments by default).
- Java 8+: **CAS + synchronized on individual bucket** (no segment lock).
- Allows concurrent reads without locking.
- Does NOT allow null keys or null values.

```java
ConcurrentHashMap<String, Integer> chm = new ConcurrentHashMap<>();
chm.put("a", 1);
chm.putIfAbsent("b", 2);
chm.computeIfAbsent("c", k -> k.length());

// Atomic operations
chm.merge("a", 1, Integer::sum);     // thread-safe increment

// Bulk operations (Java 8)
chm.forEach(2, (k, v) -> System.out.println(k)); // parallelismThreshold=2
long sum = chm.reduceValues(1, Integer::sum);
```

---

### CopyOnWriteArrayList `[tip]`
Every write creates a **new copy** of the underlying array. Reads are lock-free. Best for read-heavy, rarely-written lists.

```java
CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<>();
list.add("A"); list.add("B");

// Safe to read while writing from another thread
for (String s : list) {
    list.add("C"); // creates new copy — no CME, iterator sees old snapshot
}
```

---

### Other Thread-Safe Options

```java
// Legacy (synchronized wrapper — coarse lock on entire collection)
List<String>        syncList = Collections.synchronizedList(new ArrayList<>());
Map<String, Integer> syncMap = Collections.synchronizedMap(new HashMap<>());

// BlockingQueue — producer/consumer
BlockingQueue<String> q = new LinkedBlockingQueue<>(10); // bounded
q.put("task");           // blocks if full
String t = q.take();     // blocks if empty
q.offer("task", 1, TimeUnit.SECONDS); // timeout version
```

| Collection | Thread-Safe? | Notes |
|---|---|---|
| `ArrayList` | ❌ | Use `CopyOnWriteArrayList` or `synchronizedList` |
| `HashMap` | ❌ | Use `ConcurrentHashMap` |
| `HashSet` | ❌ | Use `ConcurrentHashMap.newKeySet()` |
| `Vector` | ✅ | Legacy — synchronized, slow |
| `Hashtable` | ✅ | Legacy — synchronized, slow |
| `ConcurrentHashMap` | ✅ | Preferred for concurrent maps |
| `CopyOnWriteArrayList` | ✅ | Read-heavy scenarios |
| `LinkedBlockingQueue` | ✅ | Producer-consumer pattern |

---

## 9. Immutable Collections

```java
// Java 9+ factory methods (most concise)
List<String>        list = List.of("A", "B", "C");
Set<String>         set  = Set.of("X", "Y");
Map<String,Integer> map  = Map.of("a", 1, "b", 2);
Map<String,Integer> map2 = Map.ofEntries(
    Map.entry("key1", 1),
    Map.entry("key2", 2)
);

// Java 8 — Collections utility
List<String> immutable = Collections.unmodifiableList(new ArrayList<>(list));

// list.add("D"); // ⚠ throws UnsupportedOperationException

// List.of vs Arrays.asList
// List.of       — truly immutable, no nulls allowed
// Arrays.asList — fixed size (no add/remove) but set() allowed, nulls OK
```

---

## 10. Java 8 — Streams with Collections

```java
List<Employee> emps = getEmployees();

// Filter + Map + Collect
List<String> seniorNames = emps.stream()
    .filter(e -> e.getSalary() > 80000)
    .map(Employee::getName)
    .sorted()
    .collect(Collectors.toList());

// groupingBy
Map<String, List<Employee>> byDept =
    emps.stream().collect(Collectors.groupingBy(Employee::getDept));

// groupingBy + counting
Map<String, Long> countByDept =
    emps.stream().collect(Collectors.groupingBy(Employee::getDept, Collectors.counting()));

// toMap
Map<Integer, String> idToName =
    emps.stream().collect(Collectors.toMap(Employee::getId, Employee::getName));

// partitioningBy
Map<Boolean, List<Employee>> partition =
    emps.stream().collect(Collectors.partitioningBy(e -> e.getSalary() > 50000));

// Statistics
IntSummaryStatistics stats =
    emps.stream().collect(Collectors.summarizingInt(Employee::getSalary));

// joining
String names = emps.stream().map(Employee::getName)
    .collect(Collectors.joining(", ", "[", "]"));

// flatMap
List<List<String>> skills = emps.stream().map(Employee::getSkills).collect(Collectors.toList());
List<String> allSkills = emps.stream()
    .flatMap(e -> e.getSkills().stream())
    .distinct()
    .sorted()
    .collect(Collectors.toList());

// Stream → Map with merge (handle duplicate keys)
Map<String, Integer> mergedMap = emps.stream()
    .collect(Collectors.toMap(
        Employee::getDept,
        Employee::getSalary,
        Integer::sum  // merge function for duplicate keys
    ));
```

---

## 11. Time Complexity Master Table

### List

| Operation | ArrayList | LinkedList |
|---|---|---|
| `get(i)` | O(1) | O(n) |
| `add(end)` | O(1) amortized | O(1) |
| `add(i, e)` | O(n) | O(n) traverse + O(1) |
| `remove(i)` | O(n) | O(n) traverse + O(1) |
| `contains` | O(n) | O(n) |
| `size` | O(1) | O(1) |

### Set

| Operation | HashSet | LinkedHashSet | TreeSet |
|---|---|---|---|
| `add` | O(1) avg | O(1) avg | O(log n) |
| `remove` | O(1) avg | O(1) avg | O(log n) |
| `contains` | O(1) avg | O(1) avg | O(log n) |
| Ordering | None | Insertion | Sorted |

### Map

| Operation | HashMap | LinkedHashMap | TreeMap |
|---|---|---|---|
| `put` | O(1) avg | O(1) avg | O(log n) |
| `get` | O(1) avg | O(1) avg | O(log n) |
| `remove` | O(1) avg | O(1) avg | O(log n) |
| `containsKey` | O(1) avg | O(1) avg | O(log n) |
| Ordering | None | Insertion / Access | Sorted by key |

### Queue

| Operation | PriorityQueue | ArrayDeque |
|---|---|---|
| `offer/add` | O(log n) | O(1) amortized |
| `poll/remove` | O(log n) | O(1) |
| `peek` | O(1) | O(1) |

---

## 12. Common Interview Questions

**Q1. How does HashMap work internally?**
> Array of buckets. Key → `hashCode()` → bucket index via `(n-1) & hash`. Collision → chaining (linked list). Java 8: list becomes Red-Black tree when bucket size ≥ 8. Resizes at 75% capacity.

**Q2. What happens if two keys have the same hashCode?**
> Both go to same bucket (collision). LinkedList/tree used to store multiple entries. `equals()` differentiates keys during get/put.

**Q3. Why override both equals() and hashCode()?**
> Contract: `a.equals(b)` must imply `a.hashCode() == b.hashCode()`. Violating this breaks HashMap, HashSet — equal objects may land in different buckets and duplicates won't be detected.

**Q4. ArrayList vs LinkedList — when to use what?**
> ArrayList for frequent reads, random access. LinkedList for frequent head/tail insertions or as a Deque. In practice, ArrayList almost always wins due to cache locality.

**Q5. What is ConcurrentModificationException?**
> Thrown by fail-fast iterators when collection is structurally modified during iteration (not through iterator). Use `iterator.remove()`, `removeIf()`, or concurrent collections.

**Q6. HashMap vs ConcurrentHashMap vs Hashtable?**
> `HashMap`: not thread-safe, allows null key. `Hashtable`: thread-safe (full lock), legacy, no null. `ConcurrentHashMap`: thread-safe (bucket-level lock), no null, preferred.

**Q7. How to sort a Map by value?**
```java
map.entrySet().stream()
   .sorted(Map.Entry.comparingByValue())
   .collect(Collectors.toMap(
       Map.Entry::getKey, Map.Entry::getValue,
       (e1, e2) -> e1, LinkedHashMap::new)); // preserve sorted order
```

**Q8. What is the initial capacity and load factor of HashMap?**
> Default capacity = **16**, load factor = **0.75**. Resizes when `entries > 16 * 0.75 = 12`.

**Q9. Difference between poll() and remove() in Queue?**
> `poll()` returns null if queue is empty. `remove()` throws `NoSuchElementException`.

**Q10. How to make a collection read-only?**
> `Collections.unmodifiableList/Set/Map()` wraps it. Java 9+ `List.of()` / `Set.of()` / `Map.of()` are truly immutable.

---

## 13. Quick Cheat Sheet

### When to Use Which Collection?

| Need | Use |
|---|---|
| Ordered list, fast reads | `ArrayList` |
| Frequent insert/delete at ends | `ArrayDeque` |
| Unique elements, fast lookup | `HashSet` |
| Unique + insertion order | `LinkedHashSet` |
| Unique + sorted | `TreeSet` |
| Key-value, fast lookup | `HashMap` |
| Key-value + insertion order | `LinkedHashMap` |
| Key-value + sorted keys | `TreeMap` |
| LRU cache | `LinkedHashMap` (access-order) |
| Priority-based processing | `PriorityQueue` |
| Thread-safe map | `ConcurrentHashMap` |
| Thread-safe list (read-heavy) | `CopyOnWriteArrayList` |
| Producer-consumer | `LinkedBlockingQueue` |
| Immutable collection | `List.of()` / `Set.of()` / `Map.of()` |
| Stack (LIFO) | `ArrayDeque` (not `Stack`) |
| Queue (FIFO) | `ArrayDeque` or `LinkedList` |

---

### Key Null Handling

| Collection | Null Key | Null Value |
|---|---|---|
| `HashMap` | ✅ one | ✅ multiple |
| `LinkedHashMap` | ✅ one | ✅ multiple |
| `TreeMap` | ❌ (NPE) | ✅ multiple |
| `Hashtable` | ❌ | ❌ |
| `ConcurrentHashMap` | ❌ | ❌ |
| `HashSet` | ✅ one null | — |
| `TreeSet` | ❌ (NPE) | — |

---

### Memory Tricks

- **HashMap** → Hash + Map → `hashCode()` decides bucket
- **TreeMap/TreeSet** → Tree → sorted (think: alphabetical tree)
- **LinkedHashMap** → Linked → remembers insertion order
- **CopyOnWrite** → Writes make a fresh copy → safe for readers
- **PECS** → Producer Extends, Consumer Super
- **Fail-Fast** → Fails immediately (CME) → ArrayList, HashMap
- **Fail-Safe** → Safe to modify → ConcurrentHashMap, CopyOnWriteArrayList
- **PriorityQueue** → Min-heap by default → smallest always at top
- **ArrayDeque** → Faster stack/queue than Stack/LinkedList

---

*Next topics: Spring Boot · Spring Batch · Concurrency & Multithreading · JVM Internals · Design Patterns*
