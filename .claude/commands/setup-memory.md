# /setup-memory — Interactive Project Memory Initialisation

Run an interactive interview to populate `.claude/memory/project-context.md` and related memory files with project-specific context. Run this once when adopting the EEIK bootstrap into a new project.

## Usage

```
/setup-memory
```

No arguments needed. The command guides you through the required fields interactively.

## What This Command Does

1. Reads the current state of all `.claude/memory/` files to determine what is already filled in
2. Asks a series of focused questions (grouped by topic) to collect project-specific information
3. Writes the answers into the correct memory files
4. Confirms what was updated and what remains as placeholder

## Interview Topics

The command covers these sections in order:

### 1. Project Identity (5 questions)
- Project / service name
- Team name and Slack channel
- Primary domain (e.g. "Order Fulfilment", "Payment Processing")
- Primary language and framework (Java 21 / Spring Boot 3.x, Python 3.12 / FastAPI, etc.)
- Repository URL

### 2. Service Inventory (open-ended)
- List of services: name, repo, purpose, owning team
- External dependencies: third-party APIs, legacy systems, partner services

### 3. Environments
- Dev, Staging, Production URLs
- AWS account IDs and regions for each environment
- Environment-specific notes (e.g. "staging uses anonymised production data")

### 4. Authentication
- User authentication mechanism (Cognito, Keycloak, Azure AD, etc.)
- Service-to-service authentication (IAM roles, client credentials, mTLS)
- External API auth (API keys, OAuth2)

### 5. Database
- Engine and version (PostgreSQL 15, Aurora, DB2, etc.)
- Schema name(s)
- Migration tool (Flyway, Liquibase)
- ORM (Spring Data JPA, JDBC, SQLAlchemy)

### 6. Messaging
- Broker type (Kafka, SQS/SNS, EventBridge, RabbitMQ)
- Key topics / queues with their purpose

### 7. On-Call & Escalation
- Engineering on-call rotation (PagerDuty, OpsGenie)
- Escalation contacts for P1 incidents

### 8. Domain Glossary (optional)
- 5–10 key domain terms and their definitions that a new developer would need to understand

## Output

After the interview, the command confirms:
```
✅ project-context.md updated — 12 fields populated
✅ domain-glossary.md updated — 6 terms added
⚠️  Messaging section skipped — answer /setup-memory messaging to fill in later
```

## Tips

- You do not need to complete all sections in one session — skip any section and come back with `/setup-memory <section>`
- After running, ask Claude Code a question about your project — the memory context will be active immediately
- Use `/memory-update "what changed"` to update individual fields as the project evolves
- The filled-in memory files should be committed to the repository so the whole team benefits
