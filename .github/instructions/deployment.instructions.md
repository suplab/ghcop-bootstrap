---
applyTo: "**/deploy/**, **/scripts/deploy*, **/appspec.yml, **/appspec.yaml, **/scripts/smoke*, **/deployment/**"
description: "Deployment standards: blue/green and canary patterns, pre-deploy checklist, rollback procedures, AWS CDK/SAM deploy commands, and local Docker Compose startup."
---

## Context

This instruction file applies to deployment scripts, appspec files, and deployment configuration. All deployments must be observable, reversible, and gated by a pre-deploy checklist. "Works in dev" is not a deployment strategy.

---

## Deployment Patterns

### Blue/Green Deployment (Production Default)

- Route 100% traffic to Green (current) while Blue (new) is provisioned
- Smoke test Blue fully before any traffic shift
- Shift 100% traffic to Blue atomically
- Keep Green available for 30 minutes for instant rollback
- AWS services: CodeDeploy `AllAtOnce` on ECS with `BlueGreenDeploymentConfig`

### Canary Deployment (Progressive Rollout)

- Shift traffic incrementally: 10% → 25% → 50% → 100%
- Monitor error rate and latency at each stage before proceeding
- Automatic rollback if error rate exceeds threshold at any stage
- AWS services: CodeDeploy Linear/Canary deployment configuration

### Rolling Update (Non-Production)

- Replace instances/tasks one at a time while maintaining minimum healthy percent
- ECS: `minimumHealthyPercent: 100`, `maximumPercent: 200`

---

## Pre-Deploy Checklist

This checklist must be completed before any production deployment:

- [ ] `cdk diff` / `terraform plan` reviewed and approved by two engineers
- [ ] Database migration scripts are idempotent and tested in staging
- [ ] Secrets exist in Secrets Manager for target environment
- [ ] CloudWatch alarms are armed and baseline metrics recorded
- [ ] Rollback procedure tested and documented in runbook
- [ ] On-call engineer notified and standing by
- [ ] Deployment window agreed (prefer off-peak hours for P1 risk changes)
- [ ] Customer communication drafted (if user-visible change)
- [ ] Post-deploy smoke test script prepared

---

## AWS CDK Deploy Commands

```bash
# Mandatory: review changes before deploy
npx cdk diff \
  --profile ${AWS_PROFILE} \
  --context env=${ENV}

# Deploy a specific stack
npx cdk deploy ${STACK_NAME} \
  --profile ${AWS_PROFILE} \
  --context env=${ENV} \
  --require-approval never \
  --outputs-file cdk-outputs.json

# Destroy (non-production only — CAUTION)
npx cdk destroy ${STACK_NAME} \
  --profile ${AWS_PROFILE} \
  --context env=${ENV} \
  --force
```

## SAM Deploy Commands

```bash
sam build

sam deploy \
  --stack-name ${STACK_NAME} \
  --s3-bucket ${DEPLOY_BUCKET} \
  --parameter-overrides Env=${ENV} \
  --capabilities CAPABILITY_IAM CAPABILITY_NAMED_IAM \
  --no-confirm-changeset \
  --region ${AWS_REGION}
```

---

## Smoke Test Template

```bash
#!/usr/bin/env bash
set -euo pipefail

BASE_URL="${BASE_URL:-http://localhost:8080}"
PASS=0; FAIL=0

check() {
  local name=$1; local cmd=$2
  if eval "$cmd" > /dev/null 2>&1; then
    echo "✅ PASS: $name"; ((PASS++))
  else
    echo "❌ FAIL: $name"; ((FAIL++))
  fi
}

check "Health endpoint UP"      "curl -sf ${BASE_URL}/actuator/health | grep -q '\"status\":\"UP\"'"
check "API v1 ping"             "curl -sf ${BASE_URL}/api/v1/ping | grep -q 'pong'"
check "No 500s in CloudWatch"   "aws cloudwatch get-metric-statistics --namespace AWS/ApplicationELB \
  --metric-name HTTPCode_Target_5XX_Count --statistics Sum --period 300 \
  --start-time $(date -u -v-5M +%FT%TZ) --end-time $(date -u +%FT%TZ) \
  --dimensions Name=LoadBalancer,Value=${ALB_NAME} | jq '.Datapoints[0].Sum // 0 | . == 0'"

echo ""
echo "Results: ${PASS} passed, ${FAIL} failed"
[[ $FAIL -eq 0 ]] || exit 1
```

---

## Rollback Runbook Template

```markdown
## Rollback Runbook: {Service Name}

### Trigger Conditions
Initiate rollback if any of the following occur within 10 minutes of deployment:
- Error rate > 1% (production baseline)
- p99 latency > 2x baseline
- Any smoke test failure

### Rollback Steps

#### ECS (CodeDeploy)
```bash
# Identify the failed deployment
aws deploy list-deployments --application-name ${APP_NAME} --deployment-group-name ${GROUP_NAME}

# Rollback to previous task definition
aws ecs update-service \
  --cluster ${CLUSTER} \
  --service ${SERVICE} \
  --task-definition ${PREVIOUS_TASK_DEF_ARN} \
  --force-new-deployment
```

#### CDK Stack
```bash
# Redeploy previous version from git tag
git checkout ${PREVIOUS_TAG}
npx cdk deploy ${STACK_NAME} --context env=prod --require-approval never
```

### Verification
- Run smoke tests against rollback deployment
- Confirm error rate returns to baseline in CloudWatch
- Post rollback completion in #incidents channel
```

---

## Anti-Patterns

- ❌ Deploying directly to production without staging validation
- ❌ Skipping `cdk diff` / `terraform plan` before apply
- ❌ Deploying during peak business hours without business sign-off
- ❌ No rollback plan for a production deployment
- ❌ Long-lived feature flags instead of clean deployments
