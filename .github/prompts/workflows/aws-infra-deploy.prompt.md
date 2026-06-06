---
mode: "agent"
description: "AWS infrastructure delivery workflow: Well-Architected review → CDK stack generation → CI/CD pipeline → deploy → smoke test"
tools: ['read', 'edit', 'search', 'execute', 'runCommands']
---

## AWS Infrastructure Delivery Workflow

This workflow guides the full delivery of a new AWS infrastructure component — from architecture review through CDK implementation to deployment and verification.

---

## Step 1 — AWS Architect: Well-Architected Review

**Agent:** `@aws-architect`

Produce a Well-Architected review for the proposed infrastructure:

1. Review the infrastructure requirement against all six pillars (Operational Excellence, Security, Reliability, Performance Efficiency, Cost Optimisation, Sustainability)
2. Identify the AWS services to be used and their configurations
3. Produce a cost estimate (monthly, at expected load)
4. Flag any Well-Architected risks with recommended mitigations
5. Produce a CDK stack skeleton (TypeScript) with construct placeholders

**Output required before Step 2:**
- [ ] Service topology diagram (or Mermaid sequence diagram)
- [ ] CDK stack skeleton with `// TODO` placeholders
- [ ] Cost estimate table
- [ ] Well-Architected findings table

---

## Step 2 — CDK/Terraform Helper: IaC Implementation

**Agent:** `@cdk-terraform-helper`

Using the skeleton from Step 1, implement the complete CDK stack:

1. Replace all `// TODO` placeholders with complete construct code
2. Apply mandatory tagging via CDK Aspect
3. Configure auto-scaling policies
4. Add IAM roles and policies following least-privilege
5. Add CloudWatch alarms (as code)

**Standards:** See `.github/instructions/cdk-terraform.instructions.md` and `.github/instructions/aws-architecture.instructions.md`

**Output required before Step 3:**
- [ ] Complete CDK stack TypeScript file(s)
- [ ] `cdk.json` context variables
- [ ] IAM policy documents
- [ ] CDK unit tests (`@aws-cdk/assertions`)

---

## Step 3 — CI Engineer: Pipeline Configuration

**Agent:** `@ci-engineer`

Create or update the CI/CD pipeline to include the new infrastructure:

1. Add CDK diff step to PR pipeline (informational)
2. Add CDK deploy to dev on merge to main
3. Add CDK deploy to staging with approval gate
4. Add CDK deploy to production with approval gate + rollback configuration

**Output required before Step 4:**
- [ ] GitHub Actions workflow file (or Jenkinsfile update)
- [ ] OIDC IAM role configuration
- [ ] Branch protection rules documentation

---

## Step 4 — AWS Deploy Helper: Deployment Execution

**Agent:** `@aws-deploy-helper`

Execute the deployment:

1. Run `cdk diff` and confirm output with the team
2. Complete the pre-deploy checklist
3. Execute `cdk deploy` with correct profile and context
4. Monitor CloudWatch for the first 15 minutes post-deploy
5. Execute smoke tests

**Standards:** See `.github/instructions/deployment.instructions.md`

**Output required before Step 5:**
- [ ] `cdk diff` output reviewed
- [ ] Pre-deploy checklist completed
- [ ] `cdk-outputs.json` saved
- [ ] CloudWatch baseline confirmed

---

## Step 5 — Ops Engineer: Observability Setup

**Agent:** `@ops-engineer`

Ensure the deployed infrastructure is fully observable:

1. Create CloudWatch dashboard with RED metrics (Rate, Errors, Duration)
2. Configure standard alarms (CPU, memory, error rate, queue depth)
3. Create CloudWatch Logs Insights saved queries
4. Document in runbook: symptoms, diagnosis steps, resolution, escalation

**Output — Workflow Complete:**
- [ ] CloudWatch dashboard created
- [ ] Alarms configured and tested (send a test notification)
- [ ] Runbook committed to `docs/runbooks/`
- [ ] Architecture diagram updated
