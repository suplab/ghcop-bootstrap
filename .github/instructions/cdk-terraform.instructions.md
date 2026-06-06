---
applyTo: "**/cdk.json, **/cdk/**/*.ts, **/*.tf, **/*.tfvars, **/terraform/**/*.tf, **/infra/**/*.ts, **/infra/**/*.tf"
description: "IaC standards for AWS CDK (TypeScript) and Terraform HCL: module structure, remote state, least-privilege, naming conventions, and environment management."
---

## Context

This instruction file applies to all Infrastructure as Code: AWS CDK TypeScript stacks and Terraform HCL modules. IaC is production code and must meet the same quality bar: code review, testing, version control, and documentation.

---

## CDK TypeScript Standards

### Project Structure

```
infrastructure/
├── bin/
│   └── app.ts                  # CDK app entry point
├── lib/
│   ├── stacks/
│   │   ├── network-stack.ts    # VPC, subnets, security groups
│   │   ├── database-stack.ts   # RDS, ElastiCache
│   │   └── app-stack.ts        # ECS, ALB, IAM
│   └── constructs/
│       └── fargate-service.ts  # Reusable custom construct
├── test/
│   └── stacks/
│       └── app-stack.test.ts   # CDK assertions tests
├── cdk.json
└── tsconfig.json
```

### CDK Code Rules

- **L2/L3 constructs preferred** over L1 (`CfnXxx`) — less code, more defaults, fewer mistakes
- **Stack props typed** — every stack has a typed `interface XxxStackProps extends cdk.StackProps`
- **No hardcoded account/region** — use `this.account`, `this.region`, or context
- **Secrets via `SecretValue.ssmSecure()` or `Secret.fromSecretNameV2()`**
- **`RemovalPolicy.RETAIN`** for production databases, S3 buckets with data, DynamoDB tables
- **CDK Aspects** for organisation-wide cross-cutting concerns (tagging, compliance)
- **CDK Assertions** (`@aws-cdk/assertions`) for unit tests on stack templates

```typescript
// CORRECT: Environment-aware removal policy
const isProd = props.env === 'prod';
new dynamodb.Table(this, 'OrdersTable', {
  removalPolicy: isProd ? cdk.RemovalPolicy.RETAIN : cdk.RemovalPolicy.DESTROY,
});
```

### CDK Testing

CDK stacks must have unit tests using `@aws-cdk/assertions`:

```typescript
import { Template } from 'aws-cdk-lib/assertions';

test('ECS service has correct CPU and memory', () => {
  const template = Template.fromStack(stack);
  template.hasResourceProperties('AWS::ECS::TaskDefinition', {
    Cpu: '512',
    Memory: '1024',
  });
});
```

---

## Terraform Standards

### Module Structure

```
modules/
└── ecs-fargate-service/
    ├── main.tf           # Resources
    ├── variables.tf      # Input variables with descriptions and types
    ├── outputs.tf        # Output values
    ├── versions.tf       # Required providers and versions
    └── README.md         # Module documentation
```

### Remote State (Required)

```hcl
terraform {
  backend "s3" {
    bucket         = "{project}-terraform-state-{account_id}"
    key            = "{env}/{component}/terraform.tfstate"
    region         = "eu-west-1"
    encrypt        = true
    dynamodb_table = "{project}-terraform-locks"
  }
}
```

### Provider Configuration

```hcl
terraform {
  required_version = ">= 1.6.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = var.aws_region
  default_tags {
    tags = {
      Project     = var.project_name
      Environment = var.environment
      ManagedBy   = "Terraform"
      Owner       = var.team_name
      CostCentre  = var.cost_centre
    }
  }
}
```

### Variables

```hcl
variable "environment" {
  description = "Deployment environment: dev, staging, or prod"
  type        = string
  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

---

## Naming Conventions

| Resource | Convention | Example |
|----------|-----------|---------|
| S3 bucket | `{project}-{env}-{purpose}` | `smart-retail-prod-assets` |
| ECR repository | `{project}/{service}` | `smart-retail/order-service` |
| ECS cluster | `{project}-{env}` | `smart-retail-prod` |
| Lambda function | `{project}-{env}-{function}` | `smart-retail-prod-order-processor` |
| IAM role | `{project}-{env}-{service}-role` | `smart-retail-prod-order-service-role` |

---

## Anti-Patterns to Avoid

- ❌ Hardcoded account IDs, region names, or ARNs
- ❌ Secrets in plain text in variables or environment blocks
- ❌ `terraform apply` without `terraform plan` review
- ❌ Shared state file across environments
- ❌ `force_destroy = true` on production S3 buckets
- ❌ Unversioned provider dependencies (`version = "*"`)
