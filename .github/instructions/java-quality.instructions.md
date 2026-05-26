---
applyTo: "**/*.java, **/pom.xml, **/.checkstyle*, **/checkstyle*.xml"
---

## Context

This instruction file enforces Java code quality standards across all Java modules: Google Java Style Guide formatting, static analysis tooling (SpotBugs, Checkstyle, SonarLint), test coverage mandates (JaCoCo), and security vulnerability scanning (OWASP Dependency-Check). These rules apply to both legacy Spring MVC modules and modern Spring Boot modules. They complement the stack-specific instructions in `spring-boot.instructions.md` and `java-legacy.instructions.md`.

---

## Google Java Style Guide — Enforced Rules

All generated Java code must comply with the [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html).

### Formatting Rules

| Rule | Value |
|------|-------|
| Indentation | 2 spaces (no tabs) |
| Continuation indent | +4 spaces |
| Column limit | 100 characters |
| Brace style | Egyptian (K&R): opening brace on same line |
| Braces | Always present — even for single-statement blocks |
| Blank lines between members | 1 blank line |
| Blank line after class opening brace | None |
| Wildcard imports | Forbidden |
| Static imports | Grouped first, then all others |
| `var` | Allowed for local variables where the type is clear from the right-hand side |

### Naming

| Element | Convention | Example |
|---------|-----------|---------|
| Class | `UpperCamelCase` | `CustomerService` |
| Method | `lowerCamelCase` | `findActiveCustomers` |
| Variable | `lowerCamelCase` | `orderId` |
| Constant | `UPPER_SNAKE_CASE` | `MAX_RETRY_COUNT` |
| Type parameter | Single uppercase or `UpperCamelCase` + `T` | `T`, `CustomerT` |
| Package | All lowercase, dot-separated | `com.example.order` |
| Acronyms | Treated as words | `HttpUrl`, not `HTTPUrl`; `JsonParser`, not `JSONParser` |

### Javadoc Requirements

```java
/**
 * Processes a customer payment request and returns the transaction result.
 *
 * <p>If the customer's balance is insufficient, the transaction is declined
 * and a {@link PaymentDeclinedException} is thrown.
 *
 * @param customerId the UUID of the customer making the payment
 * @param amount the payment amount; must be positive
 * @return the completed transaction result
 * @throws PaymentDeclinedException if the payment cannot be processed
 * @throws IllegalArgumentException if {@code amount} is null or non-positive
 */
public TransactionResult processPayment(UUID customerId, BigDecimal amount) { ... }
```

- `@param` for every parameter
- `@return` for every non-void method
- `@throws` for every checked exception and significant runtime exception
- Use `{@code ...}` for inline code references
- Use `{@link ...}` for type references
- Do not write Javadoc that merely restates the method signature

---

## Code Quality — Static Analysis Tools

### Checkstyle (Google Checks)

Checkstyle is configured with `google_checks.xml` (provided by the `checkstyle` library). Run locally:

```bash
mvn checkstyle:check
```

Common violations to eliminate before committing:
- Line length > 100 characters
- Missing Javadoc on public methods
- Wildcard imports
- Tabs instead of spaces
- Magic numbers (use named constants)
- Missing `@Override` annotation

### SpotBugs

SpotBugs performs bytecode-level static analysis. Run locally:

```bash
mvn spotbugs:check
```

SpotBugs bug categories to treat as build failures:

| Category | Examples |
|----------|---------|
| `CORRECTNESS` | Null dereference, infinite loop, integer overflow |
| `SECURITY` | SQL injection, path traversal, hardcoded password |
| `BAD_PRACTICE` | Unclosed streams, ignored return values |
| `PERFORMANCE` | Unnecessary object creation in loops |

SpotBugs suppression — only with justification:

```java
@SuppressFBWarnings(
    value = "NP_NULL_ON_SOME_PATH_FROM_RETURN_VALUE",
    justification = "findById is called after an existence check; null is impossible here"
)
```

### PMD (Optional)

If PMD is configured, enforce:
- `UnusedImports`, `UnusedLocalVariable`
- `EmptyCatchBlock` — must always have a logged message or re-throw
- `SystemPrintln` — use SLF4J
- `AvoidDeeplyNestedIfStmts` — extract methods instead

---

## JaCoCo — Test Coverage

Minimum thresholds enforced at build time:

| Metric | Threshold |
|--------|----------|
| Line coverage (business logic) | 80% |
| Branch coverage (business logic) | 70% |
| Line coverage (overall) | 70% |

Exclude from coverage measurement:
- `**/*MapperImpl.java` (MapStruct generated)
- `**/generated/**`
- `**/*Application.java`
- `**/*Config.java` (pure Spring config classes)
- `**/dto/**`, `**/model/**` (pure data holders with no logic)

```xml
<!-- In jacoco-maven-plugin configuration -->
<excludes>
  <exclude>**/*MapperImpl.class</exclude>
  <exclude>**/generated/**</exclude>
  <exclude>**/*Application.class</exclude>
</excludes>
```

---

## OWASP Dependency-Check

Scans all declared dependencies for known CVEs. Integrated as a Maven plugin; run locally:

```bash
mvn dependency-check:check
```

- **CVSS score ≥ 7.0** (High/Critical) → build fails
- **CVSS score 4.0–6.9** (Medium) → generate report; review before merge
- Suppressions require a `suppression.xml` entry with `<notes>` explaining the justification and a review date

---

## SonarLint Local Workflow

1. Install `sonarsource.sonarlint-vscode` extension (in `.vscode/extensions.json`)
2. On file save, SonarLint highlights issues inline in the editor
3. Connect to SonarQube/SonarCloud for team-wide rule synchronization:
   - `Ctrl+Shift+P` → `SonarLint: Connect to SonarQube`
   - Provide server URL and token
4. Run a full file analysis: right-click → `SonarLint: Analyze All Open Files`

### Rules to Never Suppress

| Rule Key | Description |
|----------|-------------|
| `java:S2068` | Hardcoded credentials |
| `java:S106` | `System.out.println` usage |
| `java:S1481` | Unused local variable |
| `java:S2095` | Resources must be closed |
| `java:S3457` | Format string not properly formatted |
| `java:S2259` | Null dereference |
| `java:S1874` | Deprecated API usage |

---

## Pre-Commit Quality Checklist

Before every commit, verify:

- [ ] `mvn checkstyle:check` passes (zero violations)
- [ ] `mvn spotbugs:check` passes (zero high/critical bugs)
- [ ] `mvn test` passes (all unit tests green)
- [ ] SonarLint shows no Blocker or Critical issues in changed files
- [ ] No `System.out.println` in changed files
- [ ] All public methods have Javadoc
- [ ] No hardcoded secrets or connection strings

---

## CI Quality Gates

In CI/CD pipelines, run in this order:

```bash
mvn verify                         # compile + unit tests + checkstyle + spotbugs + jacoco
mvn sonar:sonar                    # SonarQube analysis
mvn dependency-check:check         # OWASP CVE scan (may be separate pipeline step)
mvn failsafe:integration-test      # integration tests (separate profile)
```

The pipeline must not pass unless all quality gates are green.
