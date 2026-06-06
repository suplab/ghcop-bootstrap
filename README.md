# GitHub Copilot Bootstrap Repository

A ready-to-fork workspace seed that provisions GitHub Copilot with rich, structured context for enterprise software development programs. Drop the files from this repository into any project and Copilot becomes a context-aware assistant that understands your technology stack, coding conventions, specialist agent roles, and operational processes — from day one.

This is **not a runnable application**. It is a configuration and context layer for GitHub Copilot.

---

## Repository Structure

This repository follows the [GitHub Copilot Customization Cheat Sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet) recommended layout:

```
.github/
├── copilot-instructions.md          ← Always-on repo-wide context (auto-loaded)
├── instructions/                    ← Path-specific instructions (auto-applied by file type)
│   ├── spring-boot.instructions.md
│   ├── java-legacy.instructions.md
│   ├── angular.instructions.md
│   ├── mainframe.instructions.md
│   ├── sql.instructions.md
│   ├── test.instructions.md
│   ├── aws-architecture.instructions.md
│   ├── containerisation.instructions.md
│   ├── cdk-terraform.instructions.md
│   ├── enterprise-architecture.instructions.md
│   ├── cicd.instructions.md
│   ├── aws-data-ml-ai.instructions.md
│   ├── incident-ops.instructions.md
│   ├── project-estimation.instructions.md
│   └── deployment.instructions.md
├── agents/                          ← Custom agent personas (select via @ dropdown)
│   ├── developer.agent.md           ← 10 migrated from prompts/agents/
│   ├── ... (10 original agents)
│   └── ... (22 new specialist agents)
├── prompts/                         ← Reusable prompt templates (manual invocation)
│   ├── tasks/                       ← 13 single-purpose task prompts
│   └── workflows/                   ← 6 multi-step orchestrated workflows
├── skills/                          ← Auto-loaded skill bundles
│   ├── estimation/SKILL.md
│   ├── jacoco-analysis/SKILL.md
│   ├── aws-cdk-deploy/SKILL.md
│   ├── incident-response/SKILL.md
│   └── code-quality-scan/SKILL.md
└── hooks/                           ← Lifecycle hooks (session, tool-use logging)
    ├── session-hooks.json
    └── tool-use-hooks.json
AGENTS.md                            ← Full agent catalogue and quick-reference
```

---

## Supported Technology Domains

| Domain | Stack |
|--------|-------|
| **Legacy Java** | Spring 4.x/5.x, Spring MVC, JdbcTemplate, Maven multi-module |
| **Modern Java** | Spring Boot 3.x, Java 17/21, Spring Data JPA, Spring Security 6.x, OpenAPI 3 |
| **Angular** | Angular 15+, Standalone Components, Signals API, NgRx, RxJS |
| **Mainframe** | IBM COBOL 6.x, Assembler (HLASM), JCL, CICS, DB2 z/OS |
| **AWS** | CDK (TypeScript), Terraform HCL, ECS/EKS, Lambda, RDS, SageMaker, Bedrock |
| **Data / ML / AI** | SageMaker, Glue, Athena, Bedrock, LangChain, RAG pipelines |
| **Platform** | Docker, Kubernetes, GitHub Actions, Jenkins, CloudWatch |

---

## How to Adopt This Bootstrap

### 1. Copy the Configuration Files

```bash
cp -r .github/        /path/to/your-project/.github/
cp -r .vscode/        /path/to/your-project/.vscode/
cp    .editorconfig   /path/to/your-project/.editorconfig
cp    AGENTS.md       /path/to/your-project/AGENTS.md
```

### 2. Customise Master Instructions

Open `.github/copilot-instructions.md` and update:
- **Program Context** — describe your specific project and domains
- **Technology Stack Summary** — reflect your actual versions
- **Dependency Policy** — your real `pom.xml` / `package.json` baseline

### 3. Adjust `applyTo` Glob Patterns

Each file in `.github/instructions/` has an `applyTo` frontmatter field. Update to match your source layout:

```markdown
---
applyTo: "src/main/java/com/yourcompany/**/*.java"
---
```

### 4. Remove Unused Domains

If your project has no mainframe code, delete:
- `.github/instructions/mainframe.instructions.md`
- `.github/agents/modernization-expert.agent.md`
- `.github/prompts/tasks/modernize-cobol-to-java.prompt.md`
- `.github/prompts/workflows/cobol-to-java-workflow.prompt.md`

### 5. Open in VS Code

Accept the recommended extensions prompt, or run:
`Extensions: Show Recommended Extensions` from the Command Palette.

### 6. Verify Context is Loading

Open any `.java` file and ask Copilot Chat:
> "What coding standards apply to this file?"

Copilot should describe the standards from `copilot-instructions.md` and the relevant `*.instructions.md` file.

---

## Agent Catalogue (32 agents)

All agents live in `.github/agents/` as `.agent.md` files. Select via the **@** dropdown in Copilot Chat. Full catalogue and descriptions: see **`AGENTS.md`** at the repository root.

### Java Development
| Agent | Name | Primary Use |
|-------|------|------------|
| `java-dev.agent.md` | Java Developer | Ticket-scoped Spring Boot implementation |
| `java-tech-lead.agent.md` | Java Tech Lead | PR gating, standards, tech debt |
| `java-tester.agent.md` | Java Test Engineer | JUnit 5 / Testcontainers test suites |
| `jacoco-coverage-tester.agent.md` | JaCoCo Coverage Analyst | Coverage gap analysis |
| `developer.agent.md` | Senior Java/Angular Developer | Full-stack Java + Angular |

### Angular Development
| Agent | Name | Primary Use |
|-------|------|------------|
| `angular-dev.agent.md` | Angular Developer | Standalone components, signals, lazy routes |
| `angular-tester.agent.md` | Angular Test Engineer | Jasmine/TestBed specs |
| `angular-coverage-checker.agent.md` | Angular Coverage Analyst | Istanbul/Karma gap analysis |

### Architecture & Design
| Agent | Name | Primary Use |
|-------|------|------------|
| `architect.agent.md` | Solution Architect | ADRs, bounded contexts, API contracts |
| `enterprise-architect.agent.md` | Enterprise Architect | Capability maps, technology lifecycle, TOGAF |
| `aws-architect.agent.md` | AWS Solution Architect | Well-Architected reviews, CDK stacks |

### Code Quality & Security
| Agent | Name | Primary Use |
|-------|------|------------|
| `reviewer.agent.md` | Code Reviewer | [BLOCKER]/[MAJOR]/[MINOR]/[NIT] PR reviews |
| `security-auditor.agent.md` | Security Auditor | OWASP Top 10 + remediation code |
| `performance-reviewer.agent.md` | Performance Specialist | N+1 queries, resource leaks |
| `coverage-enforcer.agent.md` | Coverage Guardian | Gap analysis + targeted test generation |
| `test-quality-enforcer.agent.md` | Test Quality Inspector | Anti-pattern detection and regeneration |
| `tester.agent.md` | QA Automation Engineer | Full test pyramid |
| `analyst.agent.md` | Business Analyst | OpenAPI specs, Gherkin criteria |

### Infrastructure & Deployment
| Agent | Name | Primary Use |
|-------|------|------------|
| `cdk-terraform-helper.agent.md` | CDK / Terraform Helper | IaC stacks |
| `aws-deploy-helper.agent.md` | AWS Deploy Helper | Deploy commands, rollback runbooks |
| `local-deploy-helper.agent.md` | Local Deploy Helper | Docker Compose, smoke tests |
| `containerisation-helper.agent.md` | Containerisation Helper | Dockerfiles, K8s manifests |
| `ci-engineer.agent.md` | CI Engineer | GitHub Actions / Jenkins pipelines |

### Data, ML & AI (AWS Stack)
| Agent | Name | Primary Use |
|-------|------|------------|
| `data-scientist-aws.agent.md` | AWS Data Scientist | SageMaker notebooks, Glue ETL, Athena |
| `ml-engineer-aws.agent.md` | AWS ML Engineer | SageMaker pipelines, MLOps, model registry |
| `ai-engineer-aws.agent.md` | AWS AI Engineer | Bedrock LLM, RAG pipelines, guardrails |

### Delivery & Operations
| Agent | Name | Primary Use |
|-------|------|------------|
| `estimator.agent.md` | Estimator | Bottom-up estimates (8h/day × 80% = 6.4h/day) |
| `project-tracker.agent.md` | Project Tracker | Sprint burndown, story status, velocity |
| `ops-engineer.agent.md` | Ops Engineer | CloudWatch dashboards, alarms, runbooks |
| `incident-handler.agent.md` | Incident Handler | P1/P2 war room coordination |
| `rca-agent.agent.md` | RCA Agent | 5-Whys root cause analysis |

### Modernisation
| Agent | Name | Primary Use |
|-------|------|------------|
| `modernization-expert.agent.md` | Mainframe Modernization Specialist | COBOL → Java + semantic risk matrix |

---

## Instruction Files (Auto-Applied)

| File | `applyTo` Glob | Domain |
|------|---------------|--------|
| `spring-boot.instructions.md` | `**/src/main/java/**/*.java` | Spring Boot 3.x / Java 17/21 |
| `java-legacy.instructions.md` | `src/main/java/**/*.java` | Spring 4/5 / Java 8/11 |
| `angular.instructions.md` | `**/*.ts, **/*.html, **/*.scss` | Angular 15+ |
| `mainframe.instructions.md` | `**/*.cbl, **/*.asm, **/*.jcl` | COBOL / JCL / Assembler |
| `sql.instructions.md` | `**/*.sql, **/mapper/**/*.xml` | DB2 / MyBatis |
| `test.instructions.md` | `**/*Test.java, **/*.spec.ts` | JUnit 5 / Jasmine |
| `aws-architecture.instructions.md` | `**/*.tf, **/cdk/**/*.ts, **/template.yaml` | AWS CDK / Terraform / CloudFormation |
| `containerisation.instructions.md` | `**/Dockerfile*, **/docker-compose*.yml, **/*.k8s.yaml` | Docker / Kubernetes |
| `cdk-terraform.instructions.md` | `**/cdk.json, **/*.tf, **/infra/**/*.ts` | IaC (CDK + Terraform) |
| `enterprise-architecture.instructions.md` | `**/architecture/**, **/adr/**` | EA artifacts |
| `cicd.instructions.md` | `**/.github/workflows/**, **/Jenkinsfile` | CI/CD pipelines |
| `aws-data-ml-ai.instructions.md` | `**/*.ipynb, **/sagemaker/**, **/bedrock/**` | Data / ML / AI |
| `incident-ops.instructions.md` | `**/runbooks/**, **/incidents/**` | Ops / Incident / RCA |
| `project-estimation.instructions.md` | `**/estimates/**, **/*.estimate.md` | Estimation |
| `deployment.instructions.md` | `**/deploy/**, **/scripts/deploy*` | Deployment scripts |

---

## Skills (Auto-Loaded)

Skills in `.github/skills/` are automatically selected by Copilot when relevant:

| Skill | Triggers When |
|-------|--------------|
| `estimation` | Asked to estimate effort, size a story, or plan delivery |
| `jacoco-analysis` | Asked about JaCoCo reports, coverage thresholds, or missed branches |
| `aws-cdk-deploy` | Asked about CDK deploy commands, cdk diff, or rollback |
| `incident-response` | Declaring or managing a P1/P2 incident |
| `code-quality-scan` | Triaging SonarQube, SpotBugs, Checkstyle, or OWASP findings |

---

## Orchestrated Workflows (6 total)

| Workflow File | Agents Involved | Use For |
|--------------|----------------|---------|
| `full-feature-dev.prompt.md` | Analyst → Architect → Developer → Tester → Coverage → Reviewer | New feature end-to-end |
| `pr-review-workflow.prompt.md` | Reviewer → Security → Performance → Test Quality | Automated PR review |
| `tdd-cycle.prompt.md` | Analyst → Tester → Developer → Reviewer → Coverage | Test-Driven Development |
| `cobol-to-java-workflow.prompt.md` | Analyst → Modernization → Architect → Developer → Tester → Security | COBOL migration |
| `aws-infra-deploy.prompt.md` | AWS Architect → CDK Helper → CI Engineer → Deploy Helper → Ops | AWS infra delivery |
| `incident-rca-workflow.prompt.md` | Incident Handler → Ops → Incident Handler → RCA → Project Tracker | Incident to RCA |

---

## Estimator Formula

The `estimator.agent.md` uses this formula for all estimates:

> **Human Days = Σ Raw Hours ÷ 6.4**
>
> 6.4 = 8 hours/day × 80% efficiency (accounts for meetings, PR cycles, context-switching, interruptions)

Estimates include P50 / P80 / P90 confidence ranges. Commit to **P80** for sprints; use **P90** for release planning.

---

## Task Prompts Quick Reference

Reference task prompts in Copilot Chat with `#file:.github/prompts/tasks/<name>.prompt.md`:

| Prompt | Action |
|--------|--------|
| `generate-unit-tests.prompt.md` | Full JUnit 5 test class |
| `generate-integration-tests.prompt.md` | Spring Boot + Testcontainers |
| `code-review.prompt.md` | Structured single-class review |
| `generate-rest-api.prompt.md` | Controller + service + DTO + OpenAPI |
| `generate-angular-component.prompt.md` | Standalone component + spec |
| `generate-angular-service.prompt.md` | HttpClient service + spec |
| `add-javadoc.prompt.md` | Complete Javadoc on all public members |
| `add-logging.prompt.md` | SLF4J at correct levels throughout |
| `explain-code.prompt.md` | Plain-English explanation (COBOL-aware) |
| `explain-mainframe-program.prompt.md` | COBOL/JCL/Assembler walkthrough |
| `refactor-to-clean-code.prompt.md` | SOLID / clean code refactor |
| `modernize-cobol-to-java.prompt.md` | COBOL → Java with risk matrix |
| `generate-mapstruct-mapper.prompt.md` | MapStruct interface |
| `generate-openapi-spec.prompt.md` | OpenAPI 3.0 YAML spec |

---

## Hooks

Lifecycle hooks in `.github/hooks/` log Copilot activity to local log files:

| Hook File | Events | Log File |
|-----------|--------|---------|
| `session-hooks.json` | sessionStart, sessionEnd, userPromptSubmitted | `.copilot-session.log`, `.copilot-prompts.log` |
| `tool-use-hooks.json` | preToolUse, postToolUse, errorOccurred | `.copilot-tool.log`, `.copilot-errors.log` |

Add `*.log` to your `.gitignore` to keep local log files out of version control.

---

## IntelliJ / JetBrains Usage

GitHub Copilot in IntelliJ reads `.github/copilot-instructions.md` automatically. Access agents, instructions, and prompts via `#file:` references in Copilot Chat. See `intellij/` directory for plugin recommendations and settings guidance.

---

## Validation Checklist

Before using this bootstrap in a new project:

- [ ] `copilot-instructions.md` customised for the project (Program Context, stack versions, dependency policy)
- [ ] `applyTo` glob patterns in `*.instructions.md` updated for the project's source layout
- [ ] Unused domain instruction files removed (e.g., mainframe if no COBOL)
- [ ] At least one agent invoked and responding in the correct persona
- [ ] `pr-review-workflow` runs end-to-end on a sample PR
- [ ] VS Code recommended extensions installed
- [ ] `.editorconfig` enforcing formatting on save
- [ ] `AGENTS.md` added to repo root for team discoverability
- [ ] Log files (`.copilot-*.log`) added to `.gitignore`

---

## Contributing

1. Follow file naming: `<name>.agent.md`, `<name>.instructions.md`, `<name>.prompt.md`
2. Every `.agent.md` needs: `name`, `description`, `model`, `tools` in frontmatter
3. Every `.instructions.md` needs: `applyTo`, `description` in frontmatter
4. Every `SKILL.md` needs: `name`, `description` in frontmatter; `name` must match folder name
5. Test each new file by invoking it in Copilot Chat and verifying the response persona
6. Register new agents in `AGENTS.md` and add entries to the tables in `copilot-instructions.md`
