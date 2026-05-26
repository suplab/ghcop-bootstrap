---
applyTo: "**/src/main/java/**/*.java"
---

## Context

This instruction file applies to all Spring Boot Java source files. The project uses Spring Boot 3.x as its baseline, which requires Java 17+ and the Jakarta EE namespace (`jakarta.*`). New microservices target Java 21 and should leverage virtual threads and modern language features where appropriate. This context layer does not apply to legacy Spring MVC modules — see `java-legacy.instructions.md` for those.

---

## Coding Standards

- **Java version baseline:** Java 17 minimum; prefer Java 21 features where the runtime supports them
- **Namespace:** Always `jakarta.*` — never `javax.*` in Spring Boot 3.x code
- **Records over POJOs:** Use Java records for immutable DTOs and value objects; reserve classes for mutable entities
- **Text blocks:** Use `"""..."""` for multi-line SQL, JSON, or HTML strings
- **Pattern matching:** Use `instanceof` pattern matching (`if (obj instanceof MyType t)`) — no explicit cast
- **Sealed types:** Use sealed interfaces/classes to model closed domain hierarchies
- **Switch expressions:** Use `switch` expressions with arrow syntax over `switch` statements
- **Constructor injection only:** All Spring-managed beans use constructor injection; mark injected fields `final`
- **No field `@Autowired`:** Never inject via `@Autowired` on a field — it hides dependencies and breaks testability
- **`@ConfigurationProperties`:** All externalized config must use a typed `@ConfigurationProperties` class — no `@Value` on individual fields for grouped config
- **`application.yml`:** Always YAML, never `.properties` format — structure config hierarchically

---

## Google Java Style Guide

All Java code must conform to the **Google Java Style Guide** (https://google.github.io/styleguide/javaguide.html). Key rules:

- **Indentation:** 2 spaces (not 4, not tabs) for Google style; continuation lines indented +4
- **Column limit:** 100 characters per line
- **Braces:** Egyptian style — opening brace on the same line, always use braces even for single-line `if`/`for`/`while` blocks
- **Blank lines:** One blank line between class members; no blank lines after opening brace or before closing brace
- **Import ordering:** Static imports first, then grouped by package, no wildcard imports
- **Variable declarations:** One declaration per line; `var` is allowed for local variables where the type is obvious
- **Annotations:** One annotation per line for class/method declarations
- **Javadoc:** All `public` and `protected` members must have Javadoc; use `@param`, `@return`, `@throws`
- **Naming:** See the naming rules in the global `copilot-instructions.md`; additionally: acronyms treated as words (`HttpUrl`, not `HTTPUrl`)
- **`@Override`:** Always include `@Override` when a method overrides or implements

```java
// CORRECT: Google Java Style
public final class OrderService {

  private static final Logger log = LoggerFactory.getLogger(OrderService.class);

  private final OrderRepository orderRepository;
  private final OrderMapper orderMapper;

  public OrderService(OrderRepository orderRepository, OrderMapper orderMapper) {
    this.orderRepository = orderRepository;
    this.orderMapper = orderMapper;
  }

  /**
   * Retrieves an order by its unique identifier.
   *
   * @param id the order UUID
   * @return the order response DTO
   * @throws OrderNotFoundException if no order exists with the given ID
   */
  public OrderResponse findById(UUID id) {
    return orderRepository
        .findById(id)
        .map(orderMapper::toResponse)
        .orElseThrow(() -> new OrderNotFoundException(id));
  }
}
```

---

## Maven Best Practices

All Maven projects must follow these conventions:

### POM Structure

```xml
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <!-- Inherit Spring Boot BOM for dependency version management -->
  <parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>3.2.x</version>
    <relativePath/>
  </parent>

  <groupId>com.example</groupId>
  <artifactId>my-service</artifactId>
  <version>1.0.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <properties>
    <!-- Always pin Java version explicitly -->
    <java.version>21</java.version>
    <maven.compiler.source>${java.version}</maven.compiler.source>
    <maven.compiler.target>${java.version}</maven.compiler.target>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
    <!-- Library versions not managed by Spring BOM go here -->
    <mapstruct.version>1.5.5.Final</mapstruct.version>
    <testcontainers.version>1.19.x</testcontainers.version>
  </properties>
</project>
```

### Dependency Rules

- **Use Spring Boot BOM** — never declare `<version>` for Spring Boot-managed dependencies
- **No version in child modules** — all versions declared in parent's `<dependencyManagement>`
- **Scope everything correctly:** `test` scope for test-only libs, `provided` for servlet-api in WAR
- **No `LATEST` or `RELEASE` versions** — always pin exact versions
- **Bill of Materials (BOM) imports** for multi-library stacks (Testcontainers, etc.)
- **Dependency hygiene:** Run `mvn dependency:analyze` to find unused declared and undeclared used dependencies

### Required Maven Plugins

```xml
<build>
  <plugins>
    <!-- Enforce minimum Maven and Java versions -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-enforcer-plugin</artifactId>
      <executions>
        <execution>
          <id>enforce</id>
          <goals><goal>enforce</goal></goals>
          <configuration>
            <rules>
              <requireMavenVersion><version>[3.9,)</version></requireMavenVersion>
              <requireJavaVersion><version>[17,)</version></requireJavaVersion>
              <dependencyConvergence/>
              <banDuplicatePomDependencyVersions/>
            </rules>
          </configuration>
        </execution>
      </executions>
    </plugin>

    <!-- Code coverage — enforce minimums -->
    <plugin>
      <groupId>org.jacoco</groupId>
      <artifactId>jacoco-maven-plugin</artifactId>
      <executions>
        <execution>
          <id>prepare-agent</id>
          <goals><goal>prepare-agent</goal></goals>
        </execution>
        <execution>
          <id>check</id>
          <goals><goal>check</goal></goals>
          <configuration>
            <rules>
              <rule>
                <element>BUNDLE</element>
                <limits>
                  <limit>
                    <counter>LINE</counter>
                    <value>COVEREDRATIO</value>
                    <minimum>0.80</minimum>
                  </limit>
                  <limit>
                    <counter>BRANCH</counter>
                    <value>COVEREDRATIO</value>
                    <minimum>0.70</minimum>
                  </limit>
                </limits>
              </rule>
            </rules>
          </configuration>
        </execution>
        <execution>
          <id>report</id>
          <goals><goal>report</goal></goals>
        </execution>
      </executions>
    </plugin>

    <!-- SpotBugs static analysis -->
    <plugin>
      <groupId>com.github.spotbugs</groupId>
      <artifactId>spotbugs-maven-plugin</artifactId>
      <version>4.8.x</version>
      <executions>
        <execution>
          <goals><goal>check</goal></goals>
        </execution>
      </executions>
    </plugin>

    <!-- Checkstyle — enforce Google Java Style -->
    <plugin>
      <groupId>org.apache.maven.plugins</groupId>
      <artifactId>maven-checkstyle-plugin</artifactId>
      <version>3.3.x</version>
      <configuration>
        <configLocation>google_checks.xml</configLocation>
        <failOnViolation>true</failOnViolation>
        <violationSeverity>warning</violationSeverity>
      </configuration>
      <executions>
        <execution>
          <id>checkstyle</id>
          <phase>verify</phase>
          <goals><goal>check</goal></goals>
        </execution>
      </executions>
    </plugin>

    <!-- OWASP Dependency-Check — flag known CVEs in dependencies -->
    <plugin>
      <groupId>org.owasp</groupId>
      <artifactId>dependency-check-maven</artifactId>
      <version>9.x</version>
      <configuration>
        <failBuildOnCVSS>7</failBuildOnCVSS>
      </configuration>
    </plugin>
  </plugins>
</build>
```

### Maven Profiles

```xml
<profiles>
  <!-- Integration tests run separately from unit tests -->
  <profile>
    <id>integration-tests</id>
    <build>
      <plugins>
        <plugin>
          <groupId>org.apache.maven.plugins</groupId>
          <artifactId>maven-failsafe-plugin</artifactId>
          <executions>
            <execution>
              <goals>
                <goal>integration-test</goal>
                <goal>verify</goal>
              </goals>
            </execution>
          </executions>
        </plugin>
      </plugins>
    </build>
  </profile>
</profiles>
```

---

## SonarLint / SonarQube Integration

All Java code is validated by SonarLint locally and SonarQube/SonarCloud in CI.

### SonarLint Local (VS Code)

The `sonarsource.sonarlint-vscode` extension is included in `.vscode/extensions.json`. SonarLint runs in real time. Rules to always resolve before committing:

- **Blocker / Critical rules:** Must be resolved — never suppress without written justification
- **Major rules:** Resolve or create a ticket before PR
- **`// NOSONAR`:** Only with a trailing comment explaining the rationale; never used to hide real defects

### SonarQube Maven Integration

```xml
<!-- Add to parent pom.xml properties -->
<sonar.java.source>21</sonar.java.source>
<sonar.coverage.jacoco.xmlReportPaths>
  ${project.build.directory}/site/jacoco/jacoco.xml
</sonar.coverage.jacoco.xmlReportPaths>
<sonar.exclusions>
  **/generated/**,
  **/*MapperImpl.java,
  **/config/**
</sonar.exclusions>
```

Run local analysis:
```bash
mvn sonar:sonar \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.token=${SONAR_TOKEN}
```

### SonarQube Quality Gate Thresholds (enforce in CI)

| Metric | Minimum |
|--------|---------|
| New code coverage | 80% |
| New code duplications | < 3% |
| New bugs | 0 |
| New vulnerabilities | 0 |
| New code smells (blocker/critical) | 0 |
| Security hotspots reviewed | 100% |

---

## Preferred Patterns

### REST Controller

```java
@RestController
@RequestMapping("/api/v1/orders")
@Tag(name = "Orders", description = "Order management endpoints")
@RequiredArgsConstructor  // if Lombok is already on classpath; else write constructor manually
public class OrderController {

    private final OrderService orderService;

    @GetMapping("/{id}")
    @Operation(summary = "Retrieve an order by ID")
    public ResponseEntity<OrderResponse> getOrder(@PathVariable UUID id) {
        return ResponseEntity.ok(orderService.findById(id));
    }

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    public OrderResponse createOrder(@Valid @RequestBody CreateOrderRequest request) {
        return orderService.create(request);
    }
}
```

### Exception Handling (RFC 7807 ProblemDetail)

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(OrderNotFoundException.class)
    public ProblemDetail handleOrderNotFound(OrderNotFoundException ex) {
        ProblemDetail problem = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, ex.getMessage());
        problem.setTitle("Order Not Found");
        return problem;
    }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail problem = ProblemDetail.forStatus(HttpStatus.UNPROCESSABLE_ENTITY);
        problem.setTitle("Validation Failed");
        problem.setProperty("violations", ex.getBindingResult().getFieldErrors()
            .stream().map(e -> e.getField() + ": " + e.getDefaultMessage()).toList());
        return problem;
    }
}
```

### Spring Data JPA Repository

```java
public interface OrderRepository extends JpaRepository<Order, UUID> {

    List<Order> findByCustomerIdAndStatus(UUID customerId, OrderStatus status);

    @Query("SELECT o FROM Order o JOIN FETCH o.lineItems WHERE o.id = :id")
    Optional<Order> findByIdWithLineItems(@Param("id") UUID id);

    Page<Order> findByCreatedAtAfter(Instant cutoff, Pageable pageable);
}
```

### Spring Security 6.x Configuration

```java
@Configuration
@EnableMethodSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health", "/actuator/info").permitAll()
                .requestMatchers("/api/**").authenticated()
                .anyRequest().denyAll()
            )
            .oauth2ResourceServer(oauth2 -> oauth2.jwt(Customizer.withDefaults()))
            .sessionManagement(s -> s.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
            .build();
    }
}
```

### Typed Configuration Properties

```java
@ConfigurationProperties(prefix = "app.payment")
public record PaymentProperties(
    String gatewayUrl,
    Duration timeout,
    int maxRetries
) {}
```

```yaml
# application.yml
app:
  payment:
    gateway-url: https://payment.example.com
    timeout: 30s
    max-retries: 3
```

### Java 21 Virtual Threads

```java
@Bean
public TomcatProtocolHandlerCustomizer<?> virtualThreadsCustomizer() {
    return handler -> handler.setExecutor(Executors.newVirtualThreadPerTaskExecutor());
}
```

---

## Anti-Patterns — Do NOT Generate

```java
// WRONG: field injection
@Autowired
private OrderService orderService;

// WRONG: javax namespace in Spring Boot 3
import javax.persistence.Entity;

// WRONG: WebSecurityConfigurerAdapter (removed in Spring Security 6)
public class SecurityConfig extends WebSecurityConfigurerAdapter { ... }

// WRONG: @Value on individual fields for grouped config
@Value("${app.payment.gateway-url}")
private String gatewayUrl;

// WRONG: returning null from service methods — use Optional or throw
public Order findOrder(UUID id) {
    return repository.findById(id).orElse(null);
}

// WRONG: catching and swallowing exceptions
try {
    processPayment(order);
} catch (Exception e) {
    // silent swallow
}

// WRONG: application.properties file
# app.payment.gateway-url=https://...
```

---

## Dependencies & Versions

| Library | Version | Import Style |
|---------|---------|-------------|
| Spring Boot | 3.2.x+ | `org.springframework.boot` |
| Spring Security | 6.x | `org.springframework.security` |
| Spring Data JPA | 3.x | `org.springframework.data.jpa` |
| springdoc-openapi | 2.x | `org.springdoc` |
| MapStruct | 1.5.x | `org.mapstruct` |
| Jakarta EE | 10 | `jakarta.*` |
| Java | 17 / 21 | — |

---

## Test Conventions

- Use `@WebMvcTest(MyController.class)` for controller layer tests — not full `@SpringBootTest`
- Use `@DataJpaTest` for repository tests — auto-configures H2 or Testcontainers
- Use `@MockBean` to inject mocked dependencies in slice tests
- Use `MockMvc` with `ObjectMapper` for controller request/response verification
- Name integration test classes with suffix `IT` — they run in a separate Maven phase
- Use `@Testcontainers` + `@Container` for tests requiring a real database or message broker
