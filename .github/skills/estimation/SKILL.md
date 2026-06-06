---
name: estimation
description: 'Bottom-up effort estimation using 8h/day × 80% efficiency formula. Use when asked to estimate, size, or plan effort for Java Spring Boot, Angular, AWS CDK/Terraform, or CI/CD work. Produces human-day estimates with P50/P80/P90 confidence ranges.'
---

# Estimation Skill

Produces structured effort estimates using bottom-up task decomposition and a productivity-adjusted human-day formula.

## Formula

> **Human Days = Σ Raw Hours ÷ 6.4**
> (6.4 = 8 hours/day × 80% efficiency)

## Task Reference Table

| Task | Simple | Moderate | Complex |
|------|--------|----------|---------|
| REST endpoint (Spring Boot) | 2–4h | 4–8h | 8–16h |
| Angular component (standalone) | 2–4h | 4–8h | 8–12h |
| JUnit 5 unit test class | 1–2h | 2–4h | 4–6h |
| Testcontainers integration test | 2–4h | 4–6h | 6–10h |
| DB migration (Flyway) | 1–2h | 2–4h | 4–8h |
| CDK stack (new resource type) | 2–4h | 4–8h | 8–20h |
| Terraform module | 2–4h | 4–8h | 8–16h |
| Bug fix (known root cause) | 0.5–2h | 2–4h | 4–8h |
| PR review + rework buffer | +20% of implementation | | |
| Documentation | +10–15% of feature hours | | |

## Complexity Multipliers

| Condition | Multiplier |
|-----------|-----------|
| New team member (< 3 months) | ×1.3 |
| Legacy codebase | ×1.4 |
| First-time technology | ×1.5 |
| Well-understood pattern | ×0.8 |

## Confidence Levels

| Level | Formula | When to Commit |
|-------|---------|---------------|
| P50 | Likely total | Never (internal only) |
| P80 | P50 × 1.3 | Sprint commitment |
| P90 | P50 × 1.5 | Release planning |

## Output Format

Always produce:
1. Task breakdown table with raw hours (low/likely/high)
2. Human day conversion (÷ 6.4)
3. P50/P80/P90 confidence table
4. Numbered assumptions list
5. Risks table with probability and impact
6. "Not included" section for explicit scope exclusions
