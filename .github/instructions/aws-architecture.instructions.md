---
applyTo: "**/*.ts, **/cdk.json, **/*.tf, **/*.tfvars, **/template.yaml, **/samconfig.toml, **/cloudformation/**/*.yaml, **/cdk/**/*.ts"
description: "AWS architecture standards: Well-Architected Framework, CDK TypeScript patterns, Terraform HCL conventions, IAM least-privilege, and tagging strategy."
---

## Context

This instruction file applies to AWS infrastructure code: CDK TypeScript stacks, Terraform HCL modules, CloudFormation templates, and SAM application templates. All AWS infrastructure must be designed and reviewed against the AWS Well-Architected Framework six pillars.

---

## Well-Architected Non-Negotiables

### Operational Excellence
- All resources must emit structured logs to CloudWatch Logs
- All stateful changes must produce CloudTrail audit events
- Runbooks must exist for all P1/P2 failure modes before go-live

### Security
- **No hardcoded credentials** in any IaC file — use Secrets Manager, Parameter Store, or IAM roles
- **No `*` in IAM policies** without SCPs as compensating controls and documented justification
- All data at rest encrypted: S3 SSE-KMS (or SSE-S3 minimum), RDS KMS, EBS KMS
- All data in transit over TLS 1.2+; no HTTP endpoints for sensitive data
- VPC endpoints for S3, ECR, SSM, Secrets Manager to avoid NAT Gateway costs and internet exposure
- Enable GuardDuty, Security Hub, and CloudTrail in all accounts

### Reliability
- **Multi-AZ for all production workloads** — minimum 2 AZs, prefer 3
- RDS Multi-AZ enabled in production; Aurora Global Database for RPO < 1 min
- ECS/EKS workloads must have `minHealthyPercent: 100` during deployments
- SQS DLQ configured for all consumer-facing queues
- Circuit breakers on all external service calls

### Performance Efficiency
- Right-size before deploying: use Graviton (ARM) instances where workload is compatible
- CloudFront in front of S3 static assets and API Gateway for edge caching
- ElastiCache (Redis) for session state and expensive read-only lookups
- Auto-scaling configured with warm-up periods; never fixed-capacity in production

### Cost Optimisation
- Use Graviton instances (e.g., `t4g`, `m7g`, `r7g`) — ~20% cheaper than x86 equivalents
- S3 Intelligent-Tiering for infrequently accessed objects > 128 KB
- Reserved Instances or Savings Plans for stable baseline workloads
- NAT Gateway: one per AZ max; use VPC endpoints where possible
- Lambda: set memory correctly — over-allocated Lambda is wasted spend

### Sustainability
- Consolidate workloads to reduce idle capacity
- Prefer serverless (Lambda, Fargate, Aurora Serverless) where workload is bursty

---

## CDK TypeScript Standards

- **Use L2/L3 constructs** — prefer CDK patterns libraries over raw L1 CloudFormation resources
- **Stack props typed via interface** — no `any` types in CDK code
- **Environment-aware stacks** — read `env` from CDK context, never hardcode account/region
- **`RemovalPolicy.RETAIN`** for stateful resources (RDS, DynamoDB, S3) in production environments
- **CDK Aspects for tagging** — apply mandatory tags via `Aspects.of(app).add(new TaggingAspect(tags))`
- **Secrets via `SecretValue.ssmSecure()` or `Secret.fromSecretNameV2()`** — never `SecretValue.unsafePlainText()`

```typescript
// CORRECT: Environment from context
const env = app.node.tryGetContext('env') ?? 'dev';
const isProd = env === 'prod';

// CORRECT: Conditional removal policy
new s3.Bucket(this, 'DataBucket', {
  removalPolicy: isProd ? cdk.RemovalPolicy.RETAIN : cdk.RemovalPolicy.DESTROY,
  autoDeleteObjects: !isProd,
});
```

---

## Terraform Standards

- **Remote state in S3 + DynamoDB locking** — no local state files committed to git
- **Modular structure**: `main.tf`, `variables.tf`, `outputs.tf`, `versions.tf` per module
- **`versions.tf` pins provider versions** with `~>` constraint — no unpinned providers
- **`default_tags` on AWS provider** for consistent resource tagging
- **Workspaces or directory-per-env** for environment isolation (prefer directory-per-env for production separation)

```hcl
# CORRECT: Provider with default tags
provider "aws" {
  region = var.aws_region
  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
      Owner       = var.team_name
    }
  }
}
```

---

## Mandatory Tagging Strategy

All AWS resources must carry:

| Tag Key | Example Value | Required |
|---------|-------------|---------|
| `Project` | `smart-retail` | Yes |
| `Environment` | `prod` / `staging` / `dev` | Yes |
| `Owner` | `platform-team` | Yes |
| `CostCentre` | `CC-1234` | Yes |
| `ManagedBy` | `CDK` / `Terraform` | Yes |

---

## IAM Least Privilege Checklist

- [ ] No `*` actions without documented justification
- [ ] No `*` resources; use specific ARNs with `${AWS::AccountId}` / `${AWS::Region}`
- [ ] Conditions used where applicable: `aws:RequestedRegion`, `aws:SourceAccount`
- [ ] Task roles for ECS/Lambda have only the permissions needed for that function
- [ ] Cross-account trust uses `aws:PrincipalOrgID` condition, not account IDs
