# Enterprise Engineering Intelligence Kit (EEIK)
## Volume 1 - Master Architecture Specification

# 1. Vision

EEIK is an opinionated enterprise engineering operating system designed for:

- AWS-first delivery
- Java 21/25 and Spring Boot
- Python AI engineering
- React and Angular frontends
- IBM i and Mainframe modernization
- Agentic AI engineering
- DevSecOps
- Architecture governance
- Enterprise delivery governance

---

# 2. Architectural Layers

## L0 Foundation

Purpose:
Enterprise-wide standards and governance.

Directories:

```text
.claude/standards/
.claude/governance/
.claude/checklists/
.github/standards/
```

Artifacts:

- Java standards
- Python standards
- React standards
- Angular standards
- AWS standards
- Security standards
- Architecture standards
- Documentation standards

---

## L1 Engineering

Domains:

- Java
- Python
- Frontend
- AWS
- Containers

Agents:

- java21-engineer
- java25-engineer
- springboot-engineer
- python-enterprise-engineer
- react-engineer
- angular-engineer
- docker-engineer
- kubernetes-engineer

---

## L2 Architecture

Agents:

- enterprise-architect
- solution-architect
- cloud-architect
- security-architect
- data-architect
- integration-architect
- platform-architect

Outputs:

- HLD
- LLD
- ADR
- RFC
- C4 diagrams
- Threat models

---

## L3 AI Engineering

Frameworks:

- LangGraph
- CrewAI
- AutoGen
- MCP
- A2A
- Spring AI
- Bedrock

Agents:

- ai-architect
- langgraph-architect
- mcp-architect
- rag-architect
- evaluation-specialist

---

## L4 Modernization

Domains:

- IBM i
- RPG
- COBOL
- Mainframe
- JCL
- CICS
- IMS
- DB2

Agents:

- ibmi-modernization-architect
- rpg-specialist
- cobol-specialist
- strangler-migration-specialist

---

## L5 Delivery

Agents:

- scrum-master
- product-owner
- business-analyst
- estimation-specialist
- release-manager

---

## L6 Operations

Agents:

- incident-commander
- rca-investigator
- sre-specialist
- observability-specialist

---

# 3. Complete Agent Families

## Architecture

- Enterprise Architect
- Solution Architect
- Cloud Architect
- Security Architect
- Data Architect
- Event Architect
- Integration Architect
- API Architect
- Platform Architect
- FinOps Architect
- AI Architect

## Development

- Java 17 Engineer
- Java 21 Engineer
- Java 25 Engineer
- Spring Boot Engineer
- Spring AI Engineer
- Python Engineer
- FastAPI Engineer
- React Engineer
- Angular Engineer
- Micro Frontend Engineer

## Code Explainers

- Java Explainer
- Spring Explainer
- Python Explainer
- React Explainer
- Angular Explainer
- RPG Explainer
- COBOL Explainer
- JCL Explainer
- Terraform Explainer
- CDK Explainer

## AI

- LangGraph Architect
- LangGraph Engineer
- CrewAI Architect
- CrewAI Engineer
- AutoGen Architect
- MCP Architect
- MCP Engineer
- Prompt Engineer
- Evaluation Specialist
- Hallucination Reviewer

## Data

- Data Scientist
- ML Engineer
- MLOps Engineer
- Forecasting Specialist
- Analytics Engineer
- BI Architect

## DevSecOps

- CI/CD Specialist
- Security Reviewer
- SAST Specialist
- DAST Specialist
- IAM Specialist
- SRE Specialist
- Platform Engineer

## Governance

- Architecture Review Board
- AI Governance Reviewer
- Compliance Specialist
- Audit Specialist

## Insurance

- Claims Architect
- Fraud Specialist
- Underwriting Specialist
- Policy Specialist

---

# 4. Memory Architecture

EEIK uses two distinct memory mechanisms. Understanding the difference is important
for implementation.

## Agent Persistent Memory (Claude Code native)

Set `memory: project` in an agent's YAML frontmatter. Claude Code auto-creates and
manages `.claude/agent-memory/<agent-name>/MEMORY.md`. The first 200 lines are
injected into the agent's context at every session start, enabling cross-session learning.

```text
.claude/agent-memory/
├── enterprise-architect/
│   └── MEMORY.md          ← auto-managed, injected at startup
├── incident-commander/
│   └── MEMORY.md
└── <agent-name>/
    └── MEMORY.md
```

## Project Knowledge Directories (shared reference)

Plain directories in `.claude/knowledge/` for shared reference knowledge that multiple
agents can read or write during tasks. Not agent-specific; represents project-wide
institutional knowledge.

```text
.claude/knowledge/
├── glossary/
├── business-rules/
├── architecture-decisions/
├── api-catalog/
├── event-catalog/
├── domain-models/
├── data-dictionary/
└── reference-architectures/
```

Purpose: Persistent organizational intelligence accessible to all agents.

---

# 5. Command Catalog

Commands live in `.claude/commands/<name>.md`. Each file defines: purpose, expected inputs,
output format, delegating agents, and memory read/write behaviour.

Architecture:

- /design
- /architecture-review
- /create-hld
- /create-lld
- /create-adr
- /create-rfc

Engineering:

- /implement
- /refactor
- /generate-tests
- /generate-diagram

AI:

- /create-agent
- /review-agent
- /langgraph-review
- /mcp-review

Operations:

- /rca
- /incident-report
- /postmortem

Delivery:

- /estimate
- /create-epic
- /create-story
- /release-plan

---

# 6. Hook Architecture

Hooks are configured in `.claude/settings.json` and execute shell scripts in `.claude/hooks/`.
The EEIK conceptual hook categories map to Claude Code hook events as follows:

## Claude Code Hook Events Used by EEIK

| EEIK Concept | Claude Code Hook Event | Purpose |
|---|---|---|
| Pre Prompt | `SessionStart` | Load project context, inject standards at session start |
| Per-Prompt Injection | `UserPromptSubmit` | Inject relevant standards per prompt |
| Post-Edit Validation | `PostToolUse` (matcher: `Edit\|Write`) | Run linter/formatter after file edits |
| Bash Validation | `PreToolUse` (matcher: `Bash`) | Block destructive commands |
| Pre Commit | `.git/hooks/pre-commit` | Lint, test, coverage gate (git hook, not Claude hook) |
| Pull Request | GitHub Actions workflows | Architecture/security/AI/coverage reviews |
| Release | GitHub Actions workflows | Release notes, changelog, deployment summary |

## settings.json Hook Configuration

```json
{
  "hooks": {
    "SessionStart": [
      { "hooks": [{ "type": "command", "command": ".claude/hooks/session-start.sh" }] }
    ],
    "UserPromptSubmit": [
      { "hooks": [{ "type": "command", "command": ".claude/hooks/prompt-submit.sh" }] }
    ],
    "PostToolUse": [
      { "matcher": "Edit|Write",
        "hooks": [{ "type": "command", "command": ".claude/hooks/post-edit-lint.sh" }] }
    ],
    "PreToolUse": [
      { "matcher": "Bash",
        "hooks": [{ "type": "command", "command": ".claude/hooks/validate-bash.sh" }] }
    ]
  }
}
```

## Complete Claude Code Hook Event Reference

Session lifecycle: `SessionStart`, `SessionEnd`, `Setup`
Per-turn: `UserPromptSubmit`, `UserPromptExpansion`, `Stop`, `StopFailure`
Agentic loop: `PreToolUse`, `PostToolUse`, `PostToolUseFailure`, `PermissionRequest`,
              `PermissionDenied`, `PostToolBatch`
Subagent: `SubagentStart`, `SubagentStop`
Other: `PreCompact`, `PostCompact`, `FileChanged`, `CwdChanged`, `ConfigChange`,
       `InstructionsLoaded`, `Notification`, `Elicitation`, `ElicitationResult`,
       `WorktreeCreate`, `WorktreeRemove`, `MessageDisplay`, `TeammateIdle`,
       `TaskCreated`, `TaskCompleted`

---

# 7. Model Routing

Configured via `model` field in each agent's YAML frontmatter.

Claude Sonnet 4.6

- Architecture
- Modernization
- AI Design
- Governance

Claude Sonnet 4.5

- Coding
- Refactoring
- Reviews
- Documentation

GPT-5.5

- Estimation
- RCA
- Program Management
- Business Analysis

---

# 8. Knowledge Architecture

```text
.claude/knowledge/
├── glossary/                   ← domain terminology and definitions
├── business-rules/             ← business logic and constraints
├── architecture-decisions/     ← ADR archive
├── api-catalog/                ← API inventory and contracts
├── event-catalog/              ← domain event registry
├── domain-models/              ← bounded context models
├── data-dictionary/            ← canonical data definitions
└── reference-architectures/   ← reusable architecture patterns
```

Each subdirectory contains a `README.md` defining schema, format, and usage conventions
for that knowledge type. Agents read these at task time and write to them as knowledge
accumulates.

---

# 9. Future Volumes

Volume 2:
Complete file-level repository structure.

Volume 3:
Agent catalog with all agent specifications.

Volume 4:
Command catalog and hook definitions.

Volume 5:
Implementation examples and templates.

Volume 6:
AWS, Java, Python, React, Angular standards.

Volume 7:
IBM i and Mainframe modernization framework.

Volume 8:
Insurance domain intelligence pack.
