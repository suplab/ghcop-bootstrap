# Agent Catalogue

This file indexes all GitHub Copilot agents available in this bootstrap repository. Each agent is a specialist AI persona defined as a `.agent.md` file in `.github/agents/`. Select an agent from the Copilot Chat agent dropdown to activate it.

---

## How to Use Agents

1. Open GitHub Copilot Chat in VS Code
2. Click the **@** icon in the chat input (or type `@`)
3. Select the agent by its **name** from the dropdown
4. The agent's persona, tools, and instructions activate for that session

---

## Agent Index

### Java Development

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/java-dev.agent.md` | Java Developer | Implementing a specific Spring Boot ticket (service, controller, DTO, repository) |
| `.github/agents/java-tech-lead.agent.md` | Java Tech Lead | Gating a PR, enforcing standards, classifying tech debt, mentoring |
| `.github/agents/java-tester.agent.md` | Java Test Engineer | Writing JUnit 5 unit, slice, and Testcontainers integration tests |
| `.github/agents/jacoco-coverage-tester.agent.md` | JaCoCo Coverage Analyst | Analysing JaCoCo reports and closing coverage gaps |
| `.github/agents/developer.agent.md` | Senior Java/Angular Developer | Full-stack implementation (Java + Angular together) |

### Angular Development

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/angular-dev.agent.md` | Angular Developer | Building standalone components, services, signals, lazy routes |
| `.github/agents/angular-tester.agent.md` | Angular Test Engineer | Writing Jasmine/TestBed component and service specs |
| `.github/agents/angular-coverage-checker.agent.md` | Angular Coverage Analyst | Analysing Istanbul/Karma coverage and closing gaps |

### Architecture & Design

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/architect.agent.md` | Solution Architect | ADRs, bounded context design, API contracts, non-functional requirements |
| `.github/agents/enterprise-architect.agent.md` | Enterprise Architect | Capability maps, technology lifecycle, TOGAF-aligned EA artifacts |
| `.github/agents/aws-architect.agent.md` | AWS Solution Architect | AWS Well-Architected reviews, CDK stacks, cost estimates, VPC/ECS/RDS design |

### Code Review & Quality

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/reviewer.agent.md` | Code Reviewer | PR reviews with [BLOCKER]/[MAJOR]/[MINOR]/[NIT] findings |
| `.github/agents/security-auditor.agent.md` | Application Security Auditor | OWASP Top 10 audit with CVSS ratings and remediation code |
| `.github/agents/performance-reviewer.agent.md` | Performance Specialist | N+1 queries, unbounded results, resource leaks, rendering performance |
| `.github/agents/coverage-enforcer.agent.md` | Coverage Guardian | Coverage gap analysis and targeted test generation |
| `.github/agents/test-quality-enforcer.agent.md` | Test Quality Inspector | Detecting and fixing test anti-patterns |

### Testing

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/tester.agent.md` | QA Automation Engineer | Full test pyramid (unit, slice, integration, contract) for any stack |
| `.github/agents/analyst.agent.md` | Business Analyst | OpenAPI specs, Gherkin acceptance criteria, data models |

### Infrastructure & Deployment

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/cdk-terraform-helper.agent.md` | CDK / Terraform Helper | Writing CDK TypeScript stacks or Terraform HCL modules |
| `.github/agents/aws-deploy-helper.agent.md` | AWS Deploy Helper | Generating deploy commands, pre-deploy checklists, rollback runbooks |
| `.github/agents/local-deploy-helper.agent.md` | Local Deploy Helper | Docker Compose setup, local env scripts, smoke tests |
| `.github/agents/containerisation-helper.agent.md` | Containerisation Helper | Dockerfiles, K8s manifests, Helm charts, container security |
| `.github/agents/ci-engineer.agent.md` | CI Engineer | GitHub Actions and Jenkins pipelines with quality gates |

### Data, ML & AI (AWS)

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/data-scientist-aws.agent.md` | AWS Data Scientist | SageMaker notebooks, Glue ETL, Athena, feature engineering |
| `.github/agents/ml-engineer-aws.agent.md` | AWS ML Engineer | SageMaker training pipelines, model registry, MLOps, inference endpoints |
| `.github/agents/ai-engineer-aws.agent.md` | AWS AI Engineer | Bedrock LLM integration, RAG pipelines, prompt engineering, guardrails |

### Delivery & Operations

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/estimator.agent.md` | Estimator | Bottom-up effort estimates (8h/day × 80% efficiency = 6.4h/day) |
| `.github/agents/project-tracker.agent.md` | Project Tracker | Sprint burndown, story status, velocity trends, blocker escalation |
| `.github/agents/ops-engineer.agent.md` | Ops Engineer | CloudWatch dashboards, alarms, auto-scaling, cost optimisation, runbooks |
| `.github/agents/incident-handler.agent.md` | Incident Handler | P1/P2 war room coordination, ITIL status updates, stakeholder comms |
| `.github/agents/rca-agent.agent.md` | RCA Agent | 5-Whys root cause analysis, timeline reconstruction, corrective actions |

### Modernisation

| Agent File | Name | Use When |
|-----------|------|---------|
| `.github/agents/modernization-expert.agent.md` | Mainframe Modernization Specialist | COBOL/JCL/Assembler → Java migration with semantic risk matrix |

---

## Agent Skills

The following reusable skills are available in `.github/skills/` and are automatically loaded by Copilot when relevant:

| Skill Folder | Name | Triggers When |
|-------------|------|--------------|
| `estimation/` | estimation | Asked to estimate, size, or plan effort |
| `jacoco-analysis/` | jacoco-analysis | Asked about JaCoCo, coverage thresholds, or missed branches |
| `aws-cdk-deploy/` | aws-cdk-deploy | Asked about CDK deploy, cdk diff, or stack rollback |
| `incident-response/` | incident-response | Declaring or managing a P1/P2 incident |
| `code-quality-scan/` | code-quality-scan | Triaging SonarQube, SpotBugs, Checkstyle, or OWASP findings |

---

## Orchestrated Workflows

Invoke these prompt files via `#file:` in Copilot Chat for multi-agent workflows:

| Prompt File | Workflow |
|------------|---------|
| `.github/prompts/workflows/full-feature-dev.prompt.md` | End-to-end feature: Analyst → Architect → Developer → Tester → Coverage → Reviewer |
| `.github/prompts/workflows/pr-review-workflow.prompt.md` | Automated PR scan: Code → Security → Performance → Test Quality |
| `.github/prompts/workflows/tdd-cycle.prompt.md` | Red → Green → Refactor → Coverage |
| `.github/prompts/workflows/cobol-to-java-workflow.prompt.md` | COBOL modernisation pipeline |
| `.github/prompts/workflows/aws-infra-deploy.prompt.md` | AWS infra: Architect → CDK → CI/CD → Deploy → Smoke Test |
| `.github/prompts/workflows/incident-rca-workflow.prompt.md` | Incident: Detection → Triage → War Room → Resolution → RCA |
