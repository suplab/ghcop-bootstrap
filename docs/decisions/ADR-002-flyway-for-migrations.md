# ADR-002 — Use Flyway for Database Migrations

**Date:** 2024-01-15
**Status:** Accepted
**Deciders:** Architecture team, DBA
**Context:** Order Management Platform — database schema migration strategy

---

## Context and Problem Statement

The platform's PostgreSQL schema will evolve continuously. We need a migration tool that: integrates with Spring Boot, supports CI/CD pipelines, provides rollback capability, and gives the DBA team visibility into every change applied to production.

---

## Decision Drivers

- Spring Boot native integration (auto-runs on startup)
- SQL-first: the DBA team reviews every migration as plain SQL
- Versioned and audited: every migration tracked in `flyway_schema_history`
- Supports CI testing: migrations run against Testcontainers in CI
- Team familiarity: majority of team has used Flyway before

---

## Considered Options

1. **Flyway** — SQL-first, versioned, Spring Boot auto-integration
2. **Liquibase** — XML/YAML/JSON changesets, rollback built-in, more complex
3. **Manual DDL scripts** — total control, no tooling dependency, no audit trail

---

## Decision Outcome

**Chosen: Flyway**

Rationale:
- SQL-first aligns with DBA workflow — they review `.sql` files, not XML changesets
- Spring Boot `spring.flyway.enabled=true` requires zero additional config
- Simpler mental model: versioned files, one direction, never edit a committed migration
- The Flyway Enterprise rollback feature is available if needed, but the team prefers forward-only migrations with explicit rollback scripts

---

## Migration Conventions

Enforced by the `dba-advisor` agent and code review:

- **Naming:** `V<version>__<snake_case_description>.sql` — e.g., `V3__add_email_verified_to_customers.sql`
- **Version numbering:** integers, sequential — `V1`, `V2`, `V3` (not timestamps)
- **Rollback:** every migration includes a corresponding `docs/migrations/rollbacks/V<N>__rollback.sql`
- **Forward-only:** never modify a committed migration; create a new one to fix a mistake
- **Online DDL:** see `dba-advisor` agent for large table migration patterns

---

## Consequences

**Positive:**
- Zero migration failures in production due to version conflict detection
- Full audit trail in `flyway_schema_history`
- Migrations tested automatically in CI via Testcontainers
- DBA can review SQL before deployment

**Negative:**
- No built-in rollback (Flyway community edition); rollback scripts maintained manually
- Versioning conflicts require coordination in multi-developer branches (use version suffixes: `V3.1`, `V3.2`)

**Mitigations:**
- Branch-specific migrations use sub-versions: `V3.1__feature-branch-change.sql`
- Rollback scripts stored in `docs/migrations/rollbacks/` and tested in staging
- CI gate runs `flyway validate` before any deployment

---

## Links

- [Flyway Documentation](https://documentation.red-gate.com/fd)
- Related decision: ADR-001 — Spring Data JPA over JDBC
- `/migrate-db` command: generates migration files following these conventions
