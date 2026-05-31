
- [[#📚 **What is SOLID?**|📚 **What is SOLID?**]]
- [[#1. **Single Responsibility Principle (SRP)**|1. **Single Responsibility Principle (SRP)**]]
- [[#2. **Open-Closed Principle (OCP)**|2. **Open-Closed Principle (OCP)**]]
- [[#3. **Liskov Substitution Principle (LSP)**|3. **Liskov Substitution Principle (LSP)**]]
- [[#4. **Interface Segregation Principle (ISP)**|4. **Interface Segregation Principle (ISP)**]]
- [[#5. **Dependency Inversion Principle (DIP)**|5. **Dependency Inversion Principle (DIP)**]]
- [[#🎯 **Key Takeaways**|🎯 **Key Takeaways**]]
- [[#🛠️ **Production Checklist**|🛠️ **Production Checklist**]]


---

## 📚 **What is SOLID?**  
Five object-oriented design principles that enable **maintainable, scalable, and testable backend code**.  

| Principle | Full Name | Core Idea |
|-----------|-----------|-----------|
| **S** | Single Responsibility | One reason to change |
| **O** | Open-Closed | Extend without modifying |
| **L** | Liskov Substitution | Subtypes replace supertypes |
| **I** | Interface Segregation | Smaller, specific interfaces |
| **D** | Dependency Inversion | Depend on abstractions, not concretions |

---

## 1. **Single Responsibility Principle (SRP)**  
### **What?**  
*A class/method should have **one** job.*  

### **Backend Example**  
```java
// ❌ Bad: Violates SRP
class UserService {
    void createUser(User user) { /* ... */ }
    void sendWelcomeEmail(User user) { /* ... */ } // Logging/email here!
    void logUserCreation(User user) { /* ... */ }
}

// ✅ Good: Split responsibilities
class UserService {
    void createUser(User user) { /* ... */ }
}

class UserNotificationService {
    void sendWelcomeEmail(User user) { /* ... */ }
}

class UserAuditLogger {
    void logUserCreation(User user) { /* ... */ }
}
```

### **Why?**  
- **Change Impact**: Email logic changes don’t risk breaking user creation.  
- **Testability**: Test `UserService` without mocking email/logger.  

### **Interview Tip**  
> *"How would you refactor a `ReportGenerator` that also handles file storage and email notifications?"*  

### **Common Mistake**  
- Adding logging, metrics, or validation to core business classes.  

### **Production Insight**  
Use **aspect-oriented programming (AOP)** or **decorators** for cross-cutting concerns (logging, metrics).  

### **Tradeoff**  
- **Pro**: Clear boundaries, easier debugging.  
- **Con**: Risk of over-fragmentation (too many small classes).  

---

## 2. **Open-Closed Principle (OCP)**  
### **What?**  
*Classes/functions should be **open for extension** but **closed for modification**.*  

### **Backend Example**  
```java
// ❌ Bad: Adding new payment method requires modifying PaymentProcessor
class PaymentProcessor {
    void process(PaymentMethod method) {
        if (method.type == "CREDIT_CARD") { /* ... */ }
        if (method.type == "PAYPAL") { /* ... */ }
        // TODO: Add new method here → violates OCP!
    }
}

// ✅ Good: Extend via polymorphism
interface PaymentStrategy {
    void processPayment(PaymentDetails details);
}

class CreditCardPayment implements PaymentStrategy { /* ... */ }
class PayPalPayment implements PaymentStrategy { /* ... */ }

class PaymentProcessor {
    private PaymentStrategy strategy;

    void process(PaymentStrategy strategy) {
        this.strategy = strategy;
        strategy.processPayment(...);
    }
}
```

### **Why?**  
- **Future-proof**: Add new payment methods **without touching** `PaymentProcessor`.  
- **Testability**: Mock `PaymentStrategy` easily.  

### **Interview Tip**  
> *"How would you design a logging system that supports new log levels (e.g., DEBUG, INFO) without changing existing code?"*  

### **Common Mistake**  
- Using `if-else`/`switch` for type-driven logic instead of polymorphism.  

### **Production Insight**  
Use **strategy pattern** or **plugins** (e.g., Spring’s `@Component` scanning).  

### **Tradeoff**  
- **Pro**: Easy to extend, reduces regression risk.  
- **Con**: Increased class count, learning curve for juniors.  

---

## 3. **Liskov Substitution Principle (LSP)**  
### **What?**  
*Subtypes must be substitutable for their supertypes without breaking behavior.*  

### **Backend Example**  
```java
// ❌ Bad: Violates LSP
abstract class Bird {
    abstract void fly();
}

class Eagle extends Bird {
    void fly() { /* ... */ }
}

class Ostrich extends Bird { // Ostrich cannot fly!
    void fly() { throw new UnsupportedOperationException(); }
}

// ✅ Good: Separate interfaces
interface FlyingBird {
    void fly();
}

interface NonFlyingBird {
    void walk();
}

class Eagle implements FlyingBird { /* ... */ }
class Ostrich implements NonFlyingBird { /* ... */ }
```

### **Why?**  
- **Type safety**: Code expecting `Bird` shouldn’t care about concrete subtypes.  

### **Interview Tip**  
> *"Why can’t we use `Ostrich` where `Bird` is expected if `fly()` throws an exception?"*  

### **Common Mistake**  
- Subclasses overriding methods with **narrower scope** or **different exceptions**.  

### **Production Insight**  
Use **interfaces** to define contracts; avoid inheritance for type hierarchies.  

### **Tradeoff**  
- **Pro**: Safer code, better polymorphism.  
- **Con**: More interfaces to manage.  

---

## 4. **Interface Segregation Principle (ISP)**  
### **What?**  
*Clients should not depend on interfaces they don’t use.*  

### **Backend Example**  
```java
// ❌ Bad: Fat interface
interface Worker {
    void work();
    void eat(); // Unrelated to work
    void sleep(); 
}

// ✅ Good: Split interfaces
interface Worker {
    void work();
}

interface Eater {
    void eat();
}

interface Sleeper {
    void sleep();
}

class Employee implements Worker, Eater, Sleeper { /* ... */ }
```

### **Why?**  
- **Reduced coupling**: `Worker` doesn’t need to implement `eat()` or `sleep()`.  
- **Flexibility**: `RobotWorker` can implement only `Worker`.  

### **Interview Tip**  
> *"How would you design a `NotificationService` that supports email, SMS, and push notifications without forcing clients to depend on all three?"*  

### **Common Mistake**  
- Creating “god interfaces” that group unrelated methods.  

### **Production Insight**  
Use **domain-driven design (DDD)** bounded contexts to naturally segregate interfaces.  

### **Tradeoff**  
- **Pro**: Smaller, focused interfaces.  
- **Con**: Increased interface count.  

---

## 5. **Dependency Inversion Principle (DIP)**  
### **What?**  
*Depend on **abstractions** (interfaces), not concrete implementations.*  

### **Backend Example**  
```java
// ❌ Bad: Hard-coded dependency
class ReportGenerator {
    private final FileSystemReportStorage storage = new FileSystemReportStorage();

    void generateReport() {
        storage.saveReport(...); // Tight coupling to FileSystemReportStorage
    }
}

// ✅ Good: Depend on abstraction
interface ReportStorage {
    void saveReport(Report report);
}

class FileSystemReportStorage implements ReportStorage { /* ... */ }
class S3ReportStorage implements ReportStorage { /* ... */ }

class ReportGenerator {
    private final ReportStorage storage;

    public ReportGenerator(ReportStorage storage) { // Inject abstraction
        this.storage = storage;
    }
}
```

### **Why?**  
- **Flexibility**: Swap `FileSystemReportStorage` with `S3ReportStorage` without changing `ReportGenerator`.  
- **Testability**: Mock `ReportStorage` in unit tests.  

### **Interview Tip**  
> *"How would you refactor a service that directly instantiates a `DatabaseConnection`?"*  

### **Common Mistake**  
- Instantiating concrete classes inside methods (hard-coded dependencies).  

### **Production Insight**  
Use **dependency injection frameworks** (Spring, Guice) to manage dependencies.  

### **Tradeoff**  
- **Pro**: Loose coupling, easier testing.  
- **Con**: Increased boilerplate (interfaces, DI configuration).  

---

## 🎯 **Key Takeaways**  

| Principle | Backend Impact | Common Violation | Fix |
|-----------|----------------|-------------------|-----|
| **SRP**   | Reduces bug surface | Mixing business logic with logging/validation | Extract cross-cutting concerns |
| **OCP**   | Easier to add features | `if-else` for type checks | Use strategy pattern |
| **LSP**   | Safer polymorphism | Subclass overrides with exceptions | Use interfaces, avoid inheritance |
| **ISP**   | Less unused code | “God interfaces” | Split by responsibility |
| **DIP**   | Testable, flexible | New concrete dependencies | Inject interfaces |

---

## 🛠️ **Production Checklist**  
1. **SRP**: Does a class have **one** reason to change?  
2. **OCP**: Can you add a feature **without modifying** existing code?  
3. **LSP**: Can you substitute a subtype without breaking clients?  
4. **ISP**: Are interfaces **cohesive** (all methods related)?  
5. **DIP**: Are dependencies **injected** as interfaces?  

> **Final Tip**: Apply SOLID **gradually**. Over-engineering for small services can hurt more than it helps. Balance readability and future extensibility.