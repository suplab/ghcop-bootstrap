# Enterprise Engineering Intelligence Kit (EEIK)

## Overview

Enterprise Engineering Intelligence Kit (EEIK) is an opinionated AWS-first, Java-first, AI-native enterprise bootstrap repository.

This document is the starter blueprint and includes:

- GitHub Copilot configuration structure
- Claude Code configuration structure
- Enterprise agent catalog
- Architecture governance
- AI governance
- DevSecOps standards
- Modernization support (IBM i, RPG, COBOL, Mainframe)
- AI Engineering (LangGraph, CrewAI, AutoGen, MCP, A2A, Bedrock)
- Data Science and MLOps
- RCA and Incident Management
- Project Memory Architecture

---

## Recommended Repository Structure

```text
EEIK/
├── .github/
│   ├── workflows/
│   ├── actions/
│   ├── ISSUE_TEMPLATE/
│   ├── CODEOWNERS
│   ├── PULL_REQUEST_TEMPLATE.md
│   ├── dependabot.yml
│   ├── copilot-instructions.md        ← repo-wide Copilot custom instructions
│   └── copilot/
│       ├── instructions/              ← *.instructions.md (file-pattern coding rules)
│       ├── prompts/                   ← *.prompt.md (reusable prompt files)
│       └── chatmodes/                 ← *.chatmode.md (custom Copilot Chat modes)
│
├── .claude/
│   ├── agents/                        ← subagent .md files (YAML frontmatter + system prompt)
│   ├── commands/                      ← slash command .md files
│   ├── hooks/                         ← shell scripts called by hooks in settings.json
│   ├── knowledge/                     ← reference knowledge dirs read/written by agents
│   ├── standards/                     ← engineering standards documents
│   ├── templates/
│   ├── architecture/
│   ├── governance/
│   ├── checklists/
│   ├── skills/
│   ├── playbooks/
│   ├── mcp/
│   └── settings.json                  ← permissions, hooks, model config, env vars
│
├── CLAUDE.md                          ← injected into every Claude Code session
├── README.md
└── docs/
```

---

## Agent File Format

Each agent lives at `.claude/agents/<name>.md`. The format is YAML frontmatter followed
by a markdown system prompt body:

```markdown
---
name: enterprise-architect
description: Senior enterprise architect for AWS-first systems. Use for system design, HLD/LLD, ADRs, RFCs, and architecture governance.
model: sonnet
tools: Read, Glob, Grep, Write, Edit, Bash
memory: project
---

You are a senior enterprise architect specializing in AWS-first, Java 21/25 systems...
```

Key frontmatter fields:

| Field | Required | Notes |
|-------|----------|-------|
| `name` | Yes | Lowercase, hyphens only. Must be unique. |
| `description` | Yes | Used by Claude to decide when to delegate. Write clearly. |
| `model` | No | `sonnet`, `opus`, `haiku`, full model ID, or `inherit` (default) |
| `tools` | No | Allowlist of tools. Inherits all if omitted. |
| `disallowedTools` | No | Denylist. Applied before `tools`. |
| `memory` | No | `project` → `.claude/agent-memory/<name>/MEMORY.md` (versioned); `user` → `~/.claude/agent-memory/`; `local` → unversioned |
| `permissionMode` | No | `default`, `acceptEdits`, `auto`, `dontAsk`, `bypassPermissions`, `plan` |
| `maxTurns` | No | Cap on agentic loop turns |
| `hooks` | No | Agent-scoped hooks (PreToolUse, PostToolUse, etc.) |
| `mcpServers` | No | MCP servers scoped to this agent |
| `skills` | No | Skills to preload into agent context at startup |
| `isolation` | No | `worktree` for isolated git worktree |
| `background` | No | `true` to always run as background task |
| `effort` | No | `low`, `medium`, `high`, `xhigh`, `max` |
| `color` | No | Display color in task list |

Agents are stored in `.claude/agents/` (project scope) or `~/.claude/agents/` (user scope).
Claude Code scans subdirectories recursively — agents can be organized into subfolders.

---

## Major Agent Domains

### Architecture
- Enterprise Architect
- Solution Architect
- Cloud Architect
- Security Architect
- Data Architect
- Event Driven Architect
- Platform Architect
- Integration Architect
- API Architect

### Development
- Java 17 Engineer
- Java 21 Engineer
- Java 25 Engineer
- Spring Boot Engineer
- Spring AI Engineer
- Python Enterprise Engineer
- FastAPI Engineer
- React Engineer
- Angular Engineer
- Micro Frontend Engineer

### Modernization
- IBM i Architect
- RPG Specialist
- COBOL Specialist
- Mainframe Architect
- JCL Specialist
- CICS Specialist
- IMS Specialist
- DB2 Specialist
- Strangler Migration Specialist

### Code Explanation
- Java Code Explainer
- Spring Code Explainer
- Python Code Explainer
- React Code Explainer
- Angular Code Explainer
- RPG Code Explainer
- COBOL Code Explainer
- JCL Code Explainer
- Terraform Code Explainer
- CDK Code Explainer

### AI Engineering
- AI Architect
- Prompt Engineer
- LangGraph Architect
- LangGraph Engineer
- CrewAI Architect
- CrewAI Engineer
- AutoGen Architect
- MCP Architect
- MCP Engineer
- A2A Architect
- Bedrock Architect
- RAG Architect
- Evaluation Specialist
- Hallucination Reviewer
- Trustworthiness Reviewer

### Data Science
- Data Scientist
- ML Engineer
- MLOps Engineer
- Forecasting Specialist
- Classification Specialist
- Feature Engineering Specialist
- Analytics Engineer
- BI Architect
- QuickSight Specialist

### DevSecOps
- DevOps Architect
- Platform Engineer
- CI/CD Specialist
- Security Reviewer
- IAM Specialist
- SRE Specialist
- Observability Specialist
- OpenTelemetry Specialist
- Grafana Specialist
- CloudWatch Specialist

### Quality Engineering
- Test Architect
- Unit Test Specialist
- Integration Test Specialist
- Contract Test Specialist
- Coverage Specialist
- JaCoCo Specialist
- SonarQube Specialist
- Performance Engineer
- Load Testing Specialist

### Delivery
- Scrum Master
- Agile Coach
- Product Owner
- Business Analyst
- Estimation Specialist
- Release Manager
- Program Manager

### Operations
- Incident Commander
- RCA Investigator
- Problem Manager
- Availability Specialist
- Resiliency Specialist

### Insurance Domain
- Claims Architect
- Claims Handler Assistant
- Fraud Specialist
- Underwriting Specialist
- Policy Administration Specialist

---

## Command Categories

Commands live in `.claude/commands/<name>.md`. Each file defines purpose, inputs, output
format, and which agents to delegate to.

- /design
- /review
- /estimate
- /refactor
- /modernize
- /generate-tests
- /generate-diagram
- /create-adr
- /create-rfc
- /security-review
- /performance-review
- /cost-review
- /aws-review
- /java-review
- /python-review
- /react-review
- /angular-review
- /langgraph-review
- /mcp-review
- /rca
- /release-plan

---

## Hook System

Hooks are configured in `.claude/settings.json` and call shell scripts in `.claude/hooks/`.
Claude Code hook events map to EEIK hook concepts as follows:

### SessionStart
**What it does:** Fires when a session begins. Use to load project context, memory summaries,
and inject architecture standards.
**Maps to:** "Pre Prompt" concept from EEIK blueprint.

```json
"SessionStart": [{ "hooks": [{ "type": "command", "command": ".claude/hooks/session-start.sh" }] }]
```

### UserPromptSubmit
**What it does:** Fires each time the user submits a prompt. Use to inject relevant standards
or context based on the prompt content.
**Maps to:** Per-prompt standards injection.

### PreToolUse
**What it does:** Fires before a tool executes. Can block the call. Use to validate destructive
Bash commands, enforce conventions.
**Matcher examples:** `Bash`, `Edit|Write`, `mcp__.*`

### PostToolUse
**What it does:** Fires after a tool succeeds. Use to run linters/formatters after file edits.
**Matcher examples:** `Edit|Write` (trigger lint), `Bash` (log commands)

### Git Pre-Commit Hook (`.git/hooks/pre-commit`)
**What it does:** Validates lint, tests, coverage before a git commit.
**Note:** This is a standard git hook, separate from Claude Code hooks.
**Maps to:** "Pre Commit" concept from EEIK blueprint.

### PR/Release Automation
**What it does:** Architecture reviews, security scans, AI reviews, changelog generation.
**Note:** Implemented as GitHub Actions workflows (`.github/workflows/`), not Claude hooks.
**Maps to:** "Pull Request" and "Release" hook concepts from EEIK blueprint.

### Available Claude Code Hook Events (complete list)
`SessionStart`, `SessionEnd`, `UserPromptSubmit`, `UserPromptExpansion`, `PreToolUse`,
`PostToolUse`, `PostToolUseFailure`, `PermissionRequest`, `PermissionDenied`,
`SubagentStart`, `SubagentStop`, `Stop`, `StopFailure`, `PreCompact`, `PostCompact`,
`FileChanged`, `CwdChanged`, `ConfigChange`, `InstructionsLoaded`, `Notification`,
`Elicitation`, `ElicitationResult`, `WorktreeCreate`, `WorktreeRemove`, `MessageDisplay`,
`TeammateIdle`, `TaskCreated`, `TaskCompleted`

---

## Memory Architecture

EEIK uses two distinct memory mechanisms:

### 1. Agent Persistent Memory (Claude-managed)
Set `memory: project` in an agent's frontmatter. Claude Code automatically creates and
maintains `.claude/agent-memory/<agent-name>/MEMORY.md`. The first 200 lines are
injected into the agent's context at startup for cross-session learning.

Use for: agent-specific learnings that accumulate over time (patterns, recurring issues,
codebase insights).

### 2. Project Knowledge Directories (agent-read/write)
Plain directories in `.claude/knowledge/` that any agent can read or write during tasks.
These are shared reference knowledge, not agent-specific.

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

### Memory Category Summary

| Category | Mechanism | Location |
|----------|-----------|----------|
| Architecture Memory | Agent memory | `.claude/agent-memory/enterprise-architect/` |
| ADR Memory | Knowledge dir | `.claude/knowledge/architecture-decisions/` |
| Domain Memory | Knowledge dir | `.claude/knowledge/domain-models/` |
| Technical Memory | Knowledge dir | `.claude/knowledge/` |
| Delivery Memory | Agent memory | `.claude/agent-memory/scrum-master/` |
| Incident Memory | Agent memory | `.claude/agent-memory/incident-commander/` |
| Project Memory | CLAUDE.md + knowledge dirs | Root CLAUDE.md + `.claude/knowledge/` |
| Team Memory | Agent memory | `.claude/agent-memory/` per agent |
| Governance Memory | Knowledge dir | `.claude/knowledge/architecture-decisions/` |
| AI Evaluation Memory | Agent memory | `.claude/agent-memory/evaluation-specialist/` |

---

## GitHub Workflow Categories

- Build
- Unit Tests
- Integration Tests
- Contract Tests
- Coverage
- SonarQube
- Dependency Scan
- Container Scan
- SAST
- DAST
- Terraform Validate
- Terraform Plan
- CDK Synth
- CDK Deploy
- Release
- Changelog
- Architecture Review
- AI Review
- Performance Review
- Cost Review
- Security Review

---

## Model Routing Strategy

### Claude Sonnet 4.6
- Architecture
- Modernization
- Complex design reviews
- AI architecture
- Deep reasoning

### Claude Sonnet 4.5
- Implementation
- Refactoring
- Documentation
- Code reviews
- Test generation

### GPT-5.5
- Estimation
- RCA
- Program management
- Business analysis
- Requirement decomposition

### GitHub Copilot
- Inline coding assistance
- Boilerplate generation
- Developer productivity

---

## Future Expansion Areas

- Enterprise Knowledge Graph
- Architecture Decision Graphs
- AI Governance Dashboards
- Agent Collaboration Workflows
- Delivery Intelligence
- FinOps Intelligence
- Engineering Metrics Intelligence
