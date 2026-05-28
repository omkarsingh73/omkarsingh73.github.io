# Table of Contents

- [[#1. Spring Boot Architecture|1. Spring Boot Architecture]]
- [[#2. Spring Core & IoC Container|2. Spring Core & IoC Container]]
- [[#3. Spring Boot Annotations Deep Dive|3. Spring Boot Annotations Deep Dive]]
- [[#4. Spring Boot Configuration Management|4. Spring Boot Configuration Management]]
- [[#5. Spring MVC Internals|5. Spring MVC Internals]]
- [[#6. REST API Best Practices|6. REST API Best Practices]]
- [[#7. Spring Data JPA Deep Dive|7. Spring Data JPA Deep Dive]]
- [[#8. Hibernate Performance Optimization|8. Hibernate Performance Optimization]]
- [[#9. Spring Security Advanced|9. Spring Security Advanced]]
- [[#10. Microservices with Spring Boot|10. Microservices with Spring Boot]]
- [[#11. Spring Boot Actuator|11. Spring Boot Actuator]]
- [[#12. Spring Boot Logging|12. Spring Boot Logging]]
- [[#13. Caching in Spring Boot|13. Caching in Spring Boot]]
- [[#14. Messaging Systems|14. Messaging Systems]]
- [[#15. Scheduling & Async|15. Scheduling & Async]]
- [[#16. Spring Boot Testing|16. Spring Boot Testing]]
- [[#17. Production Deployment|17. Production Deployment]]
- [[#18. JVM & Performance Tuning|18. JVM & Performance Tuning]]
- [[#19. Common Production Issues|19. Common Production Issues]]
- [[#20. Design Patterns in Spring|20. Design Patterns in Spring]]
- [[#21. Spring Boot Interview Questions|21. Spring Boot Interview Questions]]
- [[#22. Spring Boot Cheatsheet|22. Spring Boot Cheatsheet]]



---

## 1. Spring Boot Architecture

### Auto-Configuration Internals

Spring Boot auto-configuration uses `@EnableAutoConfiguration` → reads `META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports` (Boot 3.x) or `spring.factories` (Boot 2.x).

```
SpringApplication.run()
  └── prepareEnvironment()
  └── createApplicationContext()
  └── refreshContext()
       └── invokeBeanFactoryPostProcessors()
            └── ConfigurationClassPostProcessor
                 └── @Conditional evaluation
                      └── Auto-config classes loaded
```

**Spring Factories Mechanism (Boot 2.x):**
```
META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.example.MyAutoConfiguration
```

**Boot 3.x (AOT-friendly):**
```
META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports
com.example.MyAutoConfiguration
```

### Key Conditional Annotations

| Annotation | Condition |
|---|---|
| `@ConditionalOnClass` | Class present on classpath |
| `@ConditionalOnMissingBean` | No existing bean of that type |
| `@ConditionalOnProperty` | Property key/value matches |
| `@ConditionalOnWebApplication` | Is a web application |
| `@ConditionalOnExpression` | SpEL expression evaluates true |
| `@ConditionalOnResource` | Resource (file) exists |

```java
@Configuration
@ConditionalOnClass(DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
public class DataSourceAutoConfiguration {
    // Only active if DataSource class on classpath AND no DataSource bean defined
}
```

### SpringApplication Lifecycle

```
1. SpringApplicationRunListeners.starting()
2. prepareEnvironment() → ApplicationEnvironmentPreparedEvent
3. printBanner()
4. createApplicationContext() → AnnotationConfigServletWebServerApplicationContext
5. prepareContext() → ApplicationContextInitializers applied
6. refreshContext() → full IoC container init
7. afterRefresh()
8. SpringApplicationRunListeners.started() → ApplicationStartedEvent
9. callRunners() → ApplicationRunner / CommandLineRunner
10. SpringApplicationRunListeners.ready() → ApplicationReadyEvent
```

### Embedded Server Architecture

```
Tomcat (default) | Jetty | Undertow | Netty (reactive)
       ↑
TomcatServletWebServerFactory
       ↑
@ConditionalOnClass(Tomcat.class)
       ↑
spring-boot-starter-web (excludes others by default)
```

> **Production Tip:** Replace Tomcat with Undertow for better throughput under high concurrency:
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

> **Interview Note:** Auto-configuration uses a "convention over configuration" model. Beans are created only when conditions are met. The order matters — `@AutoConfigureBefore`, `@AutoConfigureAfter`, `@AutoConfigureOrder` control ordering.

---

## 2. Spring Core & IoC Container

### BeanFactory vs ApplicationContext

| Feature | BeanFactory | ApplicationContext |
|---|---|---|
| Bean instantiation | Lazy by default | Eager (singletons) |
| AOP support | No | Yes |
| Event publishing | No | Yes |
| i18n | No | Yes |
| Usage | Unit tests, embedded | All production use |

### Bean Lifecycle

```
1. Instantiate (Constructor / Factory Method)
2. Populate Properties (DI)
3. BeanNameAware.setBeanName()
4. BeanFactoryAware.setBeanFactory()
5. ApplicationContextAware.setApplicationContext()
6. BeanPostProcessor.postProcessBeforeInitialization()
7. @PostConstruct / InitializingBean.afterPropertiesSet() / init-method
8. BeanPostProcessor.postProcessAfterInitialization()
--- Bean Ready ---
9. @PreDestroy / DisposableBean.destroy() / destroy-method
```

### Bean Scopes

| Scope | Description | Thread-safe? |
|---|---|---|
| `singleton` | One per ApplicationContext | No (shared state) |
| `prototype` | New instance per injection | Yes (not shared) |
| `request` | One per HTTP request | Yes |
| `session` | One per HTTP session | Yes |
| `application` | One per ServletContext | No |
| `websocket` | One per WebSocket session | Yes |

> **Common Mistake:** Injecting a `prototype` bean into a `singleton` bean — the prototype is only created once unless you use `@Lookup` or `ObjectProvider<T>`.

```java
@Component
public class SingletonService {
    @Autowired
    private ObjectProvider<PrototypeBean> prototypeProvider;

    public void doWork() {
        PrototypeBean bean = prototypeProvider.getObject(); // fresh instance each time
    }
}
```

### Circular Dependency

- **Constructor injection** → `BeanCurrentlyInCreationException` (detected at startup)
- **Setter/field injection** → resolved via early bean reference (3-level cache)

**Spring's 3-Level Cache for Circular Dependency:**
```
singletonObjects       → fully initialized beans
earlySingletonObjects  → early exposed beans (post-processed)
singletonFactories     → ObjectFactory for early reference
```

> **Production Tip:** Prefer constructor injection. If you have circular deps, it's a design smell — refactor using `@Lazy`, an intermediary service, or event-driven communication.

### CGLIB vs JDK Dynamic Proxy

| Proxy Type | When Used | Limitation |
|---|---|---|
| JDK Dynamic Proxy | Bean implements interface | Only interfaces proxied |
| CGLIB | No interface / `proxyTargetClass=true` | Cannot proxy `final` methods/classes |

```java
@Configuration
@EnableAspectJAutoProxy(proxyTargetClass = true) // Force CGLIB
public class AppConfig {}
```

### Lazy Initialization

```java
@Lazy // On bean definition
@SpringBootApplication
public class App {
    // Or globally:
}
```
```yaml
spring:
  main:
    lazy-initialization: true  # Boot startup faster, first request slower
```

---

## 3. Spring Boot Annotations Deep Dive

### @SpringBootApplication

```java
@SpringBootApplication
// = @Configuration + @EnableAutoConfiguration + @ComponentScan(basePackages = current package)
```

> **Interview Note:** `@ComponentScan` scans from the package of the annotated class. Placing it in the root package is critical — beans in sub-packages are picked up, but sibling/parent packages are not.

### Meta-Annotations

Spring annotations are composable. `@RestController` = `@Controller` + `@ResponseBody`:
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Controller
@ResponseBody
public @interface RestController { ... }
```

### Stereotype Annotations

| Annotation | Purpose | AOP Proxy? |
|---|---|---|
| `@Component` | Generic bean | Via AOP config |
| `@Service` | Business logic (semantic) | Yes |
| `@Repository` | Data access — enables `PersistenceExceptionTranslation` | Yes |
| `@Controller` | MVC controller | Yes |
| `@RestController` | MVC + response body | Yes |
| `@Configuration` | Bean definition source (CGLIB proxied) | Yes (CGLIB) |

> **Interview Note:** `@Repository` activates `PersistenceExceptionTranslationPostProcessor` which translates JPA/JDBC exceptions to Spring's `DataAccessException` hierarchy.

### @Configuration vs @Component for bean definition

`@Configuration` classes are CGLIB-proxied — `@Bean` method calls are intercepted, ensuring singleton semantics:
```java
@Configuration
public class Config {
    @Bean
    public ServiceA serviceA() { return new ServiceA(serviceB()); } // serviceB() returns same instance

    @Bean
    public ServiceB serviceB() { return new ServiceB(); }
}
```

With `@Component`, `serviceB()` would create a new instance each call — breaks singleton contract.

---

## 4. Spring Boot Configuration Management

### Property Source Priority (highest to lowest)

```
1. Command-line args (--server.port=8081)
2. SPRING_APPLICATION_JSON env var
3. OS environment variables
4. JVM system properties (-Dserver.port=8081)
5. application-{profile}.properties/yml
6. application.properties/yml
7. @PropertySource annotations
8. Default properties
```

### Profiles

```yaml
# application.yml
spring:
  profiles:
    active: prod,metrics

---
spring:
  config:
    activate:
      on-profile: prod
datasource:
  url: jdbc:postgresql://prod-host/db
```

```bash
# Activate at runtime
java -jar app.jar --spring.profiles.active=prod
SPRING_PROFILES_ACTIVE=prod java -jar app.jar
```

### @ConfigurationProperties (Preferred over @Value)

```java
@ConfigurationProperties(prefix = "app")
@Validated
public class AppProperties {
    @NotBlank
    private String name;

    @Min(1) @Max(100)
    private int maxConnections = 10;

    private Map<String, String> headers = new HashMap<>();
    // getters/setters or use @ConstructorBinding (Boot 2.2+)
}
```

```yaml
app:
  name: MyService
  max-connections: 50
  headers:
    X-Custom: value
```

> **Production Tip:** Always use `@Validated` on `@ConfigurationProperties` to fail fast on startup with bad config rather than failing at runtime.

### Config Server (Spring Cloud)

```yaml
# bootstrap.yml (Boot 2.x) or spring.config.import (Boot 3.x)
spring:
  config:
    import: "configserver:http://config-server:8888"
  application:
    name: my-service
  cloud:
    config:
      fail-fast: true
      retry:
        max-attempts: 6
```

---

## 5. Spring MVC Internals

### DispatcherServlet Request Flow

```
HTTP Request
    ↓
DispatcherServlet (Front Controller)
    ↓
HandlerMapping (finds handler + interceptors)
  → RequestMappingHandlerMapping (most common)
    ↓
HandlerAdapter (invokes handler)
  → RequestMappingHandlerAdapter
    ↓
  ArgumentResolvers (bind @RequestParam, @RequestBody, etc.)
    ↓
  MessageConverters (JSON ↔ Object via Jackson)
    ↓
Controller method executes → returns value/ModelAndView
    ↓
ReturnValueHandlers (@ResponseBody → skip ViewResolver)
    ↓
ViewResolver (if returning view name)
    ↓
View renders → HTTP Response
```

### Filters vs Interceptors

| Aspect | Filter (Servlet) | Interceptor (Spring MVC) |
|---|---|---|
| Scope | Any servlet | Spring MVC only |
| Access to Spring beans | Difficult | Full access |
| Invocation point | Before DispatcherServlet | After DispatcherServlet |
| Can abort request | Yes (`chain.doFilter` not called) | Yes (`preHandle` returns false) |
| Use cases | Auth, CORS, logging, compression | Logging, auth (Spring-aware), locale |

```java
// Interceptor registration
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(new MyInterceptor())
                .addPathPatterns("/api/**")
                .excludePathPatterns("/api/health");
    }
}
```

### MessageConverters

Common converters registered by `WebMvcConfigurationSupport`:
- `MappingJackson2HttpMessageConverter` — JSON (requires jackson-databind)
- `StringHttpMessageConverter` — plain text
- `ByteArrayHttpMessageConverter` — binary
- `FormHttpMessageConverter` — form data

---

## 6. REST API Best Practices

### HTTP Method Idempotency

| Method | Idempotent | Safe | Body |
|---|---|---|---|
| GET | Yes | Yes | No |
| PUT | Yes | No | Yes |
| DELETE | Yes | No | No |
| POST | No | No | Yes |
| PATCH | No | No | Yes |

### API Versioning Strategies

```java
// 1. URI versioning (most common, cache-friendly)
@GetMapping("/api/v1/users")

// 2. Header versioning
@GetMapping(value = "/users", headers = "X-API-VERSION=1")

// 3. Accept header (content negotiation)
@GetMapping(value = "/users", produces = "application/vnd.app.v1+json")
```

### Global Exception Handling

```java
@ControllerAdvice
@Slf4j
public class GlobalExceptionHandler extends ResponseEntityExceptionHandler {

    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ErrorResponse> handleNotFound(
            ResourceNotFoundException ex, HttpServletRequest request) {
        log.warn("Resource not found: {}", ex.getMessage());
        ErrorResponse error = new ErrorResponse(
            HttpStatus.NOT_FOUND.value(), ex.getMessage(), request.getRequestURI());
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
    }

    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ConstraintViolationException ex) {
        Map<String, String> errors = ex.getConstraintViolations().stream()
            .collect(Collectors.toMap(
                v -> v.getPropertyPath().toString(),
                ConstraintViolation::getMessage));
        return ResponseEntity.badRequest().body(new ValidationErrorResponse(errors));
    }

    @Override
    protected ResponseEntity<Object> handleMethodArgumentNotValid(
            MethodArgumentNotValidException ex, ...) {
        // Override for @Valid failures on request body
    }
}
```

### Pagination Best Practices

```java
@GetMapping("/users")
public Page<UserDTO> getUsers(
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "20") int size,
        @RequestParam(defaultValue = "createdAt,desc") String[] sort) {

    Sort sortObj = Sort.by(Arrays.stream(sort)
        .map(s -> s.split(","))
        .map(a -> new Sort.Order(Sort.Direction.fromString(a[1]), a[0]))
        .toList());

    return userService.findAll(PageRequest.of(page, size, sortObj))
                      .map(userMapper::toDTO);
}
```

> **Production Tip:** For large datasets, use cursor-based pagination (keyset pagination) instead of offset-based — offset becomes O(n) at large pages.

### API Documentation — SpringDoc / OpenAPI 3

```java
@Operation(summary = "Get user by ID", description = "Returns a single user")
@ApiResponse(responseCode = "200", description = "User found")
@ApiResponse(responseCode = "404", description = "User not found")
@GetMapping("/users/{id}")
public UserDTO getUser(@Parameter(description = "User ID") @PathVariable Long id) { ... }
```

---

## 7. Spring Data JPA Deep Dive

### Persistence Context & Dirty Checking

```
EntityManager = Persistence Context
  → Tracks all "managed" entities
  → On transaction commit: compares current state to snapshot
  → Auto-generates UPDATE if changed (dirty checking)
  → Uses byte-code instrumentation or reflection
```

```java
@Transactional
public void updateUser(Long id, String newName) {
    User user = userRepository.findById(id).orElseThrow();
    user.setName(newName); // No save() needed — dirty checking handles it
}
```

### @Transactional Internals

```java
@Transactional(
    propagation = Propagation.REQUIRED,      // default
    isolation = Isolation.READ_COMMITTED,    // default in most DBs
    readOnly = false,
    rollbackFor = Exception.class,           // defaults to RuntimeException
    timeout = 30
)
```

### Propagation Types

| Propagation | Behavior |
|---|---|
| `REQUIRED` | Use existing TX or create new |
| `REQUIRES_NEW` | Always create new TX, suspend existing |
| `NESTED` | Nested within existing TX (savepoint) |
| `SUPPORTS` | Use existing if present, else non-TX |
| `NOT_SUPPORTED` | Suspend existing TX, run non-TX |
| `MANDATORY` | Must have existing TX, else exception |
| `NEVER` | Must NOT have TX, else exception |

### Isolation Levels

| Level | Dirty Read | Non-Repeatable Read | Phantom Read |
|---|---|---|---|
| `READ_UNCOMMITTED` | Possible | Possible | Possible |
| `READ_COMMITTED` | Safe | Possible | Possible |
| `REPEATABLE_READ` | Safe | Safe | Possible |
| `SERIALIZABLE` | Safe | Safe | Safe |

> **Common Mistake:** `@Transactional` only works on public methods via proxy. Self-invocation (calling `this.method()`) bypasses the proxy — use `@Autowired` self-reference or `AopContext.currentProxy()`.

### N+1 Problem

```java
// BAD — N+1 queries
List<Order> orders = orderRepository.findAll();
orders.forEach(o -> o.getItems().size()); // Each triggers a SELECT

// GOOD — Fetch join
@Query("SELECT o FROM Order o LEFT JOIN FETCH o.items WHERE o.status = :status")
List<Order> findWithItems(@Param("status") Status status);

// GOOD — Entity graph
@EntityGraph(attributePaths = {"items", "customer"})
List<Order> findByStatus(Status status);
```

### First-Level Cache (Session Cache)

```java
@Transactional
public void demo() {
    User u1 = repo.findById(1L).get(); // SELECT
    User u2 = repo.findById(1L).get(); // No SQL — from L1 cache
    // u1 == u2 (same object reference)
}
```

### Lazy vs Eager Loading

```java
@ManyToOne(fetch = FetchType.LAZY)   // Default for @ManyToOne is EAGER — override it!
private Department department;

@OneToMany(fetch = FetchType.LAZY)   // Default for @OneToMany is LAZY — keep it
private List<Item> items;
```

> **Production Tip:** Always use `LAZY` loading and explicitly fetch what you need. `EAGER` loading causes unnecessary JOINs for every query on the entity.

---

## 8. Hibernate Performance Optimization

### Batch Processing

```yaml
spring:
  jpa:
    properties:
      hibernate:
        jdbc:
          batch_size: 50            # Batch INSERTs/UPDATEs
        order_inserts: true         # Group same-type inserts
        order_updates: true
        generate_statistics: true   # Dev only
```

```java
@Transactional
public void bulkInsert(List<Entity> entities) {
    for (int i = 0; i < entities.size(); i++) {
        entityManager.persist(entities.get(i));
        if (i % 50 == 0) {
            entityManager.flush();
            entityManager.clear(); // Prevent OOM from L1 cache
        }
    }
}
```

### Connection Pool — HikariCP

```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20         # CPU cores * 2 + disk spindles (Brenczyc formula)
      minimum-idle: 5
      idle-timeout: 300000          # 5 min
      connection-timeout: 30000     # 30s — fail fast
      max-lifetime: 1800000         # 30 min — recycle connections
      leak-detection-threshold: 60000
```

> **Production Tip:** Pool size formula: `connections = (core_count * 2) + effective_spindle_count`. For cloud DBs (1 spindle), a 4-core machine → pool of 9.

### SQL Logging (Dev Only)

```yaml
spring:
  jpa:
    show-sql: false  # Use datasource-proxy instead in production
logging:
  level:
    org.hibernate.SQL: DEBUG
    org.hibernate.type.descriptor.sql.BasicBinder: TRACE
```

### Query Optimization

```java
// Use projections to avoid loading full entities
public interface UserSummary {
    String getName();
    String getEmail();
}
List<UserSummary> findAllProjectedBy();

// Native query for complex reporting
@Query(value = "SELECT * FROM users WHERE ...", nativeQuery = true)
```

---

## 9. Spring Security Advanced

### Security Filter Chain (Key Filters in Order)

```
SecurityContextPersistenceFilter
  → UsernamePasswordAuthenticationFilter (form login)
  → BearerTokenAuthenticationFilter (JWT/OAuth2)
  → BasicAuthenticationFilter
  → RememberMeAuthenticationFilter
  → AnonymousAuthenticationFilter
  → ExceptionTranslationFilter
  → FilterSecurityInterceptor (authorization)
```

### JWT Authentication Flow

```
1. POST /auth/login → UsernamePasswordAuthenticationToken
2. AuthenticationManager.authenticate()
3. UserDetailsService.loadUserByUsername()
4. PasswordEncoder.matches()
5. JWT generated → returned to client
--- subsequent requests ---
6. Bearer token extracted from header
7. JwtAuthenticationFilter validates token
8. SecurityContextHolder.getContext().setAuthentication(...)
9. Downstream method security works
```

```java
@Bean
public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
    return http
        .csrf(AbstractHttpConfigurer::disable)          // Stateless JWT = no CSRF needed
        .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/auth/**", "/actuator/health").permitAll()
            .requestMatchers("/api/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
        )
        .addFilterBefore(jwtAuthFilter, UsernamePasswordAuthenticationFilter.class)
        .build();
}
```

### Method Security

```java
@EnableMethodSecurity  // Enables @PreAuthorize, @PostAuthorize, @Secured
public class SecurityConfig { }

@PreAuthorize("hasRole('ADMIN') or #userId == authentication.principal.id")
public UserDTO getUser(Long userId) { ... }

@PostAuthorize("returnObject.ownerId == authentication.principal.id")
public Document getDocument(Long id) { ... }
```

### OAuth2 Resource Server

```yaml
spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: https://auth.example.com
          jwk-set-uri: https://auth.example.com/.well-known/jwks.json
```

### CORS Configuration

```java
@Bean
public CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration config = new CorsConfiguration();
    config.setAllowedOriginPatterns(List.of("https://*.example.com"));
    config.setAllowedMethods(List.of("GET", "POST", "PUT", "DELETE", "OPTIONS"));
    config.setAllowedHeaders(List.of("Authorization", "Content-Type"));
    config.setAllowCredentials(true);
    config.setMaxAge(3600L);

    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/api/**", config);
    return source;
}
```

> **Interview Note:** CSRF protection is needed for cookie-based auth. Stateless JWT with `Authorization: Bearer` header does not need CSRF protection (cookies not used).

---

## 10. Microservices with Spring Boot

### Architecture Overview

```
Client → API Gateway (Spring Cloud Gateway)
              ↓
        Service Discovery (Eureka)
              ↓
   ┌──────────────────────────────┐
   │  Service A  │  Service B    │
   │  (Feign)    │  (Kafka)      │
   └─────────────────────────────┘
         ↓                ↓
    Config Server    Distributed Tracing (Zipkin/Tempo)
```

### Service Discovery — Eureka

```yaml
# eureka-server
eureka:
  client:
    register-with-eureka: false
    fetch-registry: false
```

```yaml
# microservice
eureka:
  client:
    service-url:
      defaultZone: http://eureka:8761/eureka
  instance:
    prefer-ip-address: true
    lease-renewal-interval-in-seconds: 10
```

### OpenFeign Client

```java
@FeignClient(name = "user-service", fallbackFactory = UserClientFallbackFactory.class)
public interface UserServiceClient {
    @GetMapping("/api/users/{id}")
    UserDTO getUser(@PathVariable Long id);
}

@Component
public class UserClientFallbackFactory implements FallbackFactory<UserServiceClient> {
    @Override
    public UserServiceClient create(Throwable cause) {
        return id -> {
            log.error("User service unavailable: {}", cause.getMessage());
            return UserDTO.empty();
        };
    }
}
```

### Circuit Breaker — Resilience4j

```java
@CircuitBreaker(name = "userService", fallbackMethod = "fallback")
@Retry(name = "userService")
@Bulkhead(name = "userService")
@TimeLimiter(name = "userService")
public CompletableFuture<UserDTO> getUser(Long id) {
    return CompletableFuture.supplyAsync(() -> userClient.getUser(id));
}

private CompletableFuture<UserDTO> fallback(Long id, Exception ex) {
    return CompletableFuture.completedFuture(UserDTO.empty());
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      userService:
        sliding-window-size: 10
        failure-rate-threshold: 50
        wait-duration-in-open-state: 10s
        permitted-number-of-calls-in-half-open-state: 3
```

### Circuit Breaker States

```
CLOSED (normal) → [failure rate > threshold] → OPEN (all calls fail-fast)
     ↑                                               ↓
     └─── [test calls succeed] ← HALF_OPEN ← [wait-duration elapsed]
```

### Distributed Tracing

```yaml
# Micrometer Tracing + Zipkin (Boot 3.x)
management:
  tracing:
    sampling:
      probability: 1.0  # 100% in dev, 0.1 in prod
```

Every request gets `traceId` (per request) + `spanId` (per service hop) — correlate logs across services.

### API Gateway — Spring Cloud Gateway

```yaml
spring:
  cloud:
    gateway:
      routes:
        - id: user-service
          uri: lb://user-service
          predicates:
            - Path=/api/users/**
          filters:
            - name: CircuitBreaker
              args:
                name: userService
                fallbackUri: forward:/fallback
            - name: RequestRateLimiter
              args:
                redis-rate-limiter.replenishRate: 100
                redis-rate-limiter.burstCapacity: 200
```

---

## 11. Spring Boot Actuator

### Key Endpoints

| Endpoint | Description |
|---|---|
| `/actuator/health` | App health (DB, disk, custom) |
| `/actuator/metrics` | Micrometer metrics |
| `/actuator/info` | App info (git, build) |
| `/actuator/env` | Environment properties |
| `/actuator/beans` | All registered beans |
| `/actuator/httptrace` | Recent HTTP requests |
| `/actuator/loggers` | Change log levels at runtime |
| `/actuator/threaddump` | Thread dump |
| `/actuator/heapdump` | Heap dump download |
| `/actuator/prometheus` | Prometheus-format metrics |

```yaml
management:
  endpoints:
    web:
      exposure:
        include: health,info,metrics,prometheus
  endpoint:
    health:
      show-details: when-authorized
      probes:
        enabled: true  # /health/liveness, /health/readiness for K8s
  info:
    git:
      mode: full
```

### Custom Health Indicator

```java
@Component
public class ExternalApiHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        try {
            // Check external dependency
            boolean up = externalApi.ping();
            return up ? Health.up().withDetail("api", "reachable").build()
                      : Health.down().withDetail("api", "unreachable").build();
        } catch (Exception e) {
            return Health.down(e).build();
        }
    }
}
```

### Prometheus + Grafana Integration

```yaml
management:
  metrics:
    export:
      prometheus:
        enabled: true
    tags:
      application: ${spring.application.name}
      environment: ${spring.profiles.active}
```

```java
// Custom metric
@Autowired
private MeterRegistry meterRegistry;

Counter.builder("orders.processed")
    .tag("type", orderType)
    .description("Orders processed count")
    .register(meterRegistry)
    .increment();

Timer.builder("api.latency")
    .register(meterRegistry)
    .record(() -> processRequest());
```

---

## 12. Spring Boot Logging

### Logback Configuration

```xml
<!-- logback-spring.xml -->
<configuration>
    <springProfile name="prod">
        <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder"/>
        </appender>
        <root level="INFO">
            <appender-ref ref="JSON"/>
        </root>
    </springProfile>

    <springProfile name="!prod">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss} [%thread] %-5level [%X{traceId},%X{spanId}] %logger{36} - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="DEBUG">
            <appender-ref ref="CONSOLE"/>
        </root>
    </springProfile>
</configuration>
```

### MDC for Correlation IDs

```java
@Component
public class MDCFilter implements Filter {
    @Override
    public void doFilter(ServletRequest req, ServletResponse res, FilterChain chain)
            throws IOException, ServletException {
        String traceId = Optional.ofNullable(((HttpServletRequest)req).getHeader("X-Trace-Id"))
                                 .orElse(UUID.randomUUID().toString());
        MDC.put("traceId", traceId);
        MDC.put("userId", extractUserId(req));
        try {
            chain.doFilter(req, res);
        } finally {
            MDC.clear(); // Critical — thread pool reuse
        }
    }
}
```

> **Production Tip:** Always `MDC.clear()` in a `finally` block. Thread pool reuse means MDC values leak between requests if not cleared.

---

## 13. Caching in Spring Boot

### Cache Annotations

```java
@EnableCaching
public class CacheConfig { }

@Cacheable(value = "users", key = "#id", condition = "#id > 0",
           unless = "#result == null")
public User getUser(Long id) { ... }

@CachePut(value = "users", key = "#user.id")
public User updateUser(User user) { ... }

@CacheEvict(value = "users", key = "#id")
public void deleteUser(Long id) { ... }

@CacheEvict(value = "users", allEntries = true)
@Scheduled(fixedRate = 3600000)  // Refresh entire cache hourly
public void evictAll() { }
```

### Redis Cache Configuration

```java
@Bean
public RedisCacheConfiguration redisCacheConfiguration() {
    return RedisCacheConfiguration.defaultCacheConfig()
        .entryTtl(Duration.ofMinutes(60))
        .serializeKeysWith(RedisSerializationContext.SerializationPair
            .fromSerializer(new StringRedisSerializer()))
        .serializeValuesWith(RedisSerializationContext.SerializationPair
            .fromSerializer(new GenericJackson2JsonRedisSerializer()))
        .disableCachingNullValues();
}

@Bean
public CacheManager cacheManager(RedisConnectionFactory factory) {
    Map<String, RedisCacheConfiguration> configs = Map.of(
        "users",    RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofHours(1)),
        "sessions", RedisCacheConfiguration.defaultCacheConfig().entryTtl(Duration.ofMinutes(30))
    );
    return RedisCacheManager.builder(factory)
        .cacheDefaults(redisCacheConfiguration())
        .withInitialCacheConfigurations(configs)
        .build();
}
```

### Cache Strategies

| Strategy | Description | Use Case |
|---|---|---|
| Cache-aside | App checks cache, then DB | Read-heavy, flexible |
| Write-through | Write to cache + DB together | Consistency critical |
| Write-behind | Write to cache, async to DB | Write-heavy |
| Refresh-ahead | Proactively refresh before expiry | Predictable access |

> **Common Mistake:** Caching mutable entities without proper eviction leads to stale data. Always define TTL and eviction strategy explicitly.

---

## 14. Messaging Systems

### Kafka with Spring Boot

```java
// Producer
@Service
public class EventPublisher {
    @Autowired
    private KafkaTemplate<String, Object> kafkaTemplate;

    public void publish(String topic, String key, Object event) {
        kafkaTemplate.send(topic, key, event)
            .whenComplete((result, ex) -> {
                if (ex == null) {
                    log.info("Sent to partition {} offset {}",
                        result.getRecordMetadata().partition(),
                        result.getRecordMetadata().offset());
                } else {
                    log.error("Failed to send: {}", ex.getMessage());
                }
            });
    }
}

// Consumer
@KafkaListener(
    topics = "orders",
    groupId = "order-processor",
    containerFactory = "kafkaListenerContainerFactory"
)
public void consume(ConsumerRecord<String, OrderEvent> record,
                    Acknowledgment acknowledgment) {
    try {
        processOrder(record.value());
        acknowledgment.acknowledge(); // Manual commit
    } catch (RecoverableException e) {
        // Will be retried
        throw e;
    } catch (Exception e) {
        // Send to DLT
        kafkaTemplate.send("orders.DLT", record.key(), record.value());
        acknowledgment.acknowledge();
    }
}
```

```yaml
spring:
  kafka:
    bootstrap-servers: kafka1:9092,kafka2:9092
    producer:
      acks: all              # Wait for all replicas
      retries: 3
      batch-size: 16384
      compression-type: snappy
    consumer:
      auto-offset-reset: earliest
      enable-auto-commit: false  # Manual ack for reliability
      max-poll-records: 500
    listener:
      ack-mode: manual_immediate
```

### Kafka Key Concepts

| Concept | Details |
|---|---|
| Partition | Unit of parallelism; messages with same key → same partition |
| Consumer Group | Each partition consumed by one consumer in group |
| Offset | Position in partition; committed manually or auto |
| ISR | In-Sync Replicas; `acks=all` waits for all ISR |
| DLT | Dead Letter Topic for unprocessable messages |

### RabbitMQ

```java
@RabbitListener(queues = "order.queue")
public void consume(OrderEvent event, Channel channel,
                    @Header(AmqpHeaders.DELIVERY_TAG) long tag) throws IOException {
    try {
        processOrder(event);
        channel.basicAck(tag, false);
    } catch (Exception e) {
        channel.basicNack(tag, false, true); // requeue
    }
}
```

### Retry with Dead Letter Exchange (RabbitMQ)

```
order.queue → failure → order.retry.queue (TTL) → order.queue
           → max retries exceeded → order.dead.queue
```

---

## 15. Scheduling & Async

### @Scheduled

```java
@EnableScheduling
@Configuration
public class SchedulingConfig implements SchedulingConfigurer {
    @Override
    public void configureTasks(ScheduledTaskRegistrar registrar) {
        registrar.setTaskScheduler(taskScheduler());
    }

    @Bean
    public TaskScheduler taskScheduler() {
        ThreadPoolTaskScheduler scheduler = new ThreadPoolTaskScheduler();
        scheduler.setPoolSize(10);
        scheduler.setThreadNamePrefix("scheduler-");
        scheduler.setErrorHandler(t -> log.error("Scheduler error", t));
        return scheduler;
    }
}

@Scheduled(cron = "0 0 * * * *")           // Top of every hour
@Scheduled(fixedRate = 5000)               // Every 5s (overlap possible)
@Scheduled(fixedDelay = 5000)              // 5s after last completion
@Scheduled(fixedDelay = 5000, initialDelay = 30000)
```

> **Common Mistake:** Default scheduler has only 1 thread. All scheduled tasks run sequentially. Always configure a thread pool.

### @Async

```java
@EnableAsync
@Configuration
public class AsyncConfig implements AsyncConfigurer {
    @Override
    public Executor getAsyncExecutor() {
        ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
        executor.setCorePoolSize(10);
        executor.setMaxPoolSize(50);
        executor.setQueueCapacity(100);
        executor.setThreadNamePrefix("async-");
        executor.setRejectedExecutionHandler(new ThreadPoolExecutor.CallerRunsPolicy());
        executor.initialize();
        return executor;
    }

    @Override
    public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
        return (ex, method, params) ->
            log.error("Async exception in {}: {}", method.getName(), ex.getMessage());
    }
}

@Async
public CompletableFuture<Result> processAsync(Data data) {
    // Runs in async executor thread pool
    return CompletableFuture.completedFuture(process(data));
}
```

### CompletableFuture Composition

```java
CompletableFuture<UserDTO> user = userService.getUserAsync(userId);
CompletableFuture<List<Order>> orders = orderService.getOrdersAsync(userId);

CompletableFuture<Dashboard> dashboard = user
    .thenCombine(orders, (u, o) -> new Dashboard(u, o))
    .exceptionally(ex -> Dashboard.empty());

// Wait for multiple
CompletableFuture.allOf(user, orders).join();
```

---

## 16. Spring Boot Testing

### Test Slices (Performance-focused)

| Annotation | Loads | Use For |
|---|---|---|
| `@SpringBootTest` | Full context | Integration tests |
| `@WebMvcTest` | MVC layer only | Controller tests |
| `@DataJpaTest` | JPA + H2 | Repository tests |
| `@DataRedisTest` | Redis layer | Redis repository tests |
| `@RestClientTest` | RestTemplate/WebClient | REST client tests |
| `@JsonTest` | JSON serialization | Jackson tests |

```java
// Controller test with MockMvc
@WebMvcTest(UserController.class)
class UserControllerTest {
    @Autowired MockMvc mockMvc;
    @MockBean UserService userService;

    @Test
    void getUser_returnsUser() throws Exception {
        given(userService.findById(1L)).willReturn(new UserDTO(1L, "Alice"));

        mockMvc.perform(get("/api/users/1")
                .header("Authorization", "Bearer test-token"))
            .andExpect(status().isOk())
            .andExpect(jsonPath("$.name").value("Alice"))
            .andDo(print());
    }
}
```

```java
// Integration test with TestContainers
@SpringBootTest(webEnvironment = WebEnvironment.RANDOM_PORT)
@Testcontainers
class OrderIntegrationTest {
    @Container
    static PostgreSQLContainer<?> postgres = new PostgreSQLContainer<>("postgres:15")
        .withDatabaseName("testdb");

    @Container
    static KafkaContainer kafka = new KafkaContainer(DockerImageName.parse("confluentinc/cp-kafka:7.4.0"));

    @DynamicPropertySource
    static void properties(DynamicPropertyRegistry registry) {
        registry.add("spring.datasource.url", postgres::getJdbcUrl);
        registry.add("spring.kafka.bootstrap-servers", kafka::getBootstrapServers);
    }
}
```

> **Production Tip:** Use `@DirtiesContext` sparingly — it restarts the Spring context, making tests slow. Prefer `@Transactional` for rollback or Testcontainers for isolation.

---

## 17. Production Deployment

### Docker

```dockerfile
# Multi-stage, layered build
FROM eclipse-temurin:21-jdk-alpine AS builder
WORKDIR /app
COPY . .
RUN ./mvnw package -DskipTests

FROM eclipse-temurin:21-jre-alpine
WORKDIR /app
# Spring Boot layered jar (Boot 2.3+)
RUN java -Djarmode=layertools -jar /app/target/app.jar extract
COPY --from=builder /app/target/dependencies/ ./
COPY --from=builder /app/target/spring-boot-loader/ ./
COPY --from=builder /app/target/snapshot-dependencies/ ./
COPY --from=builder /app/target/application/ ./

ENTRYPOINT ["java", \
  "-XX:+UseContainerSupport", \
  "-XX:MaxRAMPercentage=75.0", \
  "org.springframework.boot.loader.launch.JarLauncher"]
```

### Kubernetes Probes

```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
  initialDelaySeconds: 20
  periodSeconds: 5

startupProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080
  failureThreshold: 30    # 30 * 10s = 5 min startup window
  periodSeconds: 10
```

### Graceful Shutdown

```yaml
server:
  shutdown: graceful

spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s
```

On `SIGTERM`:
1. Stop accepting new requests (readiness probe fails)
2. Complete in-flight requests (up to 30s)
3. Close connections and stop

### Blue-Green Deployment

```
v1 (Blue, live) ←→ Load Balancer ←→ v2 (Green, staging)
                         ↓
                   Switch traffic to Green
                         ↓
                Blue idle (rollback available)
```

---

## 18. JVM & Performance Tuning

### Memory Structure

```
JVM Process Memory
  ├── Heap
  │    ├── Young Gen (Eden + S0 + S1) — Minor GC
  │    └── Old Gen (Tenured) — Major GC
  ├── Metaspace (class metadata, unlimited by default)
  ├── Code Cache (JIT compiled)
  ├── Direct Memory (NIO ByteBuffers)
  └── Thread Stacks
```

### Key JVM Flags

```bash
# Heap sizing
-Xms2g -Xmx2g           # Set equal to prevent resizing
-XX:MaxRAMPercentage=75.0  # Container-friendly

# GC selection
-XX:+UseG1GC             # Default in JDK 9+, good general purpose
-XX:+UseZGC              # Low-latency (<10ms pauses), JDK 15+
-XX:+UseShenandoahGC     # Red Hat, low-latency

# G1 tuning
-XX:MaxGCPauseMillis=200
-XX:G1HeapRegionSize=16m

# Diagnostics
-XX:+HeapDumpOnOutOfMemoryError
-XX:HeapDumpPath=/var/log/heapdump.hprof
-Xss512k                 # Stack size per thread (reduce if many threads)
```

### GC Log Analysis

```bash
-Xlog:gc*:file=/var/log/gc.log:time,uptime,level,tags:filecount=5,filesize=20m
```

### Profiling Tools

| Tool | Use Case |
|---|---|
| JProfiler / YourKit | CPU + memory profiling |
| Async-profiler | Low-overhead flame graphs |
| JFR (Java Flight Recorder) | Production profiling (JDK built-in) |
| JMC (Java Mission Control) | Analyze JFR recordings |
| VisualVM | Thread/heap monitoring |

```bash
# JFR — zero overhead production profiling
jcmd <pid> JFR.start duration=60s filename=/tmp/recording.jfr
```

> **Production Tip:** For containerized apps, always use `-XX:+UseContainerSupport` (JDK 10+) and percentage-based heap (`-XX:MaxRAMPercentage=75`) instead of absolute `-Xmx` values.

---

## 19. Common Production Issues

### Memory Leak Patterns

| Symptom | Likely Cause |
|---|---|
| Metaspace OOM | Class loader leak (app redeploy without restart) |
| Heap OOM | Entity cache, static collection growing, listener not removed |
| Direct OOM | NIO buffer leak, Netty direct memory |
| Thread leak | Unmanaged thread creation, scheduled task creating threads |

**Diagnosis:**
```bash
# Heap dump analysis
jmap -dump:live,format=b,file=heap.hprof <pid>
# Analyze with Eclipse MAT or VisualVM

# Thread dump
jstack <pid> > thread.dump
# Look for: BLOCKED threads, deadlocks, excessive RUNNABLE
```

### DB Connection Exhaustion

```
Symptoms: HikariCP timeout, connection pool full
Root causes:
  - Long-running transactions
  - Transaction not closed (missing @Transactional)
  - Slow queries blocking connections
  - N+1 queries saturating pool
```

```yaml
# Enable leak detection
spring.datasource.hikari.leak-detection-threshold: 30000  # 30s
```

### High CPU Usage

**Common causes:**
- Infinite loop / tight spin lock
- GC pressure (tune heap or fix memory leak)
- Regex backtracking on large inputs
- JSON deserialization of huge payloads
- Too many threads causing context switching

### Deadlock Detection

```bash
# Thread dump shows:
"Thread-A" BLOCKED on lock A, waiting for lock B
"Thread-B" BLOCKED on lock B, waiting for lock A
# → Deadlock

# Prevention: consistent lock ordering, use tryLock with timeout
```

### Troubleshooting Approach (MECE)

```
1. Identify: metrics spike, logs, alerts
2. Isolate: which service/pod/endpoint
3. Characterize: memory/CPU/IO/network
4. Reproduce: test env if possible
5. Root cause: code, config, data
6. Fix + verify
7. Post-mortem + prevent recurrence
```

---

## 20. Design Patterns in Spring

### Where Spring Uses Design Patterns

| Pattern | Spring Implementation |
|---|---|
| **Factory** | `BeanFactory`, `ApplicationContext.getBean()` |
| **Singleton** | Default bean scope |
| **Proxy** | AOP (`@Transactional`, `@Cacheable`, `@Async`) via CGLIB/JDK |
| **Template Method** | `JdbcTemplate`, `RestTemplate`, `KafkaTemplate` |
| **Observer** | `ApplicationEventPublisher`, `@EventListener` |
| **Decorator** | `BeanPostProcessor`, `HandlerInterceptor` |
| **Strategy** | `HandlerMapping`, `ViewResolver`, `MessageConverter` |
| **Chain of Responsibility** | `FilterChain`, `HandlerInterceptor` chain |
| **Front Controller** | `DispatcherServlet` |
| **Composite** | `CompositeHealthIndicator` |

### Template Method Pattern Example

```java
// Spring's JdbcTemplate — you supply the variable part
jdbcTemplate.query(sql, ps -> {
    ps.setLong(1, userId);              // PreparedStatementSetter (strategy)
}, (rs, rowNum) -> mapToUser(rs));      // RowMapper (strategy)
// JdbcTemplate handles: connection, statement, result set, exceptions, cleanup
```

### Observer Pattern via Events

```java
// Publisher
applicationEventPublisher.publishEvent(new OrderCreatedEvent(this, order));

// Listener
@EventListener
@Async  // Non-blocking listener
@TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
public void onOrderCreated(OrderCreatedEvent event) {
    emailService.sendConfirmation(event.getOrder());
}
```

---

## 21. Spring Boot Interview Questions

### Intermediate Level

**Q: What is the difference between `@Component`, `@Service`, `@Repository`?**
A: All are specializations of `@Component` and trigger component scanning. `@Repository` additionally enables `PersistenceExceptionTranslation`. `@Service` is semantic — marks business layer. They're all functionally equivalent otherwise.

**Q: How does `@Transactional` work internally?**
A: Spring creates a proxy around the annotated bean. When a transactional method is called, the proxy intercepts it, begins a transaction (or joins existing), calls the real method, then commits or rolls back based on outcome. Uses `PlatformTransactionManager` internally.

**Q: What is the difference between `@RequestParam` and `@PathVariable`?**
A: `@PathVariable` extracts from URI path (`/users/{id}`). `@RequestParam` extracts from query string (`/users?id=1`). `@PathVariable` is mandatory by default; `@RequestParam` can have defaults.

**Q: How to handle circular bean dependencies?**
A: (1) Redesign — usually a design smell. (2) Use `@Lazy` on one of the injections. (3) Switch from constructor to setter injection. (4) Use `ObjectProvider<T>`.

**Q: What is Spring Boot's auto-configuration and how to debug it?**
A: Run with `--debug` flag or set `logging.level.org.springframework.boot.autoconfigure=DEBUG`. The `ConditionEvaluationReport` shows what was/wasn't configured and why.

### Advanced Level

**Q: What is the difference between `PROPAGATION_REQUIRES_NEW` and `PROPAGATION_NESTED`?**
A: `REQUIRES_NEW` suspends the outer TX, starts a fresh independent TX. If inner fails, outer is unaffected. `NESTED` uses a savepoint within the outer TX — inner can roll back to savepoint while outer continues. `NESTED` is only supported by JDBC, not JTA.

**Q: Explain the N+1 problem and all solutions.**
A: Loading a list of entities then accessing a lazy collection triggers N additional queries. Solutions: (1) `JOIN FETCH` in JPQL, (2) `@EntityGraph`, (3) `@BatchSize` on the collection, (4) DTO projections with a JOIN query, (5) Hibernate's `@FetchProfile`.

**Q: How does Spring Security's filter chain work? How do you add a custom filter?**
A: Security is a chain of `GenericFilterBean` implementations. Each filter can process the request, pass it to the next, or short-circuit. Custom filter added via `addFilterBefore()` / `addFilterAfter()` in `SecurityFilterChain`.

**Q: How does `@Async` work? What are its pitfalls?**
A: Creates a proxy that submits the method call to a thread pool. Pitfalls: (1) self-invocation bypasses proxy, (2) exceptions are swallowed unless `AsyncUncaughtExceptionHandler` is set or the method returns `Future<T>`, (3) `SecurityContext` is not propagated by default (configure `DelegatingSecurityContextAsyncTaskExecutor`).

**Q: Explain Kafka consumer group rebalancing.**
A: When consumers join/leave a group or partitions change, Kafka triggers rebalancing — all consumers pause, coordinator assigns partitions to consumers. During rebalance, no messages are consumed. Minimize rebalance with: `max.poll.interval.ms` tuning, static group membership (`group.instance.id`), cooperative sticky assignor.

### Architect Level

**Q: How would you design a rate limiter for a microservices API gateway?**
A: Token bucket or sliding window algorithm. Use Redis with Lua scripts for atomic operations across instances. Spring Cloud Gateway has `RequestRateLimiter` filter backed by Redis. Track per user/IP/API key. Return `429 Too Many Requests` with `Retry-After` header.

**Q: How would you handle distributed transactions across microservices?**
A: Avoid 2PC (blocking, availability risk). Use SAGA pattern: (1) Choreography — each service publishes events, others react (Kafka), eventually consistent. (2) Orchestration — a saga orchestrator drives the workflow, issues compensating transactions on failure.

**Q: What are the tradeoffs between synchronous (REST/Feign) and asynchronous (Kafka) service communication?**
A: Synchronous: simpler, immediate response, higher coupling, cascading failures risk, tight latency dependency. Asynchronous: decoupled, resilient, higher throughput, eventual consistency, harder to debug/trace, need DLT for failures.

**Q: How do you approach performance optimization of a slow Spring Boot API?**
A: (1) Measure — identify bottleneck (DB? network? CPU?). (2) DB — fix N+1, add indexes, use projections, enable caching. (3) App — async processing, thread pool tuning, eliminate blocking calls. (4) JVM — GC tuning, reduce allocations. (5) Architecture — CDN, caching layer, read replicas.

---

## 22. Spring Boot Cheatsheet

### Important Annotations Quick Reference

```
@SpringBootApplication    = @Configuration + @ComponentScan + @EnableAutoConfiguration
@RestController           = @Controller + @ResponseBody
@Transactional            = AOP proxy, TX management
@Cacheable                = AOP proxy, cache-aside
@Async                    = AOP proxy, thread pool submit
@Scheduled                = Scheduler thread pool
@EventListener            = ApplicationContext event handling
@ConditionalOnProperty    = Conditional bean creation
@ConfigurationProperties  = Typed externalized config
@ControllerAdvice         = Global exception/model handling
@EntityGraph              = JPA fetch strategy override
@Modifying + @Query       = JPA update/delete queries
@Lock                     = JPA pessimistic/optimistic locking
```

### Common Commands

```bash
# Build
./mvnw clean package -DskipTests
./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Docker
docker build -t myapp:latest .
docker run -e SPRING_PROFILES_ACTIVE=prod -p 8080:8080 myapp

# Actuator
curl localhost:8080/actuator/health
curl localhost:8080/actuator/metrics/jvm.memory.used
curl -X POST localhost:8080/actuator/loggers/com.example -H 'Content-Type: application/json' \
     -d '{"configuredLevel":"DEBUG"}'  # Change log level at runtime

# JVM diagnostics
jstack <pid>                           # Thread dump
jmap -histo <pid>                      # Heap histogram
jcmd <pid> VM.native_memory            # Native memory
```

### Debugging Tips

```yaml
# Debug auto-configuration
logging:
  level:
    org.springframework.boot.autoconfigure: DEBUG

# Show SQL
spring.jpa.show-sql: true
logging.level.org.hibernate.SQL: DEBUG

# Show bean wiring
logging.level.org.springframework: TRACE

# Actuator debug
management.endpoint.conditions.enabled: true
# GET /actuator/conditions
```

### Performance Checklist

- [ ] HikariCP pool size tuned (not default 10)
- [ ] `@Transactional(readOnly=true)` on read-only methods
- [ ] No `EAGER` fetch types on collections
- [ ] N+1 queries eliminated (use JOIN FETCH or @EntityGraph)
- [ ] Database indexes on filtered/sorted columns
- [ ] Caching on expensive reads (Redis with TTL)
- [ ] Async processing for non-critical paths
- [ ] Thread pools configured explicitly (not default 1-thread scheduler)
- [ ] JVM heap % based in containers
- [ ] GC selected appropriately (ZGC for low latency)

### Security Checklist

- [ ] HTTPS enforced (redirect HTTP to HTTPS)
- [ ] JWT secrets rotated, stored in vault (not in properties file)
- [ ] Password encoder: `BCryptPasswordEncoder` with strength 12+
- [ ] SQL injection: use parameterized queries (JPA/JDBC template)
- [ ] XSS: sanitize inputs, CSP headers configured
- [ ] CORS: whitelist specific origins, not `*`
- [ ] Actuator endpoints secured (not exposed publicly)
- [ ] Dependencies scanned for CVEs (`./mvnw dependency-check:check`)
- [ ] Secrets in environment variables or Vault, never in code/git
- [ ] Rate limiting on auth endpoints

### Common Production Mistakes

| Mistake | Fix |
|---|---|
| `@Transactional` on private method | Make it public |
| Self-invocation for `@Cacheable` / `@Async` | Inject self or refactor |
| Not clearing MDC in finally block | Always `MDC.clear()` in finally |
| Eager loading collections | Use `FetchType.LAZY` always |
| Hardcoding secrets in `application.properties` | Use environment vars or Vault |
| Single-threaded scheduler | Configure `ThreadPoolTaskScheduler` |
| Not setting `max-lifetime` in HikariCP | Set < DB connection timeout |
| Missing `spring.jpa.open-in-view=false` | Disable OSIV in production |
| `SpringBootTest` for all tests | Use sliced tests (`@WebMvcTest`, etc.) |
| Not configuring graceful shutdown | `server.shutdown: graceful` |

---

*Spring Boot Advanced Notes — Updated for Spring Boot 3.x / Spring 6 / Java 21*
