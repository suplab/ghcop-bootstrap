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

## Available Agents

Select an agent from the Copilot Chat **@** dropdown to activate a specialist persona. Full catalogue: see `AGENTS.md` at the repository root.

### Java Development
| Agent | Activate With | Use For |
|-------|--------------|---------|
| Java Developer | `@java-dev` | Ticket-scoped Spring Boot implementation |
| Java Tech Lead | `@java-tech-lead` | PR gating, standards enforcement, tech debt |
| Java Test Engineer | `@java-tester` | JUnit 5 / Testcontainers test suites |
| JaCoCo Coverage Analyst | `@jacoco-coverage-tester` | Coverage gap analysis and targeted test generation |
| Senior Java/Angular Developer | `@developer` | Full-stack Java + Angular implementation |

### Angular Development
| Agent | Activate With | Use For |
|-------|--------------|---------|
| Angular Developer | `@angular-dev` | Standalone components, signals, lazy routes |
| Angular Test Engineer | `@angular-tester` | Jasmine/TestBed specs |
| Angular Coverage Analyst | `@angular-coverage-checker` | Istanbul/Karma coverage gap analysis |

### Architecture & Design
| Agent | Activate With | Use For |
|-------|--------------|---------|
| Solution Architect | `@architect` | ADRs, bounded contexts, API contracts |
| Enterprise Architect | `@enterprise-architect` | Capability maps, technology lifecycle, TOGAF |
| AWS Solution Architect | `@aws-architect` | Well-Architected reviews, CDK stacks, cost estimates |

### Code Quality & Security
| Agent | Activate With | Use For |
|-------|--------------|---------|
| Code Reviewer | `@reviewer` | [BLOCKER]/[MAJOR]/[MINOR]/[NIT] PR reviews |
| Security Auditor | `@security-auditor` | OWASP Top 10 audit with remediation code |
| Performance Specialist | `@performance-reviewer` | N+1 queries, resource leaks, rendering |
| Coverage Guardian | `@coverage-enforcer` | Coverage gap analysis and targeted tests |
| Test Quality Inspector | `@test-quality-enforcer` | Anti-pattern detection and test regeneration |

### Infrastructure & Deployment
| Agent | Activate With | Use For |
|-------|--------------|---------|
| CDK / Terraform Helper | `@cdk-terraform-helper` | IaC stacks (CDK TypeScript or Terraform HCL) |
| AWS Deploy Helper | `@aws-deploy-helper` | Deploy commands, pre-deploy checklist, rollback |
| Local Deploy Helper | `@local-deploy-helper` | Docker Compose setup, smoke tests |
| Containerisation Helper | `@containerisation-helper` | Dockerfiles, K8s manifests, Helm |
| CI Engineer | `@ci-engineer` | GitHub Actions / Jenkins pipelines |

### Data, ML & AI (AWS)
| Agent | Activate With | Use For |
|-------|--------------|---------|
| AWS Data Scientist | `@data-scientist-aws` | SageMaker notebooks, Glue ETL, Athena |
| AWS ML Engineer | `@ml-engineer-aws` | SageMaker pipelines, model registry, MLOps |
| AWS AI Engineer | `@ai-engineer-aws` | Bedrock LLM, RAG pipelines, guardrails |

### Delivery & Operations
| Agent | Activate With | Use For |
|-------|--------------|---------|
| Estimator | `@estimator` | Bottom-up estimates (8h/day × 80% = 6.4h/day) |
| Project Tracker | `@project-tracker` | Sprint burndown, story status, velocity |
| Ops Engineer | `@ops-engineer` | CloudWatch dashboards, alarms, runbooks |
| Incident Handler | `@incident-handler` | P1/P2 war room coordination |
| RCA Agent | `@rca-agent` | 5-Whys root cause analysis |

### Modernisation
| Agent | Activate With | Use For |
|-------|--------------|---------|
| Mainframe Modernization Specialist | `@modernization-expert` | COBOL → Java with semantic risk matrix |
| Business Analyst | `@analyst` | OpenAPI specs, Gherkin acceptance criteria |
| QA Automation Engineer | `@tester` | Full test pyramid for any stack |

---

## Agent Skills (Auto-Loaded)

The following skills in `.github/skills/` are loaded automatically by Copilot when relevant:

- **estimation** — Bottom-up effort estimation with P50/P80/P90 confidence ranges
- **jacoco-analysis** — JaCoCo report parsing and gap analysis
- **aws-cdk-deploy** — CDK deploy commands and rollback procedures
- **incident-response** — ITIL P1/P2 templates and escalation matrix
- **code-quality-scan** — SonarQube, SpotBugs, Checkstyle, OWASP report triage

---

## Hooks

Lifecycle hooks in `.github/hooks/` log session activity to `.copilot-*.log` files:

- **session-hooks.json** — logs session start/end and prompt submissions
- **tool-use-hooks.json** — logs tool invocations and outcomes

---

## IntelliJ / JetBrains Usage

This file is loaded automatically by GitHub Copilot in both VS Code and IntelliJ IDEA. Agents (`.github/agents/`), instruction files (`.github/instructions/`), and prompt files (`.github/prompts/`) are accessible via Copilot Chat `#file:` reference in IntelliJ. See `intellij/` directory for IntelliJ-specific setup guidance.
