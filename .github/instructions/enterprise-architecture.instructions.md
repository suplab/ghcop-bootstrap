---
applyTo: "**/architecture/**, **/adr/**, **/*.arch.md, **/docs/architecture/**, **/ea/**"
description: "Enterprise architecture standards: TOGAF alignment, ADR format, bounded context maps, capability maps, and architecture principles."
---

## Context

This instruction file applies to enterprise architecture documents, Architecture Decision Records (ADRs), and strategic design artifacts. Enterprise architecture operates at the level of capability, integration pattern, and technology lifecycle — not at the level of individual class designs.

---

## Architecture Decision Records (ADRs)

All significant technical decisions must be captured as ADRs. "Significant" means: hard to reverse, affects multiple teams, or introduces a new technology.

### ADR Mandatory Format

```markdown
# ADR-{NNN}: {Title — present tense, imperative}

## Status
[Proposed | Accepted | Superseded by ADR-NNN | Deprecated]

## Context
<What problem are we solving? What forces are in tension (time, cost, risk, quality)?
What constraints are non-negotiable? 1–3 paragraphs.>

## Decision
<State the decision in one clear sentence. Then justify it. No more than 2 paragraphs.>

## Consequences

### Positive
- <Concrete benefit>
- <Concrete benefit>

### Negative / Trade-offs
- <Concrete cost or risk>
- <What becomes harder>

## Alternatives Considered

| Option | Why Rejected |
|--------|-------------|
| {Option A} | {Specific reason} |
| {Option B} | {Specific reason} |

## References
- [Link to relevant RFC, doc, or prior ADR]
```

ADRs are stored in `/docs/adr/ADR-NNN-{slug}.md` and are version-controlled alongside the code they govern.

---

## Bounded Context Map Standards

Every programme context map must show:

1. **Bounded contexts** — named business domains with clear ownership
2. **Context relationships** — one of: Partnership, Shared Kernel, Customer-Supplier, Conformist, Anti-Corruption Layer, Open Host Service, Published Language
3. **Data ownership** — which context owns each domain entity
4. **Integration pattern** — synchronous API, event, or shared database (shared database requires ACL justification)

```markdown
## Context Relationships

| Upstream | Downstream | Relationship Type | Integration |
|----------|-----------|------------------|------------|
| Customer Identity | Order Management | Customer-Supplier | REST API |
| Order Management | Notification | Open Host Service | Domain Event |
| Legacy Mainframe | Order Management | Anti-Corruption Layer | Adapter pattern |
```

---

## Technology Lifecycle Classifications

| Status | Definition | Default Action |
|--------|-----------|---------------|
| **Invest** | Strategic technology, actively developed | Default choice for new work |
| **Tolerate** | Works but not strategic; maintenance-only | No new projects; plan migration |
| **Migrate** | Active migration in progress | Use only until migration complete |
| **Eliminate** | Scheduled for retirement; no new work | Set end-of-life date |

---

## Architecture Principles Format

```markdown
## Principle: {Name}

**Statement:** {One sentence, imperative.}

**Rationale:** {Why this principle exists. What problem does it prevent?}

**Implications:**
- {Concrete rule that follows from this principle}
- {Another concrete rule}

**Violations require:** ARB approval and documented exception in the relevant ADR.
```

---

## PACE Layer Model

Classify all systems by business volatility:

| Layer | Change Frequency | Examples | Architecture Style |
|-------|-----------------|---------|-------------------|
| **Systems of Record** | Years | Core banking, GL, HR | Monolith or stable microservices |
| **Systems of Differentiation** | Months | Customer portal, mobile app | Microservices, API-first |
| **Systems of Innovation** | Weeks | Experimentation, AI features | Serverless, event-driven |

---

## Integration Patterns by PACE Layer

| From \ To | Systems of Record | Systems of Differentiation | Systems of Innovation |
|-----------|------------------|--------------------------|----------------------|
| Systems of Record | Sync API or file | Async event or ACL | Read-only export |
| Systems of Differentiation | ACL + Sync API | Async event bus | Event subscription |
| Systems of Innovation | Read-only API | Event subscription | Direct API |

Anti-pattern: direct database access across PACE layers creates hidden coupling.
