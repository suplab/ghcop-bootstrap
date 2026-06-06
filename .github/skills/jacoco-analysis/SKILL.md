---
name: jacoco-analysis
description: 'Analyse JaCoCo XML/HTML coverage reports to identify uncovered lines and branches in Java business logic. Use when asked to check, improve, or enforce code coverage on Java Spring Boot projects.'
---

# JaCoCo Analysis Skill

Parses JaCoCo coverage reports and maps uncovered paths to targeted test generation.

## Report Locations

| Format | Path |
|--------|------|
| XML | `target/site/jacoco/jacoco.xml` |
| HTML | `target/site/jacoco/index.html` |
| Aggregate | `target/site/jacoco-aggregate/jacoco.xml` |

**To regenerate:** `mvn clean test jacoco:report`

## Coverage Thresholds

| Class Type | Line | Branch |
|-----------|------|--------|
| Domain services | 80% | 70% |
| REST controllers | 75% | 60% |
| Utilities | 80% | 70% |
| `*MapperImpl.java` | Excluded | Excluded |
| `*Config.java` | Excluded | Excluded |

## Analysis Process

1. Run `mvn clean test jacoco:report`
2. Open `target/site/jacoco/jacoco.xml`
3. Filter to classes with `<counter type="LINE" missed=">0"/>`
4. Map each missed line back to source code
5. Classify the uncovered branch type: `if/else`, `catch`, `Optional`, `switch`
6. Assess risk: HIGH (business logic/payment), MEDIUM (validation), LOW (utility)
7. Generate targeted JUnit 5 test method for each uncovered branch

## Uncovered Branch Types

| Branch Type | Java Pattern | Common Fix |
|-------------|-------------|-----------|
| Condition false branch | `if (x > 0)` | Add test with `x <= 0` |
| Exception catch | `catch (DataAccessException e)` | Mock repository to throw exception |
| Optional empty | `Optional.empty()` path | Test not-found scenario |
| Null check | `if (item != null)` | Test with null input |
| Switch default | `default:` case | Test with unhandled enum value |

## Maven Configuration Reference

```xml
<plugin>
  <groupId>org.jacoco</groupId>
  <artifactId>jacoco-maven-plugin</artifactId>
  <configuration>
    <excludes>
      <exclude>**/*MapperImpl.class</exclude>
      <exclude>**/*Config.class</exclude>
      <exclude>**/*Application.class</exclude>
    </excludes>
    <rules>
      <rule>
        <element>BUNDLE</element>
        <limits>
          <limit><counter>LINE</counter><value>COVEREDRATIO</value><minimum>0.80</minimum></limit>
          <limit><counter>BRANCH</counter><value>COVEREDRATIO</value><minimum>0.70</minimum></limit>
        </limits>
      </rule>
    </rules>
  </configuration>
</plugin>
```
