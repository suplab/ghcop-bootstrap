# ADR-001 — Choose Spring Data JPA over Spring Data JDBC

**Date:** 2024-01-15
**Status:** Accepted
**Deciders:** Architecture team
**Context:** Order Management Platform — data access layer technology selection

---

## Context and Problem Statement

The Order Management Platform needs a data access strategy for its PostgreSQL database. Two primary candidates exist within the Spring ecosystem: Spring Data JPA (with Hibernate) and Spring Data JDBC. The choice affects: developer experience, query control, performance characteristics, and long-term maintainability.

---

## Decision Drivers

- Developer familiarity: most of the team has experience with JPA/Hibernate
- Query complexity: the domain has several complex aggregations and graph-shaped relationships
- Performance: the service handles up to 500 requests/second with P99 latency target of 200ms
- Testability: repository tests must be fast and isolated
- Migration: the schema will evolve frequently during the first 6 months

---

## Considered Options

1. **Spring Data JPA (Hibernate)** — JPA standard with Hibernate as the provider
2. **Spring Data JDBC** — lightweight, explicit, no lazy loading or session management
3. **jOOQ** — typesafe SQL builder, full SQL control, code generated from schema

---

## Decision Outcome

**Chosen: Spring Data JPA (Hibernate)**

Rationale:
- Team familiarity reduces ramp-up time and review cost
- Complex entity relationships (Order → LineItem → Product) map naturally to JPA's object graph
- `@EntityGraph` provides explicit fetch control, mitigating the N+1 risk
- `@DataJpaTest` slice tests keep repository tests fast without a full Spring context
- Spring Data JPA's `Specification` API covers the dynamic query requirements from the search feature

---

## Consequences

**Positive:**
- High developer productivity; minimal boilerplate for standard CRUD
- Rich query DSL via `JpaSpecificationExecutor`
- Well-understood by the whole team

**Negative:**
- Hibernate's lazy loading requires discipline (`@EntityGraph` on every fetch that traverses associations)
- Session-scoped first-level cache can cause subtle bugs if not understood; all service methods must be `@Transactional`
- Slightly higher startup time vs JDBC

**Mitigations:**
- Code review (`java-tech-lead` agent) enforces `@EntityGraph` usage and flags lazy-loaded associations iterated in loops
- `performance-engineer` agent scans for N+1 patterns in CI
- `@DataJpaTest` with `@AutoConfigureTestDatabase(replace = NONE)` + Testcontainers is the standard for all repository tests

---

## Links

- [Spring Data JPA Reference](https://docs.spring.io/spring-data/jpa/reference/)
- [Spring Data JDBC vs JPA](https://spring.io/blog/2018/09/24/spring-data-jdbc-references-and-aggregates)
- Related decision: ADR-002 — Flyway for database migrations
