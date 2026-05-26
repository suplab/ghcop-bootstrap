# GitHub Copilot Seed Repository

A ready-to-fork workspace seed that provisions GitHub Copilot with rich, structured context for enterprise software development programs. Drop the files from this repository into any project and Copilot becomes a context-aware assistant that understands your technology stack, coding conventions, team roles, and modernization goals — from day one.

This is **not a runnable application**. It is a configuration and context layer for GitHub Copilot.

---

## What This Repository Contains

| Directory / File | Purpose |
|-----------------|---------|
| `.github/copilot-instructions.md` | Master context file — always loaded by Copilot for every interaction |
| `.github/instructions/` | Scoped instruction files — auto-loaded based on the file type being edited |
| `.github/prompts/agents/` | Role-based agent personas — invoke via Copilot Chat for specialist assistance |
| `.github/prompts/tasks/` | Single-purpose task prompts — generate code, tests, docs, and specs on demand |
| `.github/prompts/workflows/` | Multi-step orchestrated workflows — guide Copilot through end-to-end development flows |
| `.vscode/settings.json` | VS Code settings that wire up Copilot to use the instruction and prompt files |
| `.vscode/extensions.json` | Recommended extensions for the team |
| `.editorconfig` | Consistent formatting rules across all IDEs |

### Supported Technology Domains

| Domain | Stack |
|--------|-------|
| **Legacy Java** | Spring 4.x/5.x, Spring MVC, JdbcTemplate, Maven multi-module |
| **Modern Java** | Spring Boot 3.x, Java 17/21, Spring Data JPA, Spring Security 6.x, OpenAPI 3 |
| **Angular** | Angular 15+, Standalone Components, Signals API, NgRx, RxJS |
| **Mainframe** | IBM COBOL 6.x, Assembler (HLASM), JCL, CICS, DB2 z/OS |

---

## How to Adopt This Seed

### 1. Copy the Configuration Files

Clone or download this repository, then copy the following into the root of your target project:

```bash
cp -r .github/        /path/to/your-project/.github/
cp -r .vscode/        /path/to/your-project/.vscode/
cp    .editorconfig   /path/to/your-project/.editorconfig
```

### 2. Customise the Master Instructions

Open `.github/copilot-instructions.md` and update:
- The **Program Context** section to describe your specific project
- The **Technology Stack Summary** table to reflect your actual versions
- The **Dependency Policy** section with your real `pom.xml` / `package.json` baseline

### 3. Adjust Glob Patterns

Each file in `.github/instructions/` has an `applyTo` frontmatter field. Update these to match your project's source layout:

```markdown
---
applyTo: "src/main/java/com/yourcompany/**/*.java"
---
```

### 4. Open in VS Code

On first open, VS Code will prompt you to install recommended extensions. Accept the prompt or run:
```
Extensions: Show Recommended Extensions
```
from the Command Palette (`Ctrl+Shift+P` / `Cmd+Shift+P`).

### 5. Verify Copilot Context is Loading

Open any `.java` file and start a Copilot Chat session. Ask:
> "What coding standards apply to this file?"

Copilot should describe the standards from `copilot-instructions.md` and the relevant `*.instructions.md` file.

---

## Agent Quick Reference

Invoke an agent by referencing its file in Copilot Chat:
```
#file:.github/prompts/agents/architect.prompt.md
```
Or, if your team has configured slash commands, use the short form.

| Agent | Role | When to Invoke |
|-------|------|---------------|
| `architect.prompt.md` | Solution Architect | Designing components, validating patterns, producing ADRs |
| `developer.prompt.md` | Senior Developer | Writing production-grade implementation code |
| `reviewer.prompt.md` | Code Reviewer | Structured pull request and code review |
| `tester.prompt.md` | QA Engineer | Generating complete test suites |
| `analyst.prompt.md` | Business Analyst | Translating requirements to specs and acceptance criteria |
| `coverage-enforcer.prompt.md` | Coverage Guardian | Identifying untested branches, mandating remediation |
| `test-quality-enforcer.prompt.md` | Test Quality Inspector | Detecting meaningless or anti-pattern tests |
| `modernization-expert.prompt.md` | Modernization Specialist | COBOL/Assembler → Java translation and analysis |
| `security-auditor.prompt.md` | Security Auditor | OWASP Top 10 review, security findings report |
| `performance-reviewer.prompt.md` | Performance Reviewer | N+1 queries, resource leaks, scalability issues |

---

## Scoped Instruction Files

These files are auto-loaded by Copilot when you open a matching file type:

| Instruction File | Applies To | Key Concerns |
|-----------------|-----------|-------------|
| `spring-boot.instructions.md` | Spring Boot Java sources | Java 17/21 idioms, Jakarta EE, Spring Boot 3.x patterns |
| `java-legacy.instructions.md` | Legacy Spring MVC sources | Spring 4/5, JdbcTemplate, XML config, Java 8/11 |
| `angular.instructions.md` | `.ts`, `.html`, `.scss` files | Standalone components, Signals, OnPush, BEM |
| `mainframe.instructions.md` | `.cbl`, `.asm`, `.jcl`, `.cpy` | COBOL structure, CICS, DB2 embedded SQL, JCL |
| `sql.instructions.md` | `.sql`, MyBatis XML mappers | DB2 dialect, parameterized queries, schema qualification |
| `test.instructions.md` | `*Test.java`, `*IT.java`, `*.spec.ts` | JUnit 5, AssertJ, Mockito, naming conventions |

---

## Task Prompt Usage Guide

Task prompts perform a single, bounded action. Reference them in Copilot Chat with:

```
#file:.github/prompts/tasks/generate-unit-tests.prompt.md

Generate tests for the CustomerService class.
```

| Prompt | Action |
|--------|--------|
| `generate-unit-tests.prompt.md` | Full JUnit 5 test class for a given class |
| `generate-integration-tests.prompt.md` | Spring Boot integration test with Testcontainers |
| `code-review.prompt.md` | Structured review of selected code |
| `explain-code.prompt.md` | Plain-English explanation (COBOL-aware) |
| `refactor-to-clean-code.prompt.md` | SOLID/clean code refactor of selected code |
| `add-javadoc.prompt.md` | Complete Javadoc on all public members |
| `add-logging.prompt.md` | SLF4J logging at correct levels throughout a class |
| `generate-rest-api.prompt.md` | Controller + service + DTO + OpenAPI annotations |
| `generate-angular-component.prompt.md` | Standalone Angular component with spec file |
| `generate-angular-service.prompt.md` | Angular HttpClient service with typed models |
| `modernize-cobol-to-java.prompt.md` | Full COBOL → Java translation with risk matrix |
| `explain-mainframe-program.prompt.md` | Plain-English explanation of COBOL/JCL/Assembler |
| `generate-mapstruct-mapper.prompt.md` | MapStruct mapper interface between two classes |
| `generate-openapi-spec.prompt.md` | OpenAPI 3.0 YAML for a described REST interface |

---

## Workflow Guide

Workflows orchestrate multiple agents through a multi-step process.

### Full Feature Development (`full-feature-dev.prompt.md`)

A sequential workflow for implementing a new feature end-to-end:

```
Step 1 → [Analyst]              Define acceptance criteria + OpenAPI contract
Step 2 → [Architect]            Validate design against bounded context
Step 3 → [Developer]            Implement service + controller + repository
Step 4 → [Tester]               Generate unit + integration tests
Step 5 → [Coverage Enforcer]    Verify all branches covered
Step 6 → [Reviewer]             Final structured review before PR
```

**How to run:** Open the workflow file in Copilot Chat, then provide your feature description as the input. Follow each step in sequence, copying the output of one step as the input to the next.

---

### PR Review Workflow (`pr-review-workflow.prompt.md`)

Automated PR review pipeline — produces a consolidated review comment:

```
Step 1 → [Reviewer]               Correctness + style review
Step 2 → [Security Auditor]       Security scan of changed files
Step 3 → [Performance Reviewer]   Performance issues in changed code
Step 4 → [Test Quality Enforcer]  Coverage and test quality validation
```

**How to run:** Paste the diff or changed file list, then run each step. Combine all findings into a single PR comment.

---

### TDD Cycle (`tdd-cycle.prompt.md`)

Test-Driven Development flow:

```
Step 1 → [Analyst]            Define test cases from acceptance criteria (red)
Step 2 → [Tester]             Write failing tests (red phase)
Step 3 → [Developer]          Write minimal implementation to pass tests (green)
Step 4 → [Reviewer]           Refactor check (refactor phase)
Step 5 → [Coverage Enforcer]  Confirm no branches missed
```

---

### COBOL to Java Modernization (`cobol-to-java-workflow.prompt.md`)

Modernization pipeline for a single COBOL program:

```
Step 1 → [Analyst]                 Extract business rules in plain English
Step 2 → [Modernization Expert]    Produce Java skeleton + semantic risk matrix
Step 3 → [Architect]               Map to correct Java service boundary
Step 4 → [Developer]               Complete the Java implementation
Step 5 → [Tester]                  Generate tests against extracted business rules
Step 6 → [Security Auditor]        Check for security concerns in ported logic
```

---

## Customisation Guide

### Adding a New Technology Domain

1. Create `.github/instructions/<technology>.instructions.md`
2. Set the `applyTo` glob pattern for the relevant file extensions
3. Follow the standard structure: Context → Coding Standards → Preferred Patterns → Anti-Patterns → Dependencies → Test Conventions
4. Reference the new file from `.vscode/settings.json` if it needs explicit wiring

### Overriding for a Specific Project Module

Create a second instructions file with a more specific glob:

```markdown
---
applyTo: "src/main/java/com/example/payments/**/*.java"
---
## Context
This module handles payment processing. PCI-DSS compliance rules apply...
```

More specific globs take precedence over broader ones.

### Adding a Project-Specific Agent

Copy an existing agent prompt file and modify:
1. The `description` frontmatter
2. The **Role** and **Capabilities** sections
3. The **Constraints** to match project-specific restrictions

### Removing Unused Domains

If your project has no mainframe code, simply delete:
- `.github/instructions/mainframe.instructions.md`
- `.github/prompts/agents/modernization-expert.prompt.md`
- `.github/prompts/tasks/modernize-cobol-to-java.prompt.md`
- `.github/prompts/tasks/explain-mainframe-program.prompt.md`
- `.github/prompts/workflows/cobol-to-java-workflow.prompt.md`

---

## IntelliJ / JetBrains Usage

GitHub Copilot in IntelliJ IDEA reads:
- `.github/copilot-instructions.md` — loaded automatically for all Copilot interactions
- `.github/instructions/*.instructions.md` — accessible via Copilot Chat `#file:` reference
- `.github/prompts/*.prompt.md` — accessible via Copilot Chat `#file:` reference

**What does not apply in IntelliJ:**
- `.vscode/settings.json` — ignored; configure Copilot via IntelliJ's own settings panel
- `.vscode/extensions.json` — ignored; see `intellij/recommended-plugins.md` for equivalents

See the `intellij/` directory at the root of this repository for IntelliJ-specific setup guidance.

---

## Validation Checklist

Before using this seed in production, verify:

- [ ] `copilot-instructions.md` loads without error in Copilot Chat (ask "what are the coding standards?")
- [ ] Each `*.instructions.md` triggers on the correct file type
- [ ] Each agent prompt responds in the correct persona
- [ ] `pr-review-workflow` produces a complete structured review
- [ ] `modernize-cobol-to-java` correctly maps a sample COBOL program
- [ ] VS Code shows recommended extensions on workspace open
- [ ] `.editorconfig` enforces formatting on save
- [ ] No hardcoded paths or project-specific values remain in template files
- [ ] Glob patterns in `applyTo` fields updated for your project layout

---

## Contributing

To extend this seed:

1. Follow the file naming conventions: `<name>.instructions.md`, `<name>.prompt.md`
2. Use the standard frontmatter structure for each file type
3. Test each new file by invoking it in Copilot Chat and verifying the response persona
4. Update this README with the new file in the appropriate table
