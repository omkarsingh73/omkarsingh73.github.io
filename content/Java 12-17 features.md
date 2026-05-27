# Most Important Features for Interviews

|Feature|Version|Why Important|
|---|---|---|
|Records|Java 16|Replaces boilerplate POJOs|
|Sealed Classes|Java 17|Restricts inheritance|
|Text Blocks|Java 15|Cleaner SQL/JSON/XML strings|
|Pattern Matching|Java 16|Simplifies `instanceof` checks|
|Switch Expressions|Java 14|Cleaner modern switch syntax|
|Helpful NPE|Java 14|Easier debugging|

---

# Important Examples

## 1. Switch Expression

```java
String result = switch(day) {
    case 1 -> "Monday";
    case 2 -> "Tuesday";
    default -> "Invalid";
};
```

---

## 2. Text Blocks

```java
String json = """
{
   "name": "Rahul",
   "role": "Developer"
}
""";
```

---

## 3. Records

```java
record Employee(int id, String name) {}

Employee emp = new Employee(1, "Rahul");

System.out.println(emp.name());
```

---

## 4. Pattern Matching for `instanceof`

```java
Object obj = "Java";

if(obj instanceof String str) {
    System.out.println(str.length());
}
```

---

## 5. Sealed Classes

```java
sealed class Vehicle permits Car, Bike {}

final class Car extends Vehicle {}

final class Bike extends Vehicle {}
```

---

## 6. Stream `toList()`

```java
List<String> list = Stream.of("A", "B")
                          .toList();
```

---

# Java 12–17 Quick Revision Table

|Topic|Key Point|
|---|---|
|Switch Expression|Cleaner alternative to old switch|
|Text Blocks|Multi-line strings|
|Records|Immutable DTO classes|
|Pattern Matching|Removes manual casting|
|Sealed Classes|Controlled inheritance|
|Helpful NPE|Exact null variable identification|
|Stream `toList()`|Immutable list creation|

---

# `class` vs `record`

|Class|Record|
|---|---|
|Boilerplate code required|Auto-generates constructor/getters|
|Mutable by default|Immutable by default|
|Manual `equals/hashCode`|Auto-generated|

---

# Common Interview Questions

## Why use Records?

- Reduce boilerplate
    
- Immutable objects
    
- Better readability
    
- Cleaner DTO models
    

---

## Why use Sealed Classes?

- Restrict inheritance
    
- Better domain modeling
    
- Useful with pattern matching
    
- Safer API design
    

---

# Important Note

```text
Java 17 is an LTS version and widely adopted in enterprise applications.
```

---
# Java 12 to Java 17 — Important Features

|Version|Feature|Description|
|---|---|---|
|Java 12|Switch Expressions (Preview)|Cleaner switch syntax with `yield`|
|Java 12|JVM Improvements|Better G1 Garbage Collector|
|Java 13|Text Blocks (Preview)|Multi-line strings using `"""`|
|Java 13|Switch Expression Enhancement|Improved switch readability|
|Java 14|Records (Preview)|Immutable data carrier classes|
|Java 14|Helpful NullPointerException|Better NPE debugging messages|
|Java 14|Pattern Matching for `instanceof` (Preview)|Reduced casting boilerplate|
|Java 15|Text Blocks (Standard)|Official support for multi-line strings|
|Java 15|Sealed Classes (Preview)|Restrict inheritance|
|Java 15|Hidden Classes|Framework/internal JVM optimization|
|Java 16|Records (Standard)|Finalized record feature|
|Java 16|Pattern Matching for `instanceof`|Finalized pattern matching|
|Java 16|Stream `toList()`|Direct immutable list conversion|
|Java 17|Sealed Classes (Standard)|Controlled class hierarchy|
|Java 17|Pattern Matching Enhancements|Cleaner type checks|
|Java 17|Strong Encapsulation|Internal JDK APIs hidden|
|Java 17|New macOS Rendering Pipeline|Better graphics performance|

---
