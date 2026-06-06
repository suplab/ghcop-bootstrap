---
name: code-quality-scan
description: 'Interpret SonarQube/SonarCloud quality gate results, SpotBugs findings, Checkstyle violations, and OWASP dependency-check reports. Use to triage and resolve code quality issues.'
---

# Code Quality Scan Skill

Interpreting and resolving code quality tool findings for Java Spring Boot projects.

## Quality Tool Stack

| Tool | Purpose | Trigger |
|------|---------|---------|
| SonarQube/SonarCloud | SAST, code smells, duplication | CI pipeline |
| SpotBugs | Bytecode bug patterns | `mvn spotbugs:check` |
| Checkstyle | Style enforcement (Google Java Style) | `mvn checkstyle:check` |
| OWASP Dependency-Check | Known CVE dependencies | `mvn dependency-check:check` |
| JaCoCo | Coverage enforcement | `mvn jacoco:check` |

## SonarQube Quality Gate Interpretation

### Severity Levels

| Level | Description | Action |
|-------|-------------|--------|
| BLOCKER | Critical bug or security vulnerability | Fix before merge — pipeline fails |
| CRITICAL | High-impact issue | Fix before merge |
| MAJOR | Medium-impact issue | Fix in current sprint |
| MINOR | Low-impact issue | Backlog or next PR |
| INFO | Suggestion | Optional |

### Common SonarQube Rules

| Rule | Description | Fix |
|------|-------------|-----|
| `java:S2095` | Resources not closed | Wrap in try-with-resources |
| `java:S1192` | String literal duplicated | Extract to constant |
| `java:S2139` | Exception caught and re-thrown | Log or rethrow, not both |
| `java:S3776` | Cognitive complexity too high | Extract methods |
| `java:S4970` | `Optional.get()` without check | Use `orElseThrow()` |

## SpotBugs Categories

| Category | Description |
|----------|-------------|
| `CORRECTNESS` | Probable bugs (null deref, wrong equals) |
| `SECURITY` | Security vulnerabilities |
| `PERFORMANCE` | Performance issues |
| `STYLE` | Code style issues |
| `MT_CORRECTNESS` | Multithreading issues |

## OWASP Dependency-Check Thresholds

| CVSS Score | Severity | CI Action |
|-----------|---------|----------|
| ≥ 9.0 | CRITICAL | Fail build immediately |
| 7.0–8.9 | HIGH | Fail build |
| 4.0–6.9 | MEDIUM | Warning; fix within sprint |
| < 4.0 | LOW | Informational |

To suppress a false positive:
```xml
<!-- In dependency-check-suppressions.xml -->
<suppress>
  <notes>False positive: library X is used in test scope only</notes>
  <cve>CVE-YYYY-NNNNN</cve>
</suppress>
```

## Maven Commands Reference

```bash
# Run all quality checks
mvn clean verify checkstyle:check spotbugs:check dependency-check:check jacoco:check

# SonarQube scan
mvn sonar:sonar -Dsonar.projectKey=${SONAR_KEY} -Dsonar.token=${SONAR_TOKEN}

# Generate all reports without failing build
mvn clean verify -Dcheckstyle.failsOnError=false -Dspotbugs.failOnError=false
```
