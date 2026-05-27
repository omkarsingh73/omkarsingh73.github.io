
---

  

## Table of Contents

1. [Lambda Expressions](#1-lambda-expressions)

2. [Functional Interfaces](#2-functional-interfaces)

3. [Method References](#3-method-references)

4. [Stream API](#4-stream-api)

5. [Optional](#5-optional)

6. [Default & Static Methods in Interfaces](#6-default--static-methods-in-interfaces)

7. [New Date & Time API (java.time)](#7-new-date--time-api-javatime)

8. [Collectors](#8-collectors)

9. [Map Enhancements](#9-map-enhancements)

10. [Nashorn JavaScript Engine](#10-nashorn-javascript-engine)

11. [Base64 Encoding/Decoding](#11-base64-encodingdecoding)

12. [Quick Reference Cheat Sheet](#12-quick-reference-cheat-sheet)

  

---

  

## 1. Lambda Expressions

  

### What is it? `[core]`

A concise way to represent an anonymous function (behavior). Enables treating functions as first-class citizens. Syntax: `(params) -> expression` or `(params) -> { statements; }`.

  

```java

// Before Java 8 — anonymous class

Runnable r = new Runnable() {

@Override

public void run() { System.out.println("Running"); }

};

  

// Java 8 — lambda

Runnable r = () -> System.out.println("Running");

  

// With params

Comparator<String> c = (a, b) -> a.compareTo(b);

  

// With block body

Comparator<String> c2 = (a, b) -> {

System.out.println("Comparing...");

return a.compareTo(b);

};

```

  

---

  

### Variable Capture `[⚠ trap]`

Lambdas can capture local variables but they must be **effectively final** — assigned once and never changed.

  

```java

String prefix = "Hello"; // effectively final — OK

// prefix = "Hi"; // would break compilation

  

Consumer<String> greet = name -> System.out.println(prefix + " " + name);

greet.accept("Omkar"); // "Hello Omkar"

```

  

---

  

## 2. Functional Interfaces

  

### @FunctionalInterface `[core]`

An interface with exactly **one abstract method** (SAM). Lambda expressions implement them. `@FunctionalInterface` annotation is optional but enforced by compiler.

  

| Interface             | Signature             | Use Case                                             |
| --------------------- | --------------------- | ---------------------------------------------------- |
| `Function<T, R>`      | `R apply(T t)`        | Transform input into output                          |
| `Predicate<T>`        | `boolean test(T t)`   | Evaluate a condition                                 |
| `Consumer<T>`         | `void accept(T t)`    | Consume/process data without returning value         |
| `Supplier<T>`         | `T get()`             | Supply/provide data without input                    |
| `BiFunction<T, U, R>` | `R apply(T t, U u)`   | Accept two inputs and return output                  |
| `UnaryOperator<T>`    | `T apply(T t)`        | Same input and output type transformation            |
| `BinaryOperator<T>`   | `T apply(T t1, T t2)` | Operate on two same-type values and return same type |

  

```java

Function<String, Integer> len = String::length;

Predicate<Integer> isEven = n -> n % 2 == 0;

Consumer<String> print = System.out::println;

Supplier<List<String>> list = ArrayList::new;

UnaryOperator<String> upper = String::toUpperCase;

BinaryOperator<Integer> add = Integer::sum;

  

// Composing

Function<String, String> trim = String::trim;

Function<String, String> upper2 = String::toUpperCase;

Function<String, String> trimThenUpper = trim.andThen(upper2);

trimThenUpper.apply(" hello "); // "HELLO"

  

// Chaining Predicates

Predicate<Integer> positive = n -> n > 0;

Predicate<Integer> evenAndPositive = isEven.and(positive);

evenAndPositive.test(4); // true

evenAndPositive.test(-4); // false

```

  

---

  

### Custom Functional Interface `[tip]`

  

```java

@FunctionalInterface

interface TriFunction<A, B, C, R> {

R apply(A a, B b, C c);

}

  

TriFunction<Integer, Integer, Integer, Integer> sum =

(a, b, c) -> a + b + c;

  

sum.apply(1, 2, 3); // 6

```

  

---

  

## 3. Method References

  

### 4 Types of Method References `[core]`

Shorthand for a lambda that calls an existing method. Cleaner and more readable than a lambda.

  

```java

// 1. Static method reference: ClassName::staticMethod

Function<String, Integer> parse = Integer::parseInt;

parse.apply("42"); // 42

  

// 2. Instance method on a specific instance: instance::method

String prefix = "Hello";

Predicate<String> startsWith = prefix::startsWith; // wrong example, corrected:

Consumer<String> printer = System.out::println;

  

// 3. Instance method on arbitrary instance: ClassName::instanceMethod

Function<String, String> upper = String::toUpperCase;

upper.apply("java"); // "JAVA"

  

// 4. Constructor reference: ClassName::new

Supplier<ArrayList<String>> listFactory = ArrayList::new;

listFactory.get(); // new ArrayList<>()

  

Function<String, StringBuilder> sbFactory = StringBuilder::new;

sbFactory.apply("start"); // new StringBuilder("start")

```

  

---

  

## 4. Stream API

  

### Stream Pipeline `[core]`

A sequence of elements supporting sequential/parallel aggregate operations.

- **Source** → **Intermediate ops (lazy)** → **Terminal op (triggers execution)**

- Streams are **single-use** — cannot be reused after terminal op.

  

```java

List<String> names = List.of("Alice", "Bob", "Charlie", "David", "Eve");

  

List<String> result = names.stream() // source

.filter(n -> n.length() > 3) // intermediate — lazy

.map(String::toUpperCase) // intermediate — lazy

.sorted() // intermediate — lazy

.collect(Collectors.toList()); // terminal — triggers pipeline

  

// result: [ALICE, CHARLIE, DAVID]

```

  

---

  

### Intermediate Operations `[core]`

  

```java

Stream<T> filter(Predicate<T> p) // keep matching elements

Stream<R> map(Function<T,R> f) // transform each element

Stream<R> flatMap(Function<T,Stream<R>f>) // flatten nested streams

Stream<T> distinct() // remove duplicates

Stream<T> sorted() // natural order

Stream<T> sorted(Comparator<T> c) // custom order

Stream<T> limit(long n) // take first n

Stream<T> skip(long n) // skip first n

Stream<T> peek(Consumer<T> action) // debug — side effect

  

// flatMap example

List<List<Integer>> nested = List.of(List.of(1,2), List.of(3,4));

nested.stream()

.flatMap(Collection::stream)

.collect(Collectors.toList()); // [1, 2, 3, 4]

```

  

---

  

### Terminal Operations `[core]`

  

```java

// Collect

List<String> list = stream.collect(Collectors.toList());

Set<String> set = stream.collect(Collectors.toSet());

String joined = stream.collect(Collectors.joining(", "));

  

// Reduction

Optional<T> reduced = stream.reduce((a, b) -> ...);

T reduced = stream.reduce(identity, (a, b) -> ...);

  

long count = stream.count();

Optional<T> min = stream.min(Comparator.naturalOrder());

Optional<T> max = stream.max(Comparator.naturalOrder());

  

// Match

boolean anyMatch = stream.anyMatch(Predicate); // short-circuits

boolean allMatch = stream.allMatch(Predicate); // short-circuits

boolean noneMatch = stream.noneMatch(Predicate); // short-circuits

  

// Find

Optional<T> first = stream.findFirst(); // ordered — deterministic

Optional<T> any = stream.findAny(); // unordered — faster in parallel

  

// ForEach

stream.forEach(System.out::println);

```

  

---

  

### Numeric Streams — IntStream, LongStream, DoubleStream `[tip]`

Avoid boxing overhead. Use when working with primitives.

  

```java

IntStream.range(1, 6).sum(); // 15 (1..5)

IntStream.rangeClosed(1, 5).sum(); // 15 (1..5 inclusive)

  

IntStream.of(3, 1, 4, 1, 5).average().getAsDouble(); // 2.8

  

// Convert object stream to int stream

List<String> words = List.of("hello", "world");

int totalLen = words.stream()

.mapToInt(String::length)

.sum(); // 10

  

// Generate & Iterate

IntStream.generate(() -> 1).limit(5); // [1,1,1,1,1]

IntStream.iterate(0, n -> n + 2).limit(5); // [0,2,4,6,8]

```

  

---

  

### Parallel Streams `[⚠ trap]`

Uses ForkJoinPool under the hood. Fast for CPU-bound, large data. **Avoid** for small collections, I/O-bound tasks, or ordered operations.

  

```java

List<Integer> nums = IntStream.rangeClosed(1, 1_000_000)

.boxed().collect(Collectors.toList());

  

long sum = nums.parallelStream()

.filter(n -> n % 2 == 0)

.mapToLong(Integer::longValue)

.sum();

  

// ⚠ Trap: shared mutable state breaks parallel streams

List<Integer> unsafe = new ArrayList<>();

nums.parallelStream().forEach(unsafe::add); // RACE CONDITION!

  

// Safe: use collect instead

List<Integer> safe = nums.parallelStream()

.filter(n -> n % 2 == 0)

.collect(Collectors.toList());

```

  

---

  

## 5. Optional

  

### Creating & Using Optional `[core]`

A container that may or may not hold a non-null value. Designed to replace null-returning APIs.

  

```java

Optional<String> empty = Optional.empty();

Optional<String> present = Optional.of("Omkar"); // throws NPE if null

Optional<String> maybe = Optional.ofNullable(null); // safe for nulls

  

// Check & Get — BAD pattern

if (present.isPresent()) {

System.out.println(present.get()); // fragile

}

  

// GOOD patterns

present.ifPresent(System.out::println); // "Omkar"

present.orElse("default"); // "Omkar"

empty.orElse("default"); // "default"

empty.orElseGet(() -> computeDefault()); // lazy

empty.orElseThrow(() -> new RuntimeException("!")); // throw if empty

  

// Transform

Optional<Integer> len = present.map(String::length); // Optional[5]

present.filter(s -> s.startsWith("O")).isPresent(); // true

  

// Chaining (flatMap)

Optional<String> city = findUser(1)

.flatMap(user -> findAddress(user.getId()))

.map(Address::getCity);

```

  

---

  

### Optional `[⚠ trap]`

  

```java

// Never use Optional as a field or method parameter — only return type

// BAD:

public class User {

private Optional<String> nickname; // don't do this

}

  

// BAD:

void process(Optional<String> name) { ... } // don't do this

  

// GOOD:

public Optional<String> findNickname(int userId) { ... }

```

  

---

  

## 6. Default & Static Methods in Interfaces

  

### Default Methods `[core]`

Allow adding new methods to interfaces without breaking existing implementations. Multiple interfaces with same default method → must override in implementing class.

  

```java

interface Greeter {

String greet(String name); // abstract

  

default String greetLoudly(String name) {

return greet(name).toUpperCase(); // uses abstract method

}

}

  

class FriendlyGreeter implements Greeter {

@Override

public String greet(String name) { return "Hello, " + name; }

// greetLoudly() inherited for free

}

  

new FriendlyGreeter().greetLoudly("omkar"); // "HELLO, OMKAR"

```

  

---

  

### Diamond Problem `[⚠ trap]`

If two interfaces have the same default method, the implementing class **must** override.

  

```java

interface A { default void show() { System.out.println("A"); } }

interface B { default void show() { System.out.println("B"); } }

  

class C implements A, B {

@Override

public void show() {

A.super.show(); // explicit call to A's default

}

}

```

  

---

  

### Static Methods in Interfaces `[tip]`

Belong to the interface — not inherited by implementing classes. Useful for factory/utility methods.

  

```java

interface MathOp {

int operate(int a, int b);

  

static MathOp add() { return (a, b) -> a + b; }

static MathOp multiply() { return (a, b) -> a * b; }

}

  

MathOp.add().operate(3, 4); // 7

MathOp.multiply().operate(3, 4); // 12

```

  

---

  

## 7. New Date & Time API (java.time)

  

### Why New API? `[core]`

`java.util.Date` and `Calendar` were mutable, not thread-safe, and poorly designed. `java.time` is immutable, thread-safe, and ISO-8601 based.

  

| Class | Purpose |

|---|---|

| `LocalDate` | Date without time (2024-01-15) |

| `LocalTime` | Time without date (10:30:00) |

| `LocalDateTime` | Date + time, no timezone |

| `ZonedDateTime` | Date + time + timezone |

| `Instant` | Machine timestamp (epoch) |

| `Duration` | Time-based amount (hours, minutes) |

| `Period` | Date-based amount (years, months, days) |

| `DateTimeFormatter` | Format/parse dates |

  

---

  

### LocalDate, LocalTime, LocalDateTime `[core]`

  

```java

LocalDate today = LocalDate.now(); // 2024-01-15

LocalDate birthday = LocalDate.of(1995, 6, 20);

LocalDate parsed = LocalDate.parse("2024-01-15");

  

today.plusDays(10); // 2024-01-25

today.minusMonths(1); // 2023-12-15

today.getDayOfWeek(); // MONDAY

today.isLeapYear(); // false

today.isBefore(birthday); // false

  

LocalTime now = LocalTime.of(10, 30, 45);

LocalDateTime ldt = LocalDateTime.of(today, now);

ldt.toLocalDate(); // LocalDate

ldt.toLocalTime(); // LocalTime

```

  

---

  

### ZonedDateTime & Instant `[tip]`

  

```java

ZonedDateTime zdt = ZonedDateTime.now(ZoneId.of("Asia/Kolkata"));

zdt.getZone(); // Asia/Kolkata

  

// Convert between zones

ZonedDateTime utc = zdt.withZoneSameInstant(ZoneId.of("UTC"));

  

// Instant — machine time

Instant now = Instant.now();

Instant later = now.plusSeconds(3600);

Duration diff = Duration.between(now, later); // PT1H

```

  

---

  

### DateTimeFormatter `[tip]`

  

```java

DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd-MMM-yyyy");

  

LocalDate date = LocalDate.of(2024, 1, 15);

String formatted = date.format(fmt); // "15-Jan-2024"

  

LocalDate parsed = LocalDate.parse("15-Jan-2024", fmt); // back to date

  

// ISO formatters — built-in

DateTimeFormatter.ISO_LOCAL_DATE.format(date); // "2024-01-15"

```

  

---

  

### Period & Duration `[tip]`

  

```java

LocalDate start = LocalDate.of(2020, 1, 1);

LocalDate end = LocalDate.of(2024, 6, 15);

  

Period p = Period.between(start, end);

p.getYears(); // 4

p.getMonths(); // 5

p.getDays(); // 14

  

LocalTime t1 = LocalTime.of(9, 0);

LocalTime t2 = LocalTime.of(17, 30);

Duration d = Duration.between(t1, t2);

d.toHours(); // 8

d.toMinutes(); // 510

```

  

---

  

## 8. Collectors

  

### Common Collectors `[core]`

  

```java

List<Employee> employees = getEmployees();

  

// toList, toSet, toMap

List<String> names = employees.stream().map(Employee::getName).collect(Collectors.toList());

Set<String> depts = employees.stream().map(Employee::getDept).collect(Collectors.toSet());

Map<Integer, String> byId = employees.stream()

.collect(Collectors.toMap(Employee::getId, Employee::getName));

  

// joining

String csv = employees.stream()

.map(Employee::getName)

.collect(Collectors.joining(", ", "[", "]"));

// "[Alice, Bob, Charlie]"

  

// counting

long count = employees.stream().collect(Collectors.counting());

  

// summarizing

IntSummaryStatistics stats = employees.stream()

.collect(Collectors.summarizingInt(Employee::getSalary));

stats.getMax(); stats.getMin(); stats.getAverage(); stats.getSum();

```

  

---

  

### groupingBy & partitioningBy `[core]`

  

```java

// groupingBy — group into Map<K, List<V>>

Map<String, List<Employee>> byDept =

employees.stream().collect(Collectors.groupingBy(Employee::getDept));

  

// groupingBy with downstream collector

Map<String, Long> countByDept =

employees.stream().collect(Collectors.groupingBy(Employee::getDept, Collectors.counting()));

  

Map<String, Double> avgSalByDept =

employees.stream().collect(Collectors.groupingBy(Employee::getDept,

Collectors.averagingInt(Employee::getSalary)));

  

// partitioningBy — splits into true/false map

Map<Boolean, List<Employee>> partition =

employees.stream().collect(Collectors.partitioningBy(e -> e.getSalary() > 50000));

  

partition.get(true); // high earners

partition.get(false); // others

```

  

---

  

### toUnmodifiableList / toUnmodifiableMap `[tip]`

Available since Java 10 via `Collectors`, also via `List.copyOf()`.

  

```java

List<String> immutable = employees.stream()

.map(Employee::getName)

.collect(Collectors.toUnmodifiableList());

  

// immutable.add("x"); // throws UnsupportedOperationException

```

  

---

  

## 9. Map Enhancements

  

### New Map Methods (Java 8) `[core]`

  

```java

Map<String, Integer> scores = new HashMap<>();

scores.put("Alice", 90);

  

// getOrDefault — no NPE

scores.getOrDefault("Bob", 0); // 0

  

// putIfAbsent — only inserts if key absent

scores.putIfAbsent("Alice", 100); // no-op — already exists

scores.putIfAbsent("Bob", 75); // inserts

  

// computeIfAbsent — compute & insert if absent

scores.computeIfAbsent("Charlie", k -> k.length() * 10); // 70

  

// computeIfPresent — update only if key exists

scores.computeIfPresent("Alice", (k, v) -> v + 5); // 95

  

// compute — always compute

scores.compute("Alice", (k, v) -> v == null ? 1 : v + 1); // 96

  

// merge — merge value with existing

scores.merge("Alice", 10, Integer::sum); // 106 (96 + 10)

scores.merge("Dave", 50, Integer::sum); // 50 (new entry)

  

// forEach

scores.forEach((k, v) -> System.out.println(k + " → " + v));

  

// replaceAll

scores.replaceAll((k, v) -> v * 2);

```

  

---

  

## 10. Nashorn JavaScript Engine

  

### Embed & Execute JS `[tip]`

Nashorn replaced Rhino. Allows running JavaScript from Java. **Deprecated in Java 11, removed in Java 15** — use GraalVM for modern projects.

  

```java

ScriptEngine engine = new ScriptEngineManager().getEngineByName("nashorn");

  

// Execute JS

engine.eval("print('Hello from JS!')");

  

// Pass Java vars to JS

engine.put("name", "Omkar");

engine.eval("print('Hello ' + name)"); // "Hello Omkar"

  

// Get result back

Object result = engine.eval("2 + 2");

System.out.println(result); // 4.0

```

  

---

  

## 11. Base64 Encoding/Decoding

  

### java.util.Base64 `[tip]`

Finally a standard API — no need for Apache Commons or Sun's internal classes.

  

```java

// Encode

String original = "Omkar:password123";

String encoded = Base64.getEncoder().encodeToString(original.getBytes());

// "T21rYXI6cGFzc3dvcmQxMjM="

  

// Decode

byte[] decoded = Base64.getDecoder().decode(encoded);

String back = new String(decoded); // "Omkar:password123"

  

// URL-safe encoder (replaces +/ with -_)

String urlSafe = Base64.getUrlEncoder().encodeToString(original.getBytes());

  

// MIME encoder (line breaks every 76 chars)

String mime = Base64.getMimeEncoder().encodeToString(original.getBytes());

```

  

---

  

## 12. Quick Reference Cheat Sheet

  

|Feature|Key Points|
|---|---|
|Lambda|`(params) -> body` · captured variables must be effectively final|
|Functional Interface|SAM (Single Abstract Method) · `@FunctionalInterface` · examples: `Function`, `Predicate`, `Consumer`, `Supplier`|
|Method Reference|4 types: static method, instance method on object, instance method on type, constructor reference|
|Stream|Lazy pipeline · single-use · common flow: `filter → map → collect`|
|Parallel Stream|Uses `ForkJoinPool` · avoid for small datasets, ordered processing, or I/O-heavy tasks|
|Optional|Avoid as field/parameter · prefer `orElse`, `map`, `ifPresent` instead of `get()`|
|Default Method|Multiple inheritance can create diamond problem → implementation class must override|
|`LocalDate` / `LocalDateTime`|Immutable · thread-safe · replacement for legacy `Date` and `Calendar`|
|`groupingBy`|Groups elements into `Map<K, List<V>>` with optional downstream collectors|
|`partitioningBy`|Splits data into `Map<Boolean, List<V>>` based on condition|
|Map `merge()`|Insert or update value using `BiFunction` in a single call|
|Base64|Built-in encoder/decoder supports standard, URL-safe, and MIME variants|
  

---

  

### Stream Operation Summary

  

| Operation            | Type         | Returns                         |
| -------------------- | ------------ | ------------------------------- |
| `filter()`           | Intermediate | `Stream<T>`                     |
| `map()`              | Intermediate | `Stream<R>`                     |
| `flatMap()`          | Intermediate | `Stream<R>`                     |
| `sorted()`           | Intermediate | `Stream<T>`                     |
| `distinct()`         | Intermediate | `Stream<T>`                     |
| `limit()` / `skip()` | Intermediate | `Stream<T>`                     |
| `collect()`          | Terminal     | Collection / `Collector` result |
| `reduce()`           | Terminal     | `Optional<T>` / `T`             |
| `count()`            | Terminal     | `long`                          |
| `forEach()`          | Terminal     | `void`                          |
| `anyMatch()`         | Terminal     | `boolean`                       |
| `findFirst()`        | Terminal     | `Optional<T>`                   |

  

---

  

*Next topics: Spring Boot · Spring Batch · Concurrency · JVM Internals · Design Patterns*