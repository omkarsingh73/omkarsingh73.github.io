When we talk about "configuration" in Spring Boot, we are usually looking at it from two distinct angles: **How we configure our application behavior (Properties and Profiles)** and **How we configure the Java code itself (Java-based Beans & Auto-configuration)**.

Because Spring Boot operates on the principle of **Convention over Configuration (CoC)**, it provides default settings out of the box, but gives you incredibly flexible ways to override them.

## 1. Externalized Application Configuration (Properties & YAML)

Spring Boot allows you to externalize your configuration so you can use the same application code across dev, staging, and production environments. This is usually handled via `application.properties` or `application.yml`.

### The Precedence Hierarchy (How Spring Boot Resolves Conflicts)

If you define the exact same property (e.g., `server.port`) in multiple places, Spring Boot uses a strict hierarchy to decide which one wins. The higher it is on this list, the higher its priority:

1. **Command-Line Arguments** (e.g., `java -jar app.jar --server.port=9000`)
    
2. **Java System Properties** (e.g., `java -Dserver.port=9000 -jar app.jar`)
    
3. **OS Environment Variables** (e.g., `SERVER_PORT=9000`) — _Crucial for Docker & Kubernetes._
    
4. **External Profile-Specific Config** (`application-prod.yml` outside the jar)
    
5. **Packaged Profile-Specific Config** (`application-prod.yml` inside the jar)
    
6. **External Application Config** (`application.yml` outside the jar)
    
7. **Packaged Application Config** (`application.yml` inside the jar)
    

### Where Spring Boot Looks for Config Files

By default, Spring Boot searches for `application.properties` or `application.yml` in four default locations, in the following order:

1. A `/config` subdirectory of the current running directory.
    
2. The current directory.
    
3. A classpath `/config` package.
    
4. The root of the classpath (your standard `src/main/resources`).
    

## 2. Managing Environments with Profiles

Instead of manually tweaking settings, Spring Boot uses **Profiles** to segregate parts of your application configuration and make it only available in certain environments.

- **Activating a Profile:** You can tell Spring Boot which environment is active via properties or environment variables:
    
    YAML
    
    ```
    spring:
      profiles:
        active: prod
    ```
    
- **Profile-Specific Files:** You create separate files naming them `application-{profile}.yml`. For example, `application-dev.yml` will automatically load alongside your default `application.yml` when the `dev` profile is active.
    

## 3. Code-Based Configuration (Java Config)

Gone are the days of massive, unreadable XML configuration files. Modern Spring Boot configures components directly via Java classes using annotations.

- **`@Configuration`:** Tells Spring that a class is a source of bean definitions.
    
- **`@Bean`:** Used on a method inside a `@Configuration` class to state that the method returns a bean to be managed by the Spring IoC container.
    
- **`@ConfigurationProperties`:** Binds an entire block of properties (from your YAML/properties file) into a type-safe Java object automatically.
    

Java

```
@Configuration
@ConfigurationProperties(prefix = "database")
public class DatabaseConfig {
    private String url;
    private String username;

    @Bean
    public MyDataSource customDataSource() {
        return new MyDataSource(this.url, this.username);
    }
    // getters and setters
}
```

## 4. Behind the Scenes: Auto-Configuration

The "magic" of Spring Boot comes from **Auto-Configuration**. When you annotate your main class with `@SpringBootApplication`, it implicitly includes `@EnableAutoConfiguration`.

- **How it works:** Spring Boot scans your classpath for dependencies. If it sees `spring-boot-starter-data-jpa` and a driver like H2 or PostgreSQL on the classpath, it guesses you want to connect to a database and attempts to auto-configure a `DataSource` bean for you using standard defaults.
    
- **Conditional Configuration:** Auto-configuration relies heavily on conditional annotations like `@ConditionalOnClass` or `@ConditionalOnMissingBean`. This ensures that if you define your _own_ custom bean (like the `customDataSource` in the code block above), Spring Boot will gracefully step aside and not generate its default one.
    

## Summary Decision Tree: Where should you put config?

- **Is it a secret (password, API key)?** $\rightarrow$ Use Environment Variables or a Vault management system.Never commit these to Git.
    
- **Does it change based on environment (Dev vs. Prod)?** $\rightarrow$ Put it in profile-specific files (`application-dev.yml` / `application-prod.yml`).
    
- **Is it a structural definition of a Java object/service?** $\rightarrow$ Use Java `@Configuration` classes.

## Annotations

## 1. Core Framework & Component Scanning

- **`@SpringBootApplication`**: The master annotation that combines `@Configuration`, `@EnableAutoConfiguration`, and `@ComponentScan` to boot up the application.
    
- **`@Component`**: Marks a Java class as a Spring-managed bean so it can be automatically detected and registered in the application context.
    
- **`@Autowired`**: Instructs Spring to automatically inject a matching dependency (bean) into a constructor, field, or setter.
    
- **`@Qualifier`**: Used alongside `@Autowired` to specify exactly which bean to inject when multiple beans of the same type exist.
    
- **`@Primary`**: Signals that a specific bean should be given preference when multiple beans match a dependency injection target.
    
- **`@Lazy`**: Delays the initialization of a bean until it is actually requested for the first time, rather than at startup.
    
- **`@Scope`**: Defines the lifecycle and visibility of a bean (e.g., `singleton`, `prototype`, `request`, `session`).
    

## 2. Configuration & Properties

- **`@Configuration`**: Indicates that a class declares one or more `@Bean` methods and can be processed by the Spring container to generate bean definitions.
    
- **`@Bean`**: Placed on a method within a `@Configuration` class to explicitly register the method's return value as a Spring bean.
    
- **`@Value`**: Injects values from external properties files, system variables, or expressions directly into fields.
    
- **`@ConfigurationProperties`**: Binds a group of related external configuration properties (like a YAML block) into a strongly-typed Java object.
    
- **`@Profile`**: Configures a component or configuration class to only be loaded when a specific application environment (profile) is active.
    

## 3. Web & REST APIs (Spring MVC)

- **`@RestController`**: A convenience annotation that combines `@Controller` and `@ResponseBody`, marking a class as a web request handler that returns data directly (usually JSON).
    
- **`@RequestMapping`**: Maps web requests to specific handler classes or methods based on the URL path and HTTP method.
    
- **`@GetMapping`**: A shortcut annotation specifically for mapping HTTP GET requests onto handler methods.
    
- **`@PostMapping`**: A shortcut annotation specifically for mapping HTTP POST requests onto handler methods.
    
- **`@PutMapping`**: A shortcut annotation specifically for mapping HTTP PUT requests onto handler methods.
    
- **`@DeleteMapping`**: A shortcut annotation specifically for mapping HTTP DELETE requests onto handler methods.
    
- **`@PathVariable`**: Extracts a dynamic value directly out of the URI path (e.g., `/users/{id}`).
    
- **`@RequestParam`**: Extracts query parameters (e.g., `?name=John`) or form data from the incoming HTTP request.
    
- **`@RequestBody`**: Automatically deserializes the incoming HTTP request body into a Java object (typically from JSON).
    
- **`@ResponseStatus`**: Defines the specific HTTP status code (e.g., `201 Created` or `404 Not Found`) that a method or exception should return.
    

## 4. Stereotype Layering (Architecture)

- **`@Service`**: A specialized form of `@Component` used to mark classes that house business logic.
    
- **`@Repository`**: A specialized form of `@Component` used for data access classes, which also automatically translates database-specific exceptions into Spring's hierarchy.
    
- **`@Controller`**: Marks a class as a traditional Spring MVC web controller, typically used to render HTML views/templates.
    

## 5. Data & Transactions (JPA/Hibernate)

- **`@Entity`**: Specifies that a Java class maps directly to a database table row.
    
- **`@Id`**: Designates the primary key field of a database entity.
    
- **`@GeneratedValue`**: Defines the strategy used to automatically generate primary key values (like database auto-increment).
    
- **`@Transactional`**: Wraps a method or class in a database transaction, automatically committing on success and rolling back on runtime exceptions.
    

## 6. Testing & Monitoring

- **`@SpringBootTest`**: Tells Spring Boot to look for a main configuration class and use it to spin up a full application context for integration tests.
    
- **`@MockBean`**: Creates a Mockito mock for a bean and injects it into the Spring application context to isolate components during testing.
    
- **`@WebMvcTest`**: Focuses a test strictly on the Spring MVC web layer, mocking out databases and external services for fast controller testing.