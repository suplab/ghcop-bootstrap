---
applyTo: "**/estimates/**, **/*.estimate.md, **/planning/**, **/backlog/**"
description: "Estimation methodology: bottom-up sizing, 8h/day × 80% efficiency formula, confidence ranges, and assumption documentation. Produces human-day estimates for Java, Angular, and AWS work."
---

## Context

This instruction file applies to effort estimation documents. All estimates use bottom-up task decomposition with a productivity-adjusted human-day formula that accounts for real-world developer throughput.

---

## Core Formula

> **Human Days = Σ Raw Hours ÷ 6.4**
>
> - 8 hours per working day
> - 80% productive efficiency (accounts for: meetings, PR review cycles, context switching, environment setup, interruptions)
> - 6.4 = 8 × 0.80 productive hours per developer per day

---

## Estimate Tiers

| Tier | When to Use | Accuracy |
|------|------------|---------|
| **Rough Order of Magnitude (ROM)** | Idea/inception stage; < 1 day of analysis | ±50% |
| **Ballpark** | Feature outlined, no design | ±30% |
| **Bottom-Up** | Story broken into tasks, design known | ±15% |
| **Refined** | Technical spike done, full design agreed | ±10% |

State the tier in every estimate document.

---

## Task Sizing Reference

| Task | Simple | Moderate | Complex | Notes |
|------|--------|----------|---------|-------|
| REST endpoint (Spring Boot) | 2–4h | 4–8h | 8–16h | Includes service + repository |
| Angular component (standalone) | 2–4h | 4–8h | 8–12h | Includes HTML + SCSS |
| Database migration (Flyway) | 1–2h | 2–4h | 4–8h | |
| JUnit 5 unit test class | 1–2h | 2–4h | 4–6h | Per class, not per method |
| Integration test (Testcontainers) | 2–4h | 4–6h | 6–10h | |
| AWS CDK stack (new resource type) | 2–4h | 4–8h | 8–20h | Includes testing |
| Terraform module (new resource) | 2–4h | 4–8h | 8–16h | |
| Bug fix (known root cause) | 0.5–2h | 2–4h | 4–8h | |
| Bug fix (investigation needed) | Add 2–4h investigation time | | |
| Docker / K8s manifest | 1–2h | 2–4h | 4–8h | |
| CI/CD pipeline stage | 1–2h | 2–4h | 4–8h | |
| Technical documentation | 10–15% of feature raw hours | | |
| PR review iterations | 20% of implementation raw hours | | |

---

## Complexity Multipliers

| Condition | Multiplier | When to Apply |
|-----------|-----------|--------------|
| New team member (< 3 months on project) | ×1.3 | Per person on the task |
| Legacy codebase (> 5 years, limited tests) | ×1.4 | For any work touching legacy code |
| External API integration (unknown API) | Add 1–2 days discovery | Per new third-party API |
| Mainframe/COBOL integration | ×1.5 | Any mainframe touchpoints |
| First time using this technology | ×1.5 | Per new technology for the team |
| Well-understood feature (done before) | ×0.8 | Well-defined, team experienced |
| Greenfield (no legacy constraints) | ×0.9 | New service, clean slate |

---

## Confidence Ranges

| Confidence Level | Description | When to Use for Commitment |
|-----------------|-------------|--------------------------|
| **P50** | 50% probability of completing within | Never commit to P50 |
| **P80** | 80% probability | Sprint commitment |
| **P90** | 90% probability | Release planning, external promises |

P80 = P50 × 1.3 (rule of thumb for software estimates)
P90 = P50 × 1.5

---

## Estimate Document Template

```markdown
## Estimate: {Feature Title}

**Date:** {date}
**Prepared by:** {name/agent}
**Tier:** {ROM / Ballpark / Bottom-Up / Refined}
**Confidence:** {LOW / MEDIUM / HIGH}

### Task Breakdown

| # | Task | Stack | Complexity | Raw Hours Low | Raw Hours Likely | Raw Hours High |
|---|------|-------|-----------|--------------|-----------------|---------------|
| 1 | REST endpoint + service | Spring Boot | Moderate | 4h | 6h | 10h |
| 2 | Unit tests | JUnit 5 | Simple | 2h | 3h | 5h |
| 3 | Angular component | Angular | Moderate | 4h | 6h | 8h |
| 4 | Angular specs | Jasmine | Simple | 2h | 3h | 5h |
| 5 | DB migration | Flyway | Simple | 1h | 2h | 3h |
| 6 | PR review cycles | — | — | 2h | 3h | 4h |
| **Total** | | | | **15h** | **23h** | **35h** |

### Human Day Conversion (÷ 6.4 hrs/day)

| Scenario | Raw Hours | Human Days |
|----------|-----------|-----------|
| Optimistic | 15h | **2.3 days** |
| Likely (P50) | 23h | **3.6 days** |
| Pessimistic | 35h | **5.5 days** |

**P80 Commitment: ~4.7 days**
**P90 Release Buffer: ~5.4 days**

### Assumptions
1. {Assumption that affects the estimate if wrong}

### Risks
| Risk | Probability | Impact | Buffered? |
|------|------------|--------|----------|
| {Risk} | {H/M/L} | +{N} days | No — flag for PM |

### Not Included
- {What is explicitly out of scope}
```

---

## Anti-Patterns

- ❌ Estimating without listing assumptions
- ❌ Committing to P50 estimates externally
- ❌ Omitting PR review and rework time
- ❌ "We'll figure it out as we go" — always produce at least a ROM
- ❌ Single-point estimates without a confidence range
