# Modernization Patterns Standard

> Approved patterns for migrating legacy applications (COBOL, RPG, Spring 4/5) to modern platforms. Used by `modernization-expert` and `ibmi-modernization-expert` agents.

---

## Core Principle: Never Big Bang

Migrate incrementally. Each increment must leave the system in a working, deployable state. A migration that takes more than one sprint before going live is a risk. Decompose until each step is independently shippable.

---

## Pattern 1 — Strangler Fig

**Use when:** Replacing a monolith or legacy system that cannot be shut down during migration.

**Mechanism:** Route traffic through a façade (API Gateway, ALB, or reverse proxy). Gradually reroute paths from the legacy system to new services as each domain is migrated. The legacy system "strangles" until all routes have been moved and it can be decommissioned.

```
Phase 1: All traffic → Legacy
Phase 2: /orders/* → New Order Service | everything else → Legacy
Phase 3: /orders/* + /customers/* → New Services | /payments/* → Legacy
Phase N: All traffic → New Services | Legacy decommissioned
```

**Rules:**
- The façade must never contain business logic — routing only
- Keep the legacy system in read-write mode until the new system has been in production for one full billing cycle
- Feature-flag each route cut-over to enable instant rollback

---

## Pattern 2 — Anti-Corruption Layer (ACL)

**Use when:** The new service must consume data or call APIs from the legacy system during the transition period.

**Mechanism:** An ACL translates between the legacy model (COBOL copybooks, RPG data structures, legacy DB schemas) and the new domain model. The new service never imports legacy types directly.

```java
// CORRECT — ACL translates legacy DTO to domain object
@Component
public class LegacyOrderAdapter implements OrderPort {

    private final LegacyOrderClient legacyClient;
    private final LegacyOrderMapper mapper;  // MapStruct

    @Override
    public Order findById(OrderId orderId) {
        LegacyOrderDto legacy = legacyClient.getOrder(orderId.value());
        return mapper.toDomain(legacy);  // translate field names, date formats, status codes
    }
}

// WRONG — domain object directly uses legacy types
public class OrderService {
    public Order process(LegacyOrderRecord record) {  // legacy type leaks into domain
        ...
    }
}
```

**Rules:**
- The ACL lives in the `infrastructure` layer — never in `domain` or `application`
- Document every field mapping, especially where legacy semantics differ from new semantics (e.g., legacy status code `01` = new `OrderStatus.CONFIRMED`)
- The ACL must be the first thing deleted when the legacy system is decommissioned

---

## Pattern 3 — Parallel Run (Shadow Mode)

**Use when:** High-risk migration where correctness must be verified before cutting over.

**Mechanism:** Route each request to both the legacy and new system. Compare responses. Alert on divergence. Only the legacy response is returned to callers until confidence is established.

```
Request → Legacy (authoritative, response returned to caller)
        ↘ New Service (shadow, response compared async, divergences logged)
```

**Rules:**
- Never use the new service response to fulfil the request until divergence rate < 0.1% over 7 days
- Log all divergences with request payload, legacy response, and new response (redact PII)
- Define "divergence" precisely — acceptable differences (format, ordering) vs unacceptable (value, missing fields)
- Shadow traffic must not cause side effects in the new service (use a separate test database or rollback transactions)

---

## Pattern 4 — Database Decomposition (Strangler on Data)

**Use when:** Multiple services share a legacy database and need to be separated without breaking existing consumers.

**Phases:**

1. **Identify bounded contexts** — group tables by domain using event storming or domain analysis
2. **Introduce views** — create views in the legacy DB that the new service reads from (read-only path)
3. **Dual-write** — new service writes to its own DB; a synchronisation job writes back to the legacy DB for other consumers
4. **Cut read path** — once all consumers are migrated, remove the sync job and drop legacy table access

**Rules:**
- Never do cross-context joins in the new service — replicate read models instead
- The synchronisation job must be idempotent (safe to re-run)
- Legacy tables owned by another context are read-only to the new service — write only to your own tables

---

## COBOL to Java Migration

### Field Mapping Rules

| COBOL Type | Java Equivalent |
|---|---|
| `PIC 9(n)` (unsigned integer) | `int` / `long` |
| `PIC S9(n)` (signed integer) | `int` / `long` |
| `PIC 9(n)V9(m)` (decimal) | `BigDecimal` — never `double` |
| `PIC X(n)` (alphanumeric) | `String` (trimmed) |
| `PIC 9(8)` used as YYYYMMDD date | `LocalDate.parse(s, DateTimeFormatter.BASIC_ISO_DATE)` |
| `COMP-3` (packed decimal) | `BigDecimal` |

### Procedure Division to Service Method

- Each `PERFORM` section becomes a private method
- Each `CALL` to a sub-program becomes an injected dependency
- `WORKING-STORAGE` fields become method-local variables or constructor-injected state — never static
- Replicate COBOL error codes as a Java enum; do not invent new error semantics

---

## RPG / IBM i to Java Migration

### Program Call to Service

- `CALL` to RPG programs via IBM i toolkit becomes a call to a new Java service exposing the same interface
- Map IBM i data queues to Kafka topics or SQS queues
- DB2 for i tables accessed by RPG programs: wrap in JPA repositories; do not change table structure until all RPG programs are migrated

### Data Structure Mapping

| RPG Type | Java Equivalent |
|---|---|
| `10A` (character 10) | `String` (padded with spaces → trimmed) |
| `10P0` (packed numeric) | `long` |
| `10P2` (packed with 2 decimals) | `BigDecimal` (scale 2) |
| `D` (date) | `LocalDate` |
| `T` (time) | `LocalTime` |
| `Z` (timestamp) | `Instant` |

---

## Spring 4/5 to Spring Boot 3.x Migration

### javax → jakarta Namespace

Replace globally (not manually):

```bash
find src -name "*.java" -exec sed -i 's/import javax\.persistence\./import jakarta.persistence./g' {} +
find src -name "*.java" -exec sed -i 's/import javax\.validation\./import jakarta.validation./g' {} +
find src -name "*.java" -exec sed -i 's/import javax\.servlet\./import jakarta.servlet./g' {} +
```

Never leave `javax.*` imports alongside `jakarta.*` — they are incompatible in the same classpath.

### Security Migration (Spring Security 6.x)

- Replace deprecated `WebSecurityConfigurerAdapter` with `SecurityFilterChain` beans
- Replace `authorizeRequests()` with `authorizeHttpRequests()`
- Replace `antMatchers()` with `requestMatchers()`

### Testing Migration

- Replace `@RunWith(SpringRunner.class)` with `@ExtendWith(SpringExtension.class)` (JUnit 5)
- Replace `junit.framework.Assert` with `org.assertj.core.api.Assertions` (AssertJ)
- Replace `Mockito.verifyNoMoreInteractions()` calls with explicit behaviour verification

---

## Anti-Patterns

| Anti-Pattern | Correct Alternative |
|---|---|
| Big bang rewrite | Strangler Fig with incremental cut-over |
| Legacy types leaking into new domain | Anti-Corruption Layer translating at boundary |
| Cross-context DB joins in new service | Replicated read model via event sync |
| Running old and new in production simultaneously without comparison | Parallel run (shadow mode) with divergence tracking |
| Using `double` for monetary values in COBOL migration | `BigDecimal` always |
| Leaving `javax.*` alongside `jakarta.*` | Full global find-and-replace before compile |
