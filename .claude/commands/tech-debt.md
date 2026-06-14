# /tech-debt — Register and Manage Technical Debt

Add, view, or prioritise technical debt items in `.claude/memory/tech-debt.md`.

## Usage

```
/tech-debt add "Remove deprecated LegacyOrderAdapter — only needed until Q1 migration completes"
/tech-debt add "Replace manual SQL in ReportingRepository with Spring Data projections — risk: SQL injection if params added without care"
/tech-debt list
/tech-debt prioritise
```

## Subcommands

### `add "description"`
Registers a new debt item. You will be prompted for:
- **Category:** Code / Architecture / Test / Security / Dependency / Operational
- **Priority:** Critical / High / Medium / Low
- **Effort:** S (< 1 day) / M (1–3 days) / L (3–10 days) / XL (> 10 days)
- **Risk if unaddressed:** brief statement of what could go wrong
- **Target sprint/quarter:** when to address (or "backlog" if no target)

The item is appended to `.claude/memory/tech-debt.md` with a unique ID (`TD-<NNN>`).

### `list`
Displays the current tech debt register, grouped by priority, with effort and category.

### `prioritise`
Activates the `java-tech-lead` or `architect` agent to review the full debt register and suggest a prioritisation based on:
- Risk if unaddressed
- Effort required
- Sprint capacity
- Dependencies between items

## Output Format (after `add`)

```markdown
## TD-042 — Remove deprecated LegacyOrderAdapter
- **Category:** Code
- **Priority:** Medium
- **Effort:** S (< 1 day)
- **Registered:** 2024-03-15
- **Target:** Q2 2024 — after full migration cut-over
- **Risk:** Confuses new developers; adapter will accumulate dead code as migration progresses
- **Owner:** order-service team
```

## Tech Debt Categories

| Category | Examples |
|---|---|
| **Code** | Dead code, deprecated API usage, duplicated logic, missing abstractions |
| **Architecture** | Shared database across contexts, wrong layer dependencies, missing ACL |
| **Test** | Missing test coverage, flaky tests, tests without assertions |
| **Security** | Outdated library with CVE, overly broad IAM permissions, disabled security feature |
| **Dependency** | Outdated framework version, EOL dependency, conflicting transitive deps |
| **Operational** | Missing runbook, unmonitored alert, manual deployment step, missing backup |

## Tips

- Critical debt items should be scheduled within the current sprint
- High items should appear in the next two sprints' planning
- Review the debt register at the start of each sprint planning session using `/tech-debt list`
- Items that have been in the register for more than 90 days without progress should be escalated to the tech lead
