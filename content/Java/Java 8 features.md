## 📋 Table of Contents

- [[#1. Lambda Expressions|1. Lambda Expressions]]
	- [[#1. Lambda Expressions#Syntax|Syntax]]
	- [[#1. Lambda Expressions#Examples|Examples]]
	- [[#1. Lambda Expressions#Key Points|Key Points]]
- [[#2. Functional Interfaces|2. Functional Interfaces]]
	- [[#2. Functional Interfaces#Built-in Functional Interfaces (java.util.function)|Built-in Functional Interfaces (java.util.function)]]
	- [[#2. Functional Interfaces#2.1 Predicate\<T>|2.1 Predicate\<T>]]
	- [[#2. Functional Interfaces#2.2 Function\<T, R>|2.2 Function\<T, R>]]
	- [[#2. Functional Interfaces#2.3 Consumer\<T>|2.3 Consumer\<T>]]
	- [[#2. Functional Interfaces#2.4 Supplier\<T>|2.4 Supplier\<T>]]
- [[#3. Stream API|3. Stream API]]
	- [[#3. Stream API#Stream Lifecycle|Stream Lifecycle]]
	- [[#3. Stream API#Key Stream Methods|Key Stream Methods]]
		- [[#Key Stream Methods#filter()|filter()]]
		- [[#Key Stream Methods#map()|map()]]
		- [[#Key Stream Methods#collect()|collect()]]
		- [[#Key Stream Methods#sorted()|sorted()]]
		- [[#Key Stream Methods#reduce()|reduce()]]
		- [[#Key Stream Methods#distinct()|distinct()]]
		- [[#Key Stream Methods#limit() and skip()|limit() and skip()]]
		- [[#Key Stream Methods#count()|count()]]
		- [[#Key Stream Methods#flatMap()|flatMap()]]
	- [[#3. Stream API#Stream Method Summary Table|Stream Method Summary Table]]
- [[#4. Method References|4. Method References]]
	- [[#4. Method References#4.1 Static Method Reference|4.1 Static Method Reference]]
	- [[#4. Method References#4.2 Instance Method Reference|4.2 Instance Method Reference]]
	- [[#4. Method References#4.3 Constructor Reference|4.3 Constructor Reference]]
- [[#5. Optional Class|5. Optional Class]]
	- [[#5. Optional Class#Creating Optional|Creating Optional]]
	- [[#5. Optional Class#Accessing Values|Accessing Values]]
	- [[#5. Optional Class#Optional Methods Summary|Optional Methods Summary]]
	- [[#5. Optional Class#Best Practices|Best Practices]]
- [[#6. Default & Static Methods in Interface|6. Default & Static Methods in Interface]]
	- [[#6. Default & Static Methods in Interface#Default Methods|Default Methods]]
	- [[#6. Default & Static Methods in Interface#Override Default Method|Override Default Method]]
	- [[#6. Default & Static Methods in Interface#Multiple Inheritance Conflict|Multiple Inheritance Conflict]]
	- [[#6. Default & Static Methods in Interface#Static Methods in Interface|Static Methods in Interface]]
	- [[#6. Default & Static Methods in Interface#Key Differences|Key Differences]]
- [[#7. Date and Time API|7. Date and Time API]]
	- [[#7. Date and Time API#Key Classes|Key Classes]]
	- [[#7. Date and Time API#7.1 LocalDate|7.1 LocalDate]]
	- [[#7. Date and Time API#7.2 LocalTime|7.2 LocalTime]]
	- [[#7. Date and Time API#7.3 LocalDateTime|7.3 LocalDateTime]]
	- [[#7. Date and Time API#7.4 DateTimeFormatter|7.4 DateTimeFormatter]]
	- [[#7. Date and Time API#Common Patterns|Common Patterns]]
- [[#8. Collectors API|8. Collectors API]]
	- [[#8. Collectors API#groupingBy|groupingBy]]
	- [[#8. Collectors API#partitioningBy|partitioningBy]]
	- [[#8. Collectors API#joining|joining]]
	- [[#8. Collectors API#counting|counting]]
	- [[#8. Collectors API#summarizingInt / averagingInt|summarizingInt / averagingInt]]
	- [[#8. Collectors API#toMap|toMap]]
- [[#9. Parallel Streams|9. Parallel Streams]]
	- [[#9. Parallel Streams#When to Use / Avoid|When to Use / Avoid]]
- [[#10. CompletableFuture Basics|10. CompletableFuture Basics]]
	- [[#10. CompletableFuture Basics#Basic Usage|Basic Usage]]
	- [[#10. CompletableFuture Basics#Chaining|Chaining]]
	- [[#10. CompletableFuture Basics#Combining Futures|Combining Futures]]
	- [[#10. CompletableFuture Basics#Error Handling|Error Handling]]
	- [[#10. CompletableFuture Basics#CompletableFuture vs Future|CompletableFuture vs Future]]
- [[#11. Java 8 Interview Quick Points|11. Java 8 Interview Quick Points]]
	- [[#11. Java 8 Interview Quick Points#🔴 Must-Know Points|🔴 Must-Know Points]]
	- [[#11. Java 8 Interview Quick Points#🟡 Commonly Confused|🟡 Commonly Confused]]
	- [[#11. Java 8 Interview Quick Points#🟢 Trick Questions|🟢 Trick Questions]]
- [[#12. Common Differences|12. Common Differences]]
	- [[#12. Common Differences#map() vs flatMap()|map() vs flatMap()]]
	- [[#12. Common Differences#Collection vs Stream|Collection vs Stream]]
	- [[#12. Common Differences#findFirst() vs findAny()|findFirst() vs findAny()]]
	- [[#12. Common Differences#Predicate vs Function|Predicate vs Function]]
	- [[#12. Common Differences#Comparable vs Comparator|Comparable vs Comparator]]
- [[#🧪 Quick Practice MCQs|🧪 Quick Practice MCQs]]
- [[#📝 Mini Practice Assignments|📝 Mini Practice Assignments]]
- [[#🎯 Revision Cheat Sheet|🎯 Revision Cheat Sheet]]




---

## 1. Lambda Expressions

> **One-liner:** Anonymous function — no name, no class, no boilerplate.

### Syntax

```java
// No param
() -> expression

// One param (brackets optional)
name -> System.out.println(name)

// Multiple params
(a, b) -> a + b

// Multi-line body
(a, b) -> {
    int sum = a + b;
    return sum;
}
```

### Examples

```java
// Before Java 8 (anonymous class)
Runnable r1 = new Runnable() {
    public void run() { System.out.println("Old way"); }
};

// Java 8 Lambda
Runnable r2 = () -> System.out.println("Lambda way");

// With list
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");
names.forEach(name -> System.out.println(name));
// Output: Alice  Bob  Charlie

// Sorting with Lambda
List<Integer> nums = Arrays.asList(5, 2, 8, 1);
nums.sort((a, b) -> a - b);
// Output: [1, 2, 5, 8]
```

### Key Points

- ✅ Reduces boilerplate code
- ✅ Enables functional programming
- ✅ Can access `effectively final` local variables
- ✅ Used heavily with Stream API and functional interfaces
- ❌ Cannot modify local variables from enclosing scope
- ❌ Cannot use `this` to refer to lambda itself

> 💡 **Interview Tip:** Lambda is basically an implementation of a **functional interface**.

---

## 2. Functional Interfaces

> **One-liner:** An interface with **exactly one abstract method** (SAM — Single Abstract Method).

```java
@FunctionalInterface
interface MyFunc {
    int compute(int a, int b);
    // Can have default/static methods — still functional!
}

MyFunc add = (a, b) -> a + b;
System.out.println(add.compute(3, 4)); // Output: 7
```

### Built-in Functional Interfaces (java.util.function)

| Interface | Method | Input | Output | Use Case |
|-----------|--------|-------|--------|----------|
| `Predicate<T>` | `test(T t)` | T | boolean | Filter/condition check |
| `Function<T,R>` | `apply(T t)` | T | R | Transform/map a value |
| `Consumer<T>` | `accept(T t)` | T | void | Perform action, no return |
| `Supplier<T>` | `get()` | none | T | Provide/generate a value |
| `BiFunction<T,U,R>` | `apply(T,U)` | T, U | R | Two-input transform |
| `UnaryOperator<T>` | `apply(T t)` | T | T | Transform same type |
| `BinaryOperator<T>` | `apply(T,T)` | T, T | T | Combine same types |

---

### 2.1 Predicate\<T>

```java
Predicate<Integer> isEven = n -> n % 2 == 0;

System.out.println(isEven.test(4));   // true
System.out.println(isEven.test(7));   // false

// Chaining
Predicate<Integer> isPositive = n -> n > 0;
Predicate<Integer> isEvenAndPositive = isEven.and(isPositive);
System.out.println(isEvenAndPositive.test(6));  // true
System.out.println(isEvenAndPositive.test(-4)); // false

// negate
Predicate<Integer> isOdd = isEven.negate();
System.out.println(isOdd.test(3)); // true
```

**Methods:** `test()`, `and()`, `or()`, `negate()`, `isEqual()`

---

### 2.2 Function\<T, R>

```java
Function<String, Integer> strLen = s -> s.length();
System.out.println(strLen.apply("Hello")); // 5

// andThen (chaining)
Function<Integer, Integer> doubleIt = n -> n * 2;
Function<String, Integer> lenThenDouble = strLen.andThen(doubleIt);
System.out.println(lenThenDouble.apply("Hello")); // 10

// compose (reverse order)
Function<Integer, Integer> addOne = n -> n + 1;
Function<Integer, Integer> doubleFirst = doubleIt.compose(addOne); // addOne first, then double
System.out.println(doubleFirst.apply(3)); // (3+1)*2 = 8
```

**Methods:** `apply()`, `andThen()`, `compose()`, `identity()`

---

### 2.3 Consumer\<T>

```java
Consumer<String> print = s -> System.out.println("Name: " + s);
print.accept("Omkar"); // Output: Name: Omkar

// andThen chaining
Consumer<String> shout = s -> System.out.println(s.toUpperCase());
Consumer<String> printThenShout = print.andThen(shout);
printThenShout.accept("omkar");
// Output:
// Name: omkar
// OMKAR
```

**Methods:** `accept()`, `andThen()`

---

### 2.4 Supplier\<T>

```java
Supplier<String> greeting = () -> "Hello, World!";
System.out.println(greeting.get()); // Hello, World!

Supplier<LocalDate> today = LocalDate::now;
System.out.println(today.get()); // 2024-01-15
```

**Methods:** `get()`

> 💡 **Interview Tip:** `Predicate` returns boolean. `Function` transforms. `Consumer` consumes. `Supplier` supplies — no input.

---

## 3. Stream API

> **One-liner:** A pipeline for processing collections — filter, transform, collect — without modifying the source.

### Stream Lifecycle

```
Source → Intermediate Operations (lazy) → Terminal Operation (triggers execution)
```

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave", "Anna");

List<String> result = names.stream()           // 1. Source
    .filter(n -> n.startsWith("A"))            // 2. Intermediate (lazy)
    .map(String::toUpperCase)                  // 3. Intermediate (lazy)
    .sorted()                                  // 4. Intermediate (lazy)
    .collect(Collectors.toList());             // 5. Terminal (executes pipeline)

System.out.println(result); // [ALICE, ANNA]
```

### Key Stream Methods

---

#### filter()

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8);

List<Integer> evens = nums.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

System.out.println(evens); // [2, 4, 6, 8]
```

---

#### map()

```java
List<String> names = Arrays.asList("alice", "bob", "charlie");

List<String> upper = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());

System.out.println(upper); // [ALICE, BOB, CHARLIE]

// Map to length
List<Integer> lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList());
System.out.println(lengths); // [5, 3, 7]
```

---

#### collect()

```java
// Collect to List
List<String> list = stream.collect(Collectors.toList());

// Collect to Set (removes duplicates)
Set<String> set = stream.collect(Collectors.toSet());

// Collect to Map
Map<String, Integer> map = names.stream()
    .collect(Collectors.toMap(n -> n, String::length));
// {"alice"=5, "bob"=3, "charlie"=7}
```

---

#### sorted()

```java
List<Integer> nums = Arrays.asList(5, 2, 8, 1, 9, 3);

// Natural sort
List<Integer> sorted = nums.stream()
    .sorted()
    .collect(Collectors.toList());
System.out.println(sorted); // [1, 2, 3, 5, 8, 9]

// Reverse sort
List<Integer> desc = nums.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());
System.out.println(desc); // [9, 8, 5, 3, 2, 1]

// Sort by custom field
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
names.stream()
    .sorted(Comparator.comparing(String::length))
    .forEach(System.out::println);
// Output: Bob  Alice  Charlie
```

---

#### reduce()

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);

// Sum
int sum = nums.stream()
    .reduce(0, (a, b) -> a + b);
System.out.println(sum); // 15

// Or use method reference
int sum2 = nums.stream().reduce(0, Integer::sum);

// Max
Optional<Integer> max = nums.stream()
    .reduce(Integer::max);
System.out.println(max.get()); // 5
```

---

#### distinct()

```java
List<Integer> nums = Arrays.asList(1, 2, 2, 3, 3, 3, 4);

List<Integer> unique = nums.stream()
    .distinct()
    .collect(Collectors.toList());
System.out.println(unique); // [1, 2, 3, 4]
```

---

#### limit() and skip()

```java
List<Integer> nums = Arrays.asList(1,2,3,4,5,6,7,8,9,10);

// First 5
nums.stream().limit(5).forEach(System.out::print);
// Output: 1 2 3 4 5

// Skip first 5
nums.stream().skip(5).forEach(System.out::print);
// Output: 6 7 8 9 10

// Pagination: page 2, size 3
nums.stream().skip(3).limit(3).forEach(System.out::print);
// Output: 4 5 6
```

---

#### count()

```java
long count = Arrays.asList(1, 2, 3, 4, 5).stream()
    .filter(n -> n > 3)
    .count();
System.out.println(count); // 2
```

---

#### flatMap()

```java
// map → gives Stream<Stream<T>>
// flatMap → flattens to Stream<T>

List<List<Integer>> nested = Arrays.asList(
    Arrays.asList(1, 2, 3),
    Arrays.asList(4, 5, 6),
    Arrays.asList(7, 8, 9)
);

List<Integer> flat = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList());
System.out.println(flat); // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// Real-world: words from sentences
List<String> sentences = Arrays.asList("Hello World", "Java 8 Rocks");
List<String> words = sentences.stream()
    .flatMap(s -> Arrays.stream(s.split(" ")))
    .collect(Collectors.toList());
System.out.println(words); // [Hello, World, Java, 8, Rocks]
```

### Stream Method Summary Table

| Method | Type | Returns | Description |
|--------|------|---------|-------------|
| `filter()` | Intermediate | `Stream<T>` | Keep elements matching predicate |
| `map()` | Intermediate | `Stream<R>` | Transform each element |
| `flatMap()` | Intermediate | `Stream<R>` | Flatten nested streams |
| `sorted()` | Intermediate | `Stream<T>` | Sort elements |
| `distinct()` | Intermediate | `Stream<T>` | Remove duplicates |
| `limit(n)` | Intermediate | `Stream<T>` | Keep first n elements |
| `skip(n)` | Intermediate | `Stream<T>` | Skip first n elements |
| `peek()` | Intermediate | `Stream<T>` | Debug/inspect without consuming |
| `collect()` | Terminal | `R` | Collect into collection |
| `forEach()` | Terminal | void | Iterate each element |
| `reduce()` | Terminal | `Optional<T>` | Combine elements to one |
| `count()` | Terminal | long | Count elements |
| `findFirst()` | Terminal | `Optional<T>` | First element |
| `findAny()` | Terminal | `Optional<T>` | Any element (better for parallel) |
| `anyMatch()` | Terminal | boolean | Any element matches predicate |
| `allMatch()` | Terminal | boolean | All elements match predicate |
| `noneMatch()` | Terminal | boolean | No elements match predicate |
| `min()` / `max()` | Terminal | `Optional<T>` | Min/max element |

> 💡 **Interview Tip:** Streams are **lazy** — intermediate operations don't execute until a terminal operation is called. Streams can be used **only once** — reuse requires creating a new stream.

---

## 4. Method References

> **One-liner:** Shorthand for a lambda that calls an existing method. Uses `::` operator.

| Type | Syntax | Lambda Equivalent |
|------|--------|-------------------|
| Static method | `ClassName::staticMethod` | `x -> ClassName.staticMethod(x)` |
| Instance method (object) | `object::instanceMethod` | `x -> object.instanceMethod(x)` |
| Instance method (type) | `ClassName::instanceMethod` | `x -> x.instanceMethod()` |
| Constructor | `ClassName::new` | `x -> new ClassName(x)` |

---

### 4.1 Static Method Reference

```java
// Lambda
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
names.stream().map(s -> s.toUpperCase()); // lambda

// Static method reference
Function<String, String> toUpper = String::toUpperCase;

// Integer.parseInt
List<String> strNums = Arrays.asList("1", "2", "3");
List<Integer> ints = strNums.stream()
    .map(Integer::parseInt)
    .collect(Collectors.toList());
System.out.println(ints); // [1, 2, 3]
```

---

### 4.2 Instance Method Reference

```java
// On specific instance
String prefix = "Hello, ";
Function<String, String> greet = prefix::concat;
System.out.println(greet.apply("World")); // Hello, World

// On arbitrary instance (of a type)
List<String> names = Arrays.asList("Charlie", "Alice", "Bob");
names.stream()
    .map(String::toLowerCase)  // each element's own method
    .forEach(System.out::println);
// Output: charlie  alice  bob
```

---

### 4.3 Constructor Reference

```java
// Without constructor reference
Function<String, StringBuilder> f1 = s -> new StringBuilder(s);

// With constructor reference
Function<String, StringBuilder> f2 = StringBuilder::new;
StringBuilder sb = f2.apply("Hello");
System.out.println(sb); // Hello

// With list of names
List<String> names = Arrays.asList("Alice", "Bob");
List<StringBuilder> builders = names.stream()
    .map(StringBuilder::new)
    .collect(Collectors.toList());
```

> 💡 **Interview Tip:** Method references improve readability. Use them when the lambda body is just calling an existing method directly.

---

## 5. Optional Class

> **One-liner:** A container that may or may not contain a non-null value. Eliminates `NullPointerException`.

### Creating Optional

```java
// of() — value must NOT be null (throws NullPointerException if null)
Optional<String> opt1 = Optional.of("Hello");

// ofNullable() — value CAN be null
Optional<String> opt2 = Optional.ofNullable(null);

// empty() — explicitly empty
Optional<String> opt3 = Optional.empty();
```

### Accessing Values

```java
Optional<String> opt = Optional.of("Java 8");

// isPresent() — check before getting
if (opt.isPresent()) {
    System.out.println(opt.get()); // Java 8
}

// ifPresent() — run action if value exists
opt.ifPresent(v -> System.out.println("Value: " + v)); // Value: Java 8

// orElse() — default if empty
String val = Optional.<String>empty().orElse("Default");
System.out.println(val); // Default

// orElseGet() — supply default lazily
String val2 = Optional.<String>empty().orElseGet(() -> "Computed Default");

// orElseThrow() — throw if empty
String val3 = opt.orElseThrow(() -> new RuntimeException("Not found"));

// map() — transform value if present
Optional<Integer> len = opt.map(String::length);
System.out.println(len.get()); // 6

// filter() — keep value only if condition met
Optional<String> filtered = opt.filter(s -> s.length() > 3);
System.out.println(filtered.isPresent()); // true
```

### Optional Methods Summary

| Method | Returns | Description |
|--------|---------|-------------|
| `of(value)` | `Optional<T>` | Wrap non-null value |
| `ofNullable(value)` | `Optional<T>` | Wrap nullable value |
| `empty()` | `Optional<T>` | Empty Optional |
| `get()` | T | Get value (throws if empty) |
| `isPresent()` | boolean | Check if value exists |
| `isEmpty()` | boolean | Check if empty (Java 11+) |
| `ifPresent(action)` | void | Run action if present |
| `orElse(other)` | T | Return value or default |
| `orElseGet(supplier)` | T | Return value or supplied default |
| `orElseThrow(supplier)` | T | Return value or throw |
| `map(mapper)` | `Optional<U>` | Transform value |
| `flatMap(mapper)` | `Optional<U>` | Transform (mapper returns Optional) |
| `filter(predicate)` | `Optional<T>` | Keep value if condition met |

### Best Practices

```java
// ✅ Good — use orElse instead of get()
String name = optional.orElse("Unknown");

// ❌ Bad — always check isPresent() before get()
String name = optional.get(); // NoSuchElementException if empty!

// ✅ Good — chain operations
Optional.ofNullable(user)
    .map(User::getAddress)
    .map(Address::getCity)
    .orElse("City not found");

// ❌ Bad — don't use Optional as method parameter
public void process(Optional<String> name) { ... } // Don't do this

// ✅ Good — use Optional as return type
public Optional<User> findById(int id) { ... }
```

> 💡 **Interview Tip:** `orElse()` always evaluates the default (even if value exists). `orElseGet()` is lazy — only evaluates if needed. Prefer `orElseGet()` for expensive operations.

---

## 6. Default & Static Methods in Interface

> **Why introduced?** To add new methods to interfaces without breaking existing implementations.

### Default Methods

```java
interface Vehicle {
    String getName();

    // Default method — concrete implementation in interface
    default String getInfo() {
        return "Vehicle: " + getName();
    }

    default void start() {
        System.out.println("Vehicle starting...");
    }
}

class Car implements Vehicle {
    public String getName() { return "Car"; }
    // getInfo() and start() are inherited — no need to override
}

// Usage
Car car = new Car();
System.out.println(car.getInfo()); // Vehicle: Car
car.start();                       // Vehicle starting...
```

### Override Default Method

```java
class ElectricCar implements Vehicle {
    public String getName() { return "Tesla"; }

    @Override
    public void start() {
        System.out.println("Electric car starting silently...");
    }
}
```

### Multiple Inheritance Conflict

```java
interface A {
    default void hello() { System.out.println("Hello from A"); }
}
interface B {
    default void hello() { System.out.println("Hello from B"); }
}

// ⚠️ Compiler error: class must override the conflicting method
class C implements A, B {
    @Override
    public void hello() {
        A.super.hello(); // Explicitly choose which one
    }
}
```

### Static Methods in Interface

```java
interface MathUtil {
    static int add(int a, int b) { return a + b; }
    static int square(int n) { return n * n; }
}

// Called on interface name (NOT on object)
System.out.println(MathUtil.add(3, 4));    // 7
System.out.println(MathUtil.square(5));    // 25
```

### Key Differences

| Feature | Default Method | Static Method |
|---------|----------------|---------------|
| Inherited? | ✅ Yes | ❌ No |
| Overridable? | ✅ Yes | ❌ No |
| Called on | Instance | Interface name |
| Purpose | Add behavior | Utility methods |

> 💡 **Interview Tip:** `static` methods in interfaces are NOT inherited by implementing classes or sub-interfaces.

---

## 7. Date and Time API

> **Why introduced?** `java.util.Date` and `Calendar` were mutable, thread-unsafe, and confusing. `java.time` is immutable and thread-safe.

### Key Classes

| Class | Description | Example |
|-------|-------------|---------|
| `LocalDate` | Date only (no time, no timezone) | 2024-01-15 |
| `LocalTime` | Time only (no date, no timezone) | 10:30:45 |
| `LocalDateTime` | Date + time (no timezone) | 2024-01-15T10:30:45 |
| `ZonedDateTime` | Date + time + timezone | 2024-01-15T10:30:45+05:30[Asia/Kolkata] |
| `Instant` | Timestamp (UTC epoch) | 2024-01-15T05:00:45Z |
| `Duration` | Time-based amount | PT8H30M |
| `Period` | Date-based amount | P1Y2M15D |

---

### 7.1 LocalDate

```java
// Create
LocalDate today = LocalDate.now();               // 2024-01-15
LocalDate dob = LocalDate.of(1995, 8, 15);       // 1995-08-15
LocalDate parsed = LocalDate.parse("2024-01-15");

// Operations (all return new instances — immutable!)
LocalDate tomorrow = today.plusDays(1);
LocalDate nextMonth = today.plusMonths(1);
LocalDate lastYear = today.minusYears(1);

// Info
System.out.println(today.getDayOfWeek());  // MONDAY
System.out.println(today.getDayOfMonth()); // 15
System.out.println(today.getMonthValue()); // 1
System.out.println(today.getYear());       // 2024
System.out.println(today.isLeapYear());    // false

// Compare
System.out.println(dob.isBefore(today));   // true
System.out.println(dob.isAfter(today));    // false

// Period between dates
Period age = Period.between(dob, today);
System.out.println("Age: " + age.getYears() + " years"); // Age: 28 years
```

---

### 7.2 LocalTime

```java
LocalTime now = LocalTime.now();          // 10:30:45.123
LocalTime time = LocalTime.of(10, 30, 0); // 10:30

System.out.println(time.getHour());   // 10
System.out.println(time.getMinute()); // 30

LocalTime later = time.plusHours(2).plusMinutes(30);
System.out.println(later); // 13:00

// Duration between times
Duration duration = Duration.between(time, later);
System.out.println(duration.toMinutes()); // 150
```

---

### 7.3 LocalDateTime

```java
LocalDateTime now = LocalDateTime.now();
LocalDateTime meeting = LocalDateTime.of(2024, 1, 15, 14, 30);

System.out.println(meeting); // 2024-01-15T14:30

// Extract parts
LocalDate date = meeting.toLocalDate(); // 2024-01-15
LocalTime time = meeting.toLocalTime(); // 14:30

// Add/subtract
LocalDateTime nextWeek = meeting.plusWeeks(1);
System.out.println(nextWeek); // 2024-01-22T14:30
```

---

### 7.4 DateTimeFormatter

```java
// Formatting
LocalDateTime now = LocalDateTime.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("dd-MM-yyyy HH:mm:ss");
String formatted = now.format(formatter);
System.out.println(formatted); // 15-01-2024 10:30:45

// Parsing
String dateStr = "15-01-2024";
DateTimeFormatter fmt = DateTimeFormatter.ofPattern("dd-MM-yyyy");
LocalDate date = LocalDate.parse(dateStr, fmt);
System.out.println(date); // 2024-01-15

// Common built-in formats
DateTimeFormatter iso = DateTimeFormatter.ISO_LOCAL_DATE;
System.out.println(LocalDate.now().format(iso)); // 2024-01-15
```

### Common Patterns

| Symbol | Meaning | Example |
|--------|---------|---------|
| `yyyy` | 4-digit year | 2024 |
| `MM` | 2-digit month | 01 |
| `dd` | 2-digit day | 15 |
| `HH` | Hour (24h) | 14 |
| `hh` | Hour (12h) | 02 |
| `mm` | Minutes | 30 |
| `ss` | Seconds | 45 |
| `a` | AM/PM | PM |
| `EEE` | Short day name | Mon |
| `EEEE` | Full day name | Monday |

> 💡 **Interview Tip:** All `java.time` classes are **immutable** and **thread-safe**. Operations return new instances — originals are never modified.

---

## 8. Collectors API

> **One-liner:** Terminal operations for collecting stream results into collections, strings, maps, or statistics.

```java
import java.util.stream.Collectors;
```

---

### groupingBy

```java
List<String> words = Arrays.asList("Hi", "Hello", "Hey", "World", "Wow", "Java");

// Group by first letter
Map<Character, List<String>> grouped = words.stream()
    .collect(Collectors.groupingBy(w -> w.charAt(0)));
System.out.println(grouped);
// {H=[Hi, Hello, Hey], W=[World, Wow], J=[Java]}

// Group by length, count each group
Map<Integer, Long> byLength = words.stream()
    .collect(Collectors.groupingBy(String::length, Collectors.counting()));
System.out.println(byLength);
// {2=1, 5=2, 3=2, 4=1}
```

---

### partitioningBy

```java
// Splits into exactly 2 groups: true and false
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

Map<Boolean, List<Integer>> partitioned = nums.stream()
    .collect(Collectors.partitioningBy(n -> n % 2 == 0));

System.out.println(partitioned.get(true));  // [2, 4, 6, 8, 10]
System.out.println(partitioned.get(false)); // [1, 3, 5, 7, 9]
```

---

### joining

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Dave");

// Simple join
String simple = names.stream().collect(Collectors.joining());
System.out.println(simple); // AliceBobCharlieDave

// With delimiter
String csv = names.stream().collect(Collectors.joining(", "));
System.out.println(csv); // Alice, Bob, Charlie, Dave

// With delimiter, prefix, suffix
String formatted = names.stream()
    .collect(Collectors.joining(", ", "[", "]"));
System.out.println(formatted); // [Alice, Bob, Charlie, Dave]
```

---

### counting

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie", "Anna", "Brian");

// Count total
long total = names.stream().collect(Collectors.counting());
System.out.println(total); // 5

// Count per group
Map<Character, Long> countByLetter = names.stream()
    .collect(Collectors.groupingBy(n -> n.charAt(0), Collectors.counting()));
System.out.println(countByLetter); // {A=2, B=2, C=1}
```

---

### summarizingInt / averagingInt

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);

// Statistics
IntSummaryStatistics stats = nums.stream()
    .collect(Collectors.summarizingInt(Integer::intValue));
System.out.println(stats.getCount()); // 5
System.out.println(stats.getSum());   // 15
System.out.println(stats.getMin());   // 1
System.out.println(stats.getMax());   // 5
System.out.println(stats.getAverage()); // 3.0

// Average only
double avg = nums.stream().collect(Collectors.averagingInt(n -> n));
System.out.println(avg); // 3.0
```

---

### toMap

```java
List<String> names = Arrays.asList("Alice", "Bob", "Charlie");

Map<String, Integer> nameLengths = names.stream()
    .collect(Collectors.toMap(
        name -> name,        // key mapper
        String::length       // value mapper
    ));
System.out.println(nameLengths); // {Alice=5, Bob=3, Charlie=7}
```

> 💡 **Interview Tip:** `groupingBy` → multiple groups (Map\<K, List\<T>>). `partitioningBy` → exactly 2 groups (Map\<Boolean, List\<T>>).

---

## 9. Parallel Streams

> **One-liner:** Splits stream into sub-streams processed concurrently by multiple threads using ForkJoinPool.

```java
// Sequential stream
long seqStart = System.currentTimeMillis();
long seqSum = LongStream.rangeClosed(1, 100_000_000L)
    .sum();
System.out.println("Sequential: " + (System.currentTimeMillis() - seqStart) + "ms");

// Parallel stream
long parStart = System.currentTimeMillis();
long parSum = LongStream.rangeClosed(1, 100_000_000L)
    .parallel()
    .sum();
System.out.println("Parallel: " + (System.currentTimeMillis() - parStart) + "ms");
```

```java
// Convert collection stream to parallel
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

List<Integer> result = nums.parallelStream()
    .filter(n -> n % 2 == 0)
    .map(n -> n * n)
    .collect(Collectors.toList());
System.out.println(result); // [4, 16, 36, 64, 100] (order may vary!)
```

### When to Use / Avoid

| ✅ Use Parallel When | ❌ Avoid Parallel When |
|---------------------|----------------------|
| Large datasets (millions) | Small datasets |
| CPU-intensive operations | Simple/fast operations |
| Order doesn't matter | Order matters (`forEachOrdered`) |
| Independent operations | Operations with shared state |
| — | UI/single-threaded contexts |

> ⚠️ **Caution:** Parallel streams use common ForkJoinPool. Blocking operations can starve other tasks. Always benchmark before using — overhead can make it **slower** for small data.

---

## 10. CompletableFuture Basics

> **One-liner:** Java 8's way to write async, non-blocking code with chainable callbacks.

### Basic Usage

```java
// Run async task (fire and forget)
CompletableFuture<Void> cf = CompletableFuture.runAsync(() -> {
    System.out.println("Running in: " + Thread.currentThread().getName());
});
cf.join(); // wait

// Supply async result
CompletableFuture<String> future = CompletableFuture.supplyAsync(() -> {
    // simulate delay
    return "Hello from async!";
});
System.out.println(future.get()); // Hello from async!
```

### Chaining

```java
CompletableFuture<String> result = CompletableFuture
    .supplyAsync(() -> "hello")              // async: returns "hello"
    .thenApply(s -> s.toUpperCase())         // transform: "HELLO"
    .thenApply(s -> "Result: " + s);         // transform: "Result: HELLO"

System.out.println(result.get()); // Result: HELLO
```

### Combining Futures

```java
CompletableFuture<String> future1 = CompletableFuture.supplyAsync(() -> "Hello");
CompletableFuture<String> future2 = CompletableFuture.supplyAsync(() -> " World");

// Combine two futures when both complete
CompletableFuture<String> combined = future1.thenCombine(future2,
    (f1, f2) -> f1 + f2);
System.out.println(combined.get()); // Hello World

// Wait for ALL to complete
CompletableFuture<Void> allDone = CompletableFuture.allOf(future1, future2);
allDone.join();

// Whichever completes first
CompletableFuture<Object> firstDone = CompletableFuture.anyOf(future1, future2);
```

### Error Handling

```java
CompletableFuture<String> safeFuture = CompletableFuture
    .supplyAsync(() -> {
        if (true) throw new RuntimeException("Something broke!");
        return "OK";
    })
    .exceptionally(ex -> "Recovered: " + ex.getMessage())
    .handle((result, ex) -> {
        if (ex != null) return "Error handled";
        return result;
    });

System.out.println(safeFuture.get()); // Recovered: Something broke!
```

### CompletableFuture vs Future

| Feature | `Future` | `CompletableFuture` |
|---------|----------|---------------------|
| Chaining | ❌ | ✅ `thenApply`, `thenCompose` |
| Error handling | ❌ | ✅ `exceptionally`, `handle` |
| Combining | ❌ | ✅ `allOf`, `anyOf`, `thenCombine` |
| Manual completion | ❌ | ✅ `complete()` |
| Async callback | ❌ | ✅ `thenAcceptAsync` |

> 💡 **Interview Tip:** `thenApply` = synchronous transform (like `map`). `thenCompose` = async transform (like `flatMap`). `thenAccept` = consumes result (no return).

---

## 11. Java 8 Interview Quick Points

### 🔴 Must-Know Points

- **Lambda** = implementation of a functional interface (anonymous function)
- **Functional interface** = exactly 1 abstract method (`@FunctionalInterface`)
- **Stream** = does NOT store data; processes lazily; one-time use
- **Optional** = avoids `NullPointerException`; NOT a replacement for all nulls
- **Default methods** = added to interfaces to avoid breaking existing code
- **`java.time`** = immutable, thread-safe; replaces `java.util.Date`
- **Parallel stream** = uses `ForkJoinPool.commonPool()` — be careful with blocking ops
- **Method reference** = shorthand lambda, uses `::` operator

### 🟡 Commonly Confused

```
Predicate<T>    → boolean   (test condition)
Function<T,R>   → R         (transform T to R)
Consumer<T>     → void      (consume, no return)
Supplier<T>     → T         (supply, no input)

filter() → keeps elements matching predicate
map()    → transforms each element
flatMap()→ maps AND flattens nested structures

findFirst() → deterministic, always first in order
findAny()   → non-deterministic, optimized for parallel
```

### 🟢 Trick Questions

```java
// Q: How many times does this stream execute?
Stream<Integer> s = Stream.of(1, 2, 3).filter(n -> n > 1);
// Answer: ZERO times until a terminal operation is called!

// Q: Can you reuse a stream?
Stream<Integer> st = Stream.of(1, 2, 3);
st.forEach(System.out::println); // works
st.forEach(System.out::println); // ❌ IllegalStateException: stream has already been operated upon

// Q: What does Optional.of(null) do?
Optional.of(null); // ❌ NullPointerException — use ofNullable(null) instead
```

---

## 12. Common Differences

### map() vs flatMap()

| | `map()` | `flatMap()` |
|--|---------|-------------|
| **Input** | `Stream<T>` | `Stream<Stream<T>>` |
| **Output** | `Stream<R>` | `Stream<R>` (flattened) |
| **Use case** | Transform each element | Flatten nested collections |
| **Result shape** | Same nesting | One level flatter |

```java
// map → gives nested stream
Stream<String[]> mapped = Stream.of("Hello World", "Java 8")
    .map(s -> s.split(" "));
// Result: Stream<String[]>  ← nested

// flatMap → flattens
Stream<String> flat = Stream.of("Hello World", "Java 8")
    .flatMap(s -> Arrays.stream(s.split(" ")));
// Result: Stream<String>  ← flat
// Elements: Hello, World, Java, 8
```

---

### Collection vs Stream

| Feature | `Collection` | `Stream` |
|---------|-------------|----------|
| **Storage** | ✅ Stores data | ❌ No storage |
| **Reusable** | ✅ Multiple iterations | ❌ One-time use |
| **External iteration** | ✅ `for`, `iterator` | ❌ Internal only |
| **Modification** | ✅ add/remove/update | ❌ Read-only |
| **Lazy evaluation** | ❌ Eager | ✅ Lazy |
| **Parallel support** | ❌ Manual | ✅ `.parallel()` |
| **When to use** | Store & manage data | Process & transform data |

---

### findFirst() vs findAny()

| | `findFirst()` | `findAny()` |
|--|---------------|-------------|
| **Returns** | First element in encounter order | Any element (undefined order) |
| **Sequential stream** | Same result | Same result (first) |
| **Parallel stream** | Must honor order (slower) | Free to pick any (faster) |
| **Best for** | When order matters | Parallel streams |

```java
List<Integer> nums = Arrays.asList(1, 2, 3, 4, 5);

// Both return Optional<Integer>
Optional<Integer> first = nums.stream().filter(n -> n > 2).findFirst(); // 3
Optional<Integer> any   = nums.parallelStream().filter(n -> n > 2).findAny(); // 3,4, or 5
```

---

### Predicate vs Function

| | `Predicate<T>` | `Function<T, R>` |
|--|----------------|-----------------|
| **Method** | `test(T)` | `apply(T)` |
| **Returns** | `boolean` | `R` (any type) |
| **Purpose** | Condition check / filtering | Transformation / mapping |
| **Used in** | `filter()`, `removeIf()` | `map()`, `computeIfAbsent()` |

```java
Predicate<String> isLong = s -> s.length() > 5;   // true/false
Function<String, Integer> getLen = String::length;  // returns Integer
```

---

### Comparable vs Comparator

| Feature | `Comparable` | `Comparator` |
|---------|-------------|--------------|
| **Package** | `java.lang` | `java.util` |
| **Method** | `compareTo(T o)` | `compare(T o1, T o2)` |
| **Where defined** | Inside the class | Outside the class |
| **Sorting** | Natural/default order | Custom/multiple orders |
| **Modifiable** | Only one sort order | Multiple sort orders |

```java
// Comparable — natural order (inside class)
class Employee implements Comparable<Employee> {
    int salary;
    public int compareTo(Employee other) {
        return this.salary - other.salary; // sort by salary
    }
}

// Comparator — custom order (outside class)
Comparator<Employee> byName = (e1, e2) -> e1.name.compareTo(e2.name);
Comparator<Employee> bySalaryDesc = Comparator.comparingInt(Employee::getSalary).reversed();

employees.sort(byName);
employees.sort(bySalaryDesc);
```

---

## 🧪 Quick Practice MCQs

**Q1.** Which of these is NOT a valid lambda syntax?
- a) `() -> 42`
- b) `x -> x * 2`
- c) `(x, y) -> x + y`
- d) `x, y -> x + y` ✅ **(Answer: d — needs parentheses for multiple params)**

**Q2.** What does `Optional.of(null)` throw?
- a) `IllegalArgumentException`
- b) `NullPointerException` ✅
- c) `NoSuchElementException`
- d) Nothing

**Q3.** Which interface has the `test()` method?
- a) `Function` b) `Consumer` c) `Predicate` ✅ d) `Supplier`

**Q4.** Streams are evaluated...
- a) Eagerly b) **Lazily** ✅ c) In parallel always d) On creation

**Q5.** Which method flattens nested streams?
- a) `map()` b) **`flatMap()`** ✅ c) `reduce()` d) `peek()`

---

## 📝 Mini Practice Assignments

```
1. From a list of integers, filter evens, square them, sort descending, collect to list.

2. Given List<Employee>, group by department using Collectors.groupingBy().

3. From List<String>, find longest string using Stream + reduce().

4. Use CompletableFuture to fetch two strings asynchronously and combine them.

5. Use Optional to safely get city from a nullable User → Address → City chain.

6. Parse "25-12-2024 08:30:00" into LocalDateTime using DateTimeFormatter.

7. Partition a list of numbers into primes and non-primes using partitioningBy().

8. Using Method References, convert list of strings to uppercase without a lambda body.
```

---

## 🎯 Revision Cheat Sheet

```
Lambda            → (params) -> body
Functional IF     → 1 abstract method, @FunctionalInterface
Predicate<T>      → test() → boolean
Function<T,R>     → apply() → R
Consumer<T>       → accept() → void
Supplier<T>       → get() → T

Stream pipeline   → source → intermediate (lazy) → terminal
filter()          → keep matching
map()             → transform
flatMap()         → transform + flatten
collect()         → terminal, gather results
reduce()          → combine to single value

Optional          → of() | ofNullable() | empty()
                  → get() | orElse() | orElseGet() | orElseThrow()

Default method    → concrete method in interface (inherited)
Static method     → utility method in interface (NOT inherited)

LocalDate         → date only, immutable
LocalDateTime     → date + time, immutable
DateTimeFormatter → format/parse dates

groupingBy        → Map<K, List<T>>
partitioningBy    → Map<Boolean, List<T>>
joining           → String
counting          → Long

::                → method reference operator
Class::static     → static method reference
obj::method       → instance method reference
Class::new        → constructor reference
```

---

*📌 Last updated: 2024 | ☕ Java 8 | Ready for interviews & revision*
