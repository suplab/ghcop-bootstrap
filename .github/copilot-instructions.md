# GitHub Copilot — Repository Instructions

> This file is automatically loaded by GitHub Copilot for every interaction in this repository.
> All rules defined here apply globally unless overridden by a scoped `*.instructions.md` file.

---

## Program Context

This repository supports an enterprise software modernization program spanning three technology domains:

- **Legacy Java** — Spring 4/5 web applications (XML + annotation config, JdbcTemplate, servlet-based)
- **Modern Java** — Spring Boot 3.x microservices (Jakarta EE, Java 17/21, REST, JPA)
- **Angular** — Single-page applications built with Angular 15+ (standalone components, Signals API)
- **Mainframe** — IBM COBOL, Assembler, JCL, and CICS programs being analyzed and progressively modernized

Copilot is expected to understand all four domains and produce code that respects the conventions of each stack.

---

## Technology Stack Summary

| Domain | Language / Runtime | Key Frameworks & Libraries |
|--------|-------------------|---------------------------|
| Legacy Java | Java 8 / Java 11 | Spring 4.x/5.x, Spring MVC, JdbcTemplate, MapStruct, Maven |
| Modern Java | Java 17 / Java 21 | Spring Boot 3.x, Spring Data JPA, Spring Security 6.x, springdoc-openapi, MapStruct |
| Frontend | TypeScript (Angular 15+) | Angular, NgRx (optional), RxJS, Jasmine, Karma |
| Mainframe | IBM COBOL 6.x, Assembler, JCL | CICS, DB2 (z/OS), VSAM, QSAM |
| Testing | Java + TypeScript | JUnit 5, AssertJ, Mockito, Testcontainers, Awaitility, Pact |
| Build | Java | Maven (multi-module), npm |

---

## Universal Coding Standards

### Naming
- Classes: `PascalCase` — `CustomerService`, `OrderRepository`
- Methods and variables: `camelCase` — `findActiveOrders()`, `customerId`
- Constants: `UPPER_SNAKE_CASE` — `MAX_RETRY_COUNT`
- Packages: lowercase, dot-separated — `com.example.order.service`
- Test classes: suffix with `Test` (unit) or `IT` (integration) — `OrderServiceTest`, `OrderControllerIT`
- Angular components: kebab-case file names — `customer-list.component.ts`

### Design Principles
- Follow **SOLID** principles: every class has one reason to change
- Apply **DDD** where applicable: distinguish domain model from DTOs from persistence entities
- Use **hexagonal architecture** thinking: domain logic must not depend on infrastructure
- Prefer **composition over inheritance**
- Keep methods short (≤ 20 lines); extract private methods with descriptive names
- No magic numbers — use named constants

### Dependency Injection
- **Constructor injection only** — never `@Autowired` on fields
- Mark injected fields `final` (Java)
- Use `inject()` function in Angular (not constructor injection in modern components)

---

## Logging Standards

- Use **SLF4J** (`org.slf4j.Logger`) exclusively — never `System.out.println`, `System.err.println`, or `java.util.logging`
- Declare logger as: `private static final Logger log = LoggerFactory.getLogger(MyClass.class);`
- Log levels:
  - `DEBUG` — method entry/exit, parameter values (dev/test only)
  - `INFO` — significant business events (order created, user authenticated)
  - `WARN` — recoverable errors, unexpected but handled conditions
  - `ERROR` — exceptions that affect correctness; always include the exception object
- Never log sensitive data: passwords, tokens, PII, card numbers
- Use parameterized logging: `log.debug("Processing order {}", orderId)` — never string concatenation in log calls

---

## Security Non-Negotiables

- **No hardcoded secrets** — no passwords, API keys, tokens, or connection strings in code or properties files
- **No raw SQL with string concatenation** — always use parameterized queries (`PreparedStatement`, JPA `@Query`, named parameters)
- **Validate all user input** at system boundaries — use Bean Validation (`@Valid`, `@NotNull`, `@Size`)
- **No `@SuppressWarnings("unchecked")`** without an explanatory comment stating why it is safe
- All REST endpoints must be explicitly authorized — no anonymous access to business APIs
- Secrets belong in environment variables or a vault — reference `${ENV_VAR}` in config, never inline values

---

## Dependency Policy

- **Do not add new Maven or npm dependencies** that are not already declared in `pom.xml` / `package.json`
- If a dependency is genuinely needed, flag it with a comment: `// REQUIRES: add <groupId>:<artifactId>:<version> to pom.xml`
- Do not upgrade dependency versions without being asked — version changes require controlled review

---

## What Copilot Must NOT Generate

| Forbidden Pattern | Reason |
|------------------|--------|
| `System.out.println(...)` | Use SLF4J logger |
| `@SuppressWarnings` without comment | Masks real issues |
| Raw generic types (`List`, `Map` without type params) | Type unsafe |
| `catch (Exception e) { }` (empty catch) | Silently swallows errors |
| `catch (Exception e) { e.printStackTrace(); }` | Use logger instead |
| Hardcoded IP addresses, ports, or credentials | Use config/env vars |
| `new Date()` or `Calendar` | Use `java.time` API |
| `@SuppressWarnings("deprecation")` on new code | Fix the root cause |
| `Thread.sleep()` in tests | Use `Awaitility` |
| `SELECT *` in SQL | List columns explicitly |
| String concatenation in SQL queries | Use parameterized queries |
| Deprecated Spring APIs (`WebSecurityConfigurerAdapter`, `javax.*` in Boot 3.x) | Use current replacements |

---

## Commit Message Format

Follow **Conventional Commits** specification:

```
<type>(<scope>): <short description>

[optional body]

[optional footer]
```

Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`, `ci`

Examples:
```
feat(order): add customer credit limit validation
fix(auth): resolve JWT expiry not checked on refresh
test(invoice): add missing branch coverage for null line items
refactor(customer): extract address validation to domain service
```

---

## Pull Request Standards

- PR title follows Conventional Commits format
- PR description must include: what changed, why, how to test
- Every PR touching business logic must include or update tests
- PRs must not decrease overall test coverage
- All `[BLOCKER]` review comments must be resolved before merge

---

## Code Review Severity Labels

When reviewing code, use these labels consistently:

- `[BLOCKER]` — Must fix before merge: correctness bugs, security vulnerabilities, data loss risk
- `[MAJOR]` — Should fix before merge: missing error handling, architectural violations, no logging on exceptions
- `[MINOR]` — Fix in follow-up or current PR: naming issues, missing Javadoc, magic numbers
- `[NIT]` — Optional polish: formatting, unnecessary imports, whitespace

---

## IntelliJ / JetBrains Usage

This file is loaded automatically by GitHub Copilot in both VS Code and IntelliJ IDEA. The scoped `*.instructions.md` files in `.github/instructions/` and prompt files in `.github/prompts/` are accessible via Copilot Chat `#file:` reference in IntelliJ. See `intellij/` directory for IntelliJ-specific setup guidance.
