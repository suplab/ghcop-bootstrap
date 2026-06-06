---
applyTo: "**/.github/workflows/**, **/Jenkinsfile, **/Jenkinsfile*, **/jenkins/**/*.groovy, **/pipeline/**"
description: "CI/CD pipeline standards: GitHub Actions workflow structure, Jenkins declarative pipeline patterns, quality gate requirements, and deployment job conventions."
---

## Context

This instruction file applies to GitHub Actions workflow files (`.github/workflows/*.yml`) and Jenkins pipeline files (`Jenkinsfile`). CI/CD pipelines are production code — they must be reviewed, version-controlled, and must not bypass quality gates.

---

## Pipeline Quality Gate Requirements

Every pipeline must enforce the following gates in order:

1. **Compile** — build passes with zero errors
2. **Unit tests** — all tests pass; no flaky tests allowed in `main`
3. **Coverage** — JaCoCo 80% line / 70% branch (business logic); Angular Istanbul 80/70/80/80
4. **SAST** — SonarQube/SonarCloud quality gate: no new blockers or critical issues
5. **Dependency scan** — OWASP dependency-check: no unfixed CVSS ≥ 8.0 CVEs
6. **Container build** — Docker image builds successfully (on `main`/release branches only)
7. **Deploy** — only after all gates pass; requires approval for production

---

## GitHub Actions Standards

### Workflow File Naming

| Purpose | File Name |
|---------|-----------|
| PR build and test | `ci.yml` |
| Main branch build + deploy to dev | `cd-dev.yml` |
| Release deploy to staging/prod | `cd-release.yml` |
| Scheduled scans (weekly) | `scheduled-scan.yml` |
| Dependency updates | `dependabot.yml` |

### Required Elements

```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main, develop]

jobs:
  build:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      id-token: write   # Required for OIDC-based AWS auth
      security-events: write  # Required for SARIF upload (SAST)
```

### AWS Authentication (OIDC — Required)

Never use long-lived AWS access keys in GitHub Actions. Use OIDC:

```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v4
  with:
    role-to-assume: ${{ secrets.AWS_DEPLOY_ROLE_ARN }}
    aws-region: eu-west-1
```

### Caching

```yaml
- uses: actions/cache@v4
  with:
    path: ~/.m2/repository
    key: ${{ runner.os }}-maven-${{ hashFiles('**/pom.xml') }}
    restore-keys: |
      ${{ runner.os }}-maven-
```

### Artefact Retention

```yaml
- uses: actions/upload-artifact@v4
  if: always()   # Upload even on failure for debugging
  with:
    name: test-reports
    path: target/surefire-reports/
    retention-days: 7
```

---

## Jenkins Declarative Pipeline Standards

```groovy
pipeline {
  agent { label 'java21' }

  options {
    timeout(time: 30, unit: 'MINUTES')
    buildDiscarder(logRotator(numToKeepStr: '10'))
    disableConcurrentBuilds()
  }

  stages {
    stage('Build & Test') {
      parallel {
        stage('Maven Build') {
          steps { sh 'mvn clean verify -B -T 2' }
        }
        stage('OWASP Scan') {
          steps { sh 'mvn dependency-check:check -B' }
        }
      }
    }
    stage('SonarQube') {
      steps {
        withSonarQubeEnv('SonarQube') {
          sh 'mvn sonar:sonar -B'
        }
        timeout(time: 5, unit: 'MINUTES') {
          waitForQualityGate abortPipeline: true
        }
      }
    }
    stage('Docker Build') {
      when { branch 'main' }
      steps { sh 'docker build -t ${IMAGE_TAG} .' }
    }
    stage('Deploy Dev') {
      when { branch 'main' }
      steps {
        sh 'npx cdk deploy AppStack --context env=dev --require-approval never'
      }
    }
    stage('Deploy Prod') {
      when { branch 'release/*' }
      input { message 'Deploy to production?' }
      steps {
        sh 'npx cdk deploy AppStack --context env=prod --require-approval never'
      }
    }
  }

  post {
    always {
      junit 'target/surefire-reports/*.xml'
      publishHTML target: [reportDir: 'target/site/jacoco', reportFiles: 'index.html', reportName: 'JaCoCo']
    }
    failure {
      slackSend(channel: '#build-alerts', message: "Build FAILED: ${JOB_NAME} #${BUILD_NUMBER}")
    }
  }
}
```

---

## Anti-Patterns

- ❌ `--no-verify` or `--skip-tests` in CI pipeline commands
- ❌ Hardcoded AWS credentials as environment variables
- ❌ Production deployment without quality gate success
- ❌ Concurrent deploys to the same environment without a lock
- ❌ `continue-on-error: true` on security scan steps
