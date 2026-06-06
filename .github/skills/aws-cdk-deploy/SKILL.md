---
name: aws-cdk-deploy
description: 'AWS CDK deployment commands, pre-deploy checklist, stack diff interpretation, and rollback procedures. Use when deploying CDK stacks to AWS environments.'
---

# AWS CDK Deploy Skill

Standard commands and procedures for AWS CDK deployments.

## Bootstrap (One-Time per Account/Region)

```bash
npx cdk bootstrap \
  --profile ${AWS_PROFILE} \
  aws://${AWS_ACCOUNT_ID}/${AWS_REGION}
```

## Standard Deploy Sequence

```bash
# 1. Always diff first
npx cdk diff \
  --profile ${AWS_PROFILE} \
  --context env=${ENV}

# 2. Deploy (after diff approval)
npx cdk deploy ${STACK_NAME} \
  --profile ${AWS_PROFILE} \
  --context env=${ENV} \
  --require-approval never \
  --outputs-file cdk-outputs.json

# 3. Verify outputs
cat cdk-outputs.json
```

## Pre-Deploy Checklist

- [ ] `cdk diff` reviewed by two engineers
- [ ] Secrets exist in Secrets Manager for `${ENV}`
- [ ] DB migration tested in staging
- [ ] CloudWatch baseline recorded
- [ ] Rollback procedure known

## Common CDK Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| `UPDATE_ROLLBACK_FAILED` | Stuck stack | `cdk doctor` + manual CloudFormation rollback |
| `BOOTSTRAP_VERSION_NOT_FOUND` | Old bootstrap | Run `cdk bootstrap` again |
| `Resource already exists` | Orphaned resource from prior failed deploy | Import resource: `cdk import` |
| `Exported value X cannot be deleted` | Cross-stack export in use | Remove the import in consuming stack first |

## Rollback

```bash
# Redeploy previous version
git checkout ${PREVIOUS_TAG}
npx cdk deploy ${STACK_NAME} \
  --profile ${AWS_PROFILE} \
  --context env=prod \
  --require-approval never
```
