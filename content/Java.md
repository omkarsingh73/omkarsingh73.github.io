# ☕ Java Basics — Revision Notes

> Senior engineer quick-reference · small examples for every concept

  

---

  

## Table of Contents

1. [Primitive Types & Wrappers](#1-primitive-types--wrappers)

2. [Strings & String Pool](#2-strings--string-pool)

3. [OOP — Classes, Inheritance, Polymorphism](#3-oop--classes-inheritance-polymorphism)

4. [Collections Framework](#4-collections-framework)

5. [Generics & Type Bounds](#5-generics--type-bounds)

6. [Exception Handling](#6-exception-handling)

7. [Functional Interfaces & Lambdas](#7-functional-interfaces--lambdas)

8. [Memory Model & Garbage Collection](#8-memory-model--garbage-collection)

9. [Modern Java (11–21)](#9-modern-java-1121)

  

---

  

## 1. Primitive Types & Wrappers

  

### 8 Primitive Types `[core]`

Java has 8 primitives: `byte`(1B), `short`(2B), `int`(4B), `long`(8B), `float`(4B), `double`(8B), `char`(2B), `boolean`. Stored on stack — faster than objects.

  

```java

byte b = 127; // -128 to 127

int i = 2_147_483_647; // underscores ok since Java 7

long l = 9_999_999_999L; // needs L suffix

double d = 3.14d;

char c = 'A'; // Unicode \u0041

boolean flag = true;

```

  

---

  

### Autoboxing & Unboxing `[⚠ trap]`

Java auto-converts primitives ↔ wrappers. **Trap:** `==` on `Integer` compares references, not values. Cached only for `-128..127`.

  

```java

Integer a = 127; Integer b = 127;

System.out.println(a == b); // true (cached)

  

Integer x = 200; Integer y = 200;

System.out.println(x == y); // false (not cached!)

System.out.println(x.equals(y)); // true ✓ — always use equals()

```

  

---

  

### Type Casting `[tip]`

Widening is implicit (`int` → `long`). Narrowing needs explicit cast — potential data loss.

  

```java

int i = 300;

byte b = (byte) i; // 44 — overflow!

long l = i; // widening — safe

double d = (double) i / 7; // 42.857...

```

  

---

  

## 2. Strings & String Pool

  

### String Immutability & Pool `[⚠ trap]`

Strings are immutable — every operation creates a new object. Literals go to the String Pool (PermGen/Metaspace); `new String()` bypasses it.

  

```java

String a = "hello"; // pool

String b = "hello"; // same pool ref

String c = new String("hello"); // heap

  

a == b // true (pool)

a == c // false (heap vs pool)

a.equals(c) // true — always use equals()

c.intern() == a // true — intern() forces pool

```

  

---

  

### StringBuilder vs StringBuffer `[tip]`

`StringBuilder` is mutable & faster (not thread-safe). `StringBuffer` is thread-safe (synchronized) but slower. Use `StringBuilder` in single-threaded loops.

  

```java

StringBuilder sb = new StringBuilder();

sb.append("Hello").append(" World");

sb.insert(5, ",");

sb.reverse();

String result = sb.toString(); // "dlroW ,olleH"

```

  

---

  

### Key String Methods `[core]`

`charAt`, `substring`, `indexOf`, `contains`, `startsWith`, `endsWith`, `trim`/`strip` (strip is Unicode-aware), `split`, `format`, `valueOf`, `compareTo`.

  

```java

String s = " Java 21 ";

s.strip() // "Java 21"

s.strip().length() // 7

s.indexOf("21") // 8

s.substring(2, 6) // "Java"

String.format("%s v%d", "Java", 21); // "Java v21"

"a,b,c".split(","); // ["a", "b", "c"]

```

  

---

  

## 3. OOP — Classes, Inheritance, Polymorphism

  

### Class Anatomy `[core]`

Fields, constructors, methods, static members, initializer blocks. Instance initializer runs before constructor body.

  

```java

public class Account {

private static int count = 0; // class-level

private final int id;

private double balance;

  

// instance initializer block

{ count++; }

  

public Account(double balance) {

this.id = count;

this.balance = balance;

}

  

public static int getCount() { return count; }

}

```

  

---

  

### Inheritance & super `[core]`

Single inheritance via `extends`. `super()` must be first statement in constructor. All Java classes implicitly extend `Object`.

  

```java

public class Animal {

protected String name;

public Animal(String name) { this.name = name; }

public String sound() { return "..."; }

}

  

public class Dog extends Animal {

public Dog(String name) { super(name); }

  

@Override

public String sound() { return "Woof!"; }

}

```

  

---

  

### Polymorphism — Dynamic Dispatch `[core]`

Method resolved at runtime based on actual object type, not reference type. Only instance methods — NOT fields or static methods.

  

```java

Animal a = new Dog("Rex");

a.sound(); // "Woof!" — runtime type wins

  

// Pattern matching instanceof (Java 16)

if (a instanceof Dog d) {

System.out.println(d.name + " is a dog");

}

```

  

---

  

### Abstract Classes vs Interfaces `[⚠ trap]`

- **Abstract class:** can have state + partial implementation.

- **Interface:** no instance state, supports multiple implementation.

- Java 8+ interfaces allow `default` & `static` methods.

  

```java

interface Flyable {

void fly(); // abstract

default void land() { System.out.println("Landing"); }

static Flyable noOp() { return () -> {}; }

}

  

abstract class Vehicle {

protected int speed; // state allowed

abstract void accelerate();

void stop() { speed = 0; } // concrete method

}

```

  

---

  

## 4. Collections Framework

  

### List — ArrayList vs LinkedList `[tip]`

- `ArrayList`: O(1) random access, O(n) insert/delete at middle.

- `LinkedList`: O(1) insert/delete at head/tail, O(n) access.

- **Default choice:** `ArrayList`.

  

```java

List<String> list = new ArrayList<>();

list.add("a");

list.add(0, "z"); // [z, a]

list.get(0); // "z" — O(1)

list.remove("a");

Collections.sort(list);

  

List<String> immutable = List.of("x", "y", "z"); // Java 9+

```

  

---

  

### Map — HashMap, LinkedHashMap, TreeMap `[core]`

- `HashMap`: O(1) avg, unordered.

- `LinkedHashMap`: insertion-order.

- `TreeMap`: sorted by key, O(log n).

- All allow null value; `HashMap` allows null key.

  

```java

Map<String, Integer> map = new HashMap<>();

map.put("a", 1);

map.getOrDefault("b", 0); // 0

map.putIfAbsent("a", 99); // no-op

map.computeIfAbsent("c", k -> k.length()); // 1

map.forEach((k, v) -> System.out.println(k + "=" + v));

```

  

---

  

### Set — HashSet, LinkedHashSet, TreeSet `[tip]`

`HashSet`: O(1), unordered, no duplicates. `TreeSet`: sorted, O(log n). Requires correct `equals`/`hashCode` contract.

  

```java

Set<Integer> set = new HashSet<>(Arrays.asList(3, 1, 2, 1));

// {1, 2, 3} — duplicate removed

  

TreeSet<Integer> ts = new TreeSet<>(set);

ts.first(); // 1

ts.last(); // 3

ts.headSet(3); // [1, 2]

```

  

---

  

### Comparable vs Comparator `[core]`

- `Comparable`: natural order, implemented on the class (`compareTo`).

- `Comparator`: external comparison, passed at sort time.

  

```java

class Employee implements Comparable<Employee> {

int salary;

public int compareTo(Employee o) {

return Integer.compare(this.salary, o.salary);

}

}

  

// External comparator (lambda)

list.sort(Comparator.comparing(Employee::getSalary)

.thenComparing(Employee::getName));

```

  

---

  

## 5. Generics & Type Bounds

  

### Generic Classes & Methods `[core]`

Type parameters let you write type-safe reusable code. **Erasure:** type info removed at compile time — no `List<Integer>.class` at runtime.

  

```java

public class Pair<A, B> {

private final A first;

private final B second;

public Pair(A a, B b) { first = a; second = b; }

public A getFirst() { return first; }

}

  

Pair<String, Integer> p = new Pair<>("age", 30);

```

  

---

  

### Wildcards — `? extends` / `? super` `[⚠ trap]`

**PECS: Producer Extends, Consumer Super.**

- Use `<? extends T>` to **read** from a structure.

- Use `<? super T>` to **write** into a structure.

  

```java

// Read from list of Numbers (or subtype)

void sum(List<? extends Number> list) {

for (Number n : list) { ... } // OK

}

  

// Write Numbers into a list

void fill(List<? super Integer> list) {

list.add(42); // OK

}

```

  

---

  

## 6. Exception Handling

  

### Checked vs Unchecked `[core]`

- **Checked:** extends `Exception` — must be caught or declared (`IOException`, `SQLException`).

- **Unchecked:** extends `RuntimeException` — no forced handling (`NullPointerException`, `IllegalArgumentException`).

  

```java

class InsufficientFundsException extends Exception { // checked

public InsufficientFundsException(String msg) { super(msg); }

}

  

void withdraw(double amt) throws InsufficientFundsException {

if (amt > balance) throw new InsufficientFundsException("Low!");

}

```

  

---

  

### try-with-resources `[tip]`

`AutoCloseable` resources are automatically closed, even on exception. Replaces verbose `finally` blocks. Suppressed exceptions are accessible.

  

```java

try (BufferedReader br = new BufferedReader(new FileReader("f.txt"));

Connection conn = ds.getConnection()) {

  

return br.readLine(); // both auto-closed

}

// no finally needed!

```

  

---

  

### Multi-catch & Exception Chaining `[tip]`

Catch multiple types with `|`. Use constructor to chain exceptions — preserves root cause.

  

```java

try {

riskyOperation();

} catch (IOException | SQLException e) {

throw new ServiceException("DB or IO issue", e); // chained

} finally {

cleanup(); // always runs

}

```

  

---

  

## 7. Functional Interfaces & Lambdas

  

### Functional Interfaces `[core]`

Single abstract method (SAM). Built-in: `Function<T,R>`, `Predicate<T>`, `Consumer<T>`, `Supplier<T>`, `BiFunction<T,U,R>`. Mark with `@FunctionalInterface`.

  

```java

Function<String, Integer> len = String::length;

Predicate<Integer> even = n -> n % 2 == 0;

Consumer<String> print = System.out::println;

Supplier<List> listMaker = ArrayList::new;

  

even.and(n -> n > 0).test(4); // true — compose predicates

```

  

---

  

### Streams — map / filter / reduce `[core]`

Lazy pipeline: intermediate ops (`map`, `filter`, `sorted`) don't execute until terminal op (`collect`, `reduce`, `forEach`). Streams are **single-use**.

  

```java

List<String> names = List.of("Alice", "Bob", "Charlie", "Dave");

  

Map<Integer, List<String>> byLen = names.stream()

.filter(s -> s.length() > 3)

.map(String::toUpperCase)

.sorted()

.collect(Collectors.groupingBy(String::length));

```

  

---

  

### Optional `[tip]`

Avoid `NullPointerException`. Never call `get()` without `isPresent()`. Prefer `orElse`, `orElseGet`, `orElseThrow`, `map`, `flatMap`.

  

```java

Optional<String> opt = findUser(42);

  

// BAD:

if (opt.isPresent()) opt.get();

  

// GOOD:

String name = opt

.map(User::getName)

.filter(n -> !n.isBlank())

.orElse("Anonymous");

  

opt.ifPresentOrElse(

u -> log(u), () -> log("not found")); // Java 9+

```

  

---

  

## 8. Memory Model & Garbage Collection

  

### Heap & Stack `[core]`

- **Stack:** method frames, local variables, primitives — per-thread, LIFO, fast.

- **Heap:** all objects — shared, GC managed.

- GC flow: Young gen (Eden + Survivors) → Old gen.

  

```java

void method() {

int x = 5; // stack

String s = "hi"; // ref on stack, object on heap

Object o = new Object(); // heap

} // x, s, o refs popped; object eligible for GC

```

  

---

  

### equals() & hashCode() Contract `[⚠ trap]`

If `a.equals(b)` → `a.hashCode() == b.hashCode()`. Break this and `HashMap`/`HashSet` will misbehave. **Always override both together.**

  

```java

@Override

public boolean equals(Object o) {

if (this == o) return true;

if (!(o instanceof Point p)) return false;

return x == p.x && y == p.y;

}

  

@Override

public int hashCode() {

return Objects.hash(x, y); // consistent with equals

}

```

  

---

  

## 9. Modern Java (11–21)

  

### Records (Java 16) `[core]`

Immutable data carriers. Auto-generates constructor, getters, `equals`, `hashCode`, `toString`. Can have compact constructors and static methods.

  

```java

record Point(int x, int y) {

// compact constructor for validation

Point { if (x < 0) throw new IllegalArgumentException(); }

double distance() { return Math.sqrt(x * x + y * y); }

}

  

Point p = new Point(3, 4);

p.x(); // 3 — accessor, not getX()

p.distance(); // 5.0

```

  

---

  

### Sealed Classes (Java 17) `[core]`

Restrict which classes can extend/implement. Works great with pattern matching `switch`. Enables exhaustive checks.

  

```java

sealed interface Shape permits Circle, Rectangle, Triangle {}

  

record Circle(double r) implements Shape {}

record Rectangle(double w, double h) implements Shape {}

  

double area(Shape s) {

return switch (s) {

case Circle c -> Math.PI * c.r() * c.r();

case Rectangle r -> r.w() * r.h();

case Triangle t -> /* ... */ 0;

};

}

```

  

---

  

### Text Blocks (Java 15) `[tip]`

Multi-line strings with incidental whitespace stripped. Triple quote opens on new line. Great for JSON, SQL, HTML snippets.

  

```java

String json = """

{

"name": "Omkar",

"role": "Senior Engineer",

"skills": ["Java", "Spring Boot"]

}

"""; // no leading whitespace in output

  

String sql = """

SELECT *

FROM users

WHERE active = true

""";

```

  

---

  

### var — Local Variable Type Inference (Java 10) `[tip]`

Compiler infers type. Only for local variables — not fields, params, or return types. Never use when type is not obvious from RHS.

  

```java

var list = new ArrayList<String>(); // inferred: ArrayList<String>

var map = new HashMap<String, Integer>();

  

// Good: type clear from RHS

var conn = dataSource.getConnection();

  

// Bad: ambiguous

var x = process(); // What type is this? Avoid.

```

  

---

  

## Quick Reference Cheat Sheet

  

| Topic | Key Rule |

|---|---|

| `==` vs `equals()` | Always use `equals()` for objects |

| Integer cache | `-128..127` cached; `==` unreliable outside this range |

| PECS | Producer `extends`, Consumer `super` |

| Stream pipeline | Lazy — nothing runs until terminal op |

| `equals` + `hashCode` | Must both be overridden together |

| `StringBuilder` | Use in loops; `StringBuffer` only if thread-safe needed |

| Checked exception | Must `catch` or `throws`; unchecked is optional |

| `record` | Immutable DTO — replaces boilerplate POJOs |

| `var` | Only when type is obvious from right-hand side |

| `Optional.get()` | Never call without `isPresent()` — use `orElse` instead |

  

---

  

*Next topics: Spring Boot · Spring Batch · Concurrency · JVM Internals · Design Patterns*