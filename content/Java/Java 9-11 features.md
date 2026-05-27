
# Most Important Features for Interviews

|Feature|Why Important|
|---|---|
|Module System|Major architecture change in Java 9|
|`var`|Frequently asked Java 10 feature|
|HTTP Client API|Replaced older `HttpURLConnection`|
|Factory Methods|Cleaner immutable collections|
|String Methods|Common coding usage|
|JShell|First official Java REPL|

---

# Important Examples

## 1. Factory Methods (`List.of`)

```java
List<String> list = List.of("Java", "Spring");
```

---

## 2. `var` Keyword

```java
var name = "Rahul";
var count = 10;
```

---

## 3. HTTP Client API

```java
HttpClient client = HttpClient.newHttpClient();

HttpRequest request = HttpRequest.newBuilder()
        .uri(URI.create("https://example.com"))
        .build();

HttpResponse<String> response =
        client.send(request, BodyHandlers.ofString());

System.out.println(response.body());
```

---

## 4. String Enhancements

```java
String str = "  Java  ";

System.out.println(str.isBlank());
System.out.println(str.strip());
System.out.println("Hi ".repeat(3));
```

---

## 5. Stream API Enhancements

```java
List<Integer> nums = List.of(1,2,3,4,5);

nums.stream()
    .takeWhile(n -> n < 4)
    .forEach(System.out::println);
```

---

# Java 9–11 Quick Revision Table

|Topic|Key Point|
|---|---|
|Module System|Better encapsulation and modularity|
|`List.of()`|Immutable collections|
|`var`|Type inferred by compiler|
|HTTP Client|Supports async and HTTP/2|
|JShell|Interactive execution|
|`strip()` vs `trim()`|`strip()` supports Unicode|
|`takeWhile()`|Stops when condition becomes false|
|Single File Execution|`java Test.java`|

---

# Very Common Interview Question

## `trim()` vs `strip()`

|`trim()`|`strip()`|
|---|---|
|Old method|Introduced in Java 11|
|ASCII whitespace only|Unicode-aware|
|Less accurate|Recommended|

---
# 9-11 fetaures

|Version|Feature|Description|
|---|---|---|
|Java 9|Module System (JPMS)|Introduced modules using `module-info.java`|
|Java 9|JShell|Interactive Java REPL tool|
|Java 9|Factory Methods|`List.of()`, `Set.of()`, `Map.of()`|
|Java 9|Stream API Enhancements|`takeWhile()`, `dropWhile()`, `iterate()` improvements|
|Java 9|Optional Enhancements|`ifPresentOrElse()`, `stream()`|
|Java 9|Private Interface Methods|Interfaces can have private helper methods|
|Java 9|Try-With-Resources Improvement|Existing variables can be used directly|
|Java 10|`var` Keyword|Local variable type inference|
|Java 10|Garbage Collector Improvements|Parallel Full GC for G1|
|Java 10|Application CDS|Faster startup and lower memory usage|
|Java 11|HTTP Client API|Modern async/sync HTTP client|
|Java 11|String API Enhancements|`isBlank()`, `lines()`, `repeat()`, `strip()`|
|Java 11|Files API Enhancements|`Files.readString()`, `writeString()`|
|Java 11|Lambda `var` Support|`var` allowed in lambda parameters|
|Java 11|New Garbage Collectors|ZGC and Epsilon GC|
|Java 11|Single File Execution|Run `.java` file directly without compilation|

---
