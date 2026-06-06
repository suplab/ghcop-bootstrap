---
applyTo: "**/*.ipynb, **/sagemaker/**, **/bedrock/**, **/glue/**, **/etl/**, **/ml/**, **/ai/**, **/data-science/**, **/notebooks/**"
description: "AWS data, ML, and AI platform standards: SageMaker, Bedrock, Glue ETL, Athena, feature store, RAG pipelines, and responsible AI controls."
---

## Context

This instruction file applies to data science notebooks, ML pipeline code, Glue ETL jobs, and AI/LLM application code on AWS. The AWS data platform follows a Bronze/Silver/Gold lakehouse pattern. All ML models must be version-controlled in SageMaker Model Registry. All LLM integrations must implement output guardrails.

---

## Data Lake Architecture (Bronze / Silver / Gold)

| Layer | S3 Prefix | Format | Purpose | Retention |
|-------|-----------|--------|---------|----------|
| **Bronze** | `s3://{bucket}/bronze/{source}/{date}/` | Raw (JSON, CSV, Parquet) | Immutable raw ingestion | 7 years |
| **Silver** | `s3://{bucket}/silver/{domain}/{date}/` | Parquet, partitioned | Cleaned, validated, normalised | 3 years |
| **Gold** | `s3://{bucket}/gold/{product}/{date}/` | Parquet, aggregated | Business-ready analytics | 1 year |

- **Never overwrite Bronze** — Bronze is immutable; always append
- **Partition by date** (`year=YYYY/month=MM/day=DD`) for time-series data
- **Register all tables** in Glue Data Catalog with schemas
- **Apply Lake Formation permissions** — no direct S3 bucket policies for data access

---

## Glue ETL Standards

```python
import sys
from awsglue.context import GlueContext
from awsglue.utils import getResolvedOptions
from pyspark.context import SparkContext

args = getResolvedOptions(sys.argv, ['JOB_NAME', 'source_database', 'target_bucket', 'target_prefix'])
sc = SparkContext()
glueContext = GlueContext(sc)
logger = glueContext.get_logger()

# CORRECT: Parameterised, no hardcoded paths
source_database = args['source_database']
target_path = f"s3://{args['target_bucket']}/{args['target_prefix']}"
```

- **Parameterise all jobs** — no hardcoded bucket names, table names, or dates
- **Log record counts** before and after transformations for data quality auditing
- **Use DynamicFrame → DataFrame → DynamicFrame** pattern for complex transformations
- **Handle bad records** explicitly — use `ResolveChoice` and rejection sinks

---

## SageMaker Standards

### Notebooks (SageMaker Studio)

- **Structure notebooks** as: Imports → Data Loading → EDA → Feature Engineering → Model/Export
- **Parameterise cell inputs** using Papermill parameters for automated execution
- **Register features** in SageMaker Feature Store for reuse across models
- **Store all artefacts in S3** — never rely on notebook instance ephemeral storage

### Training Jobs

```python
from sagemaker.estimator import Estimator

estimator = Estimator(
    image_uri=training_image_uri,
    role=sagemaker_role,
    instance_type='ml.m5.xlarge',
    instance_count=1,
    use_spot_instances=True,   # Up to 90% cost saving
    max_wait=7200,
    output_path=f's3://{bucket}/models/',
    hyperparameters={
        'epochs': 10,
        'learning-rate': 0.001,
    },
)
```

- **Use Spot Instances** for training jobs (non-time-critical) — up to 90% cost reduction
- **Enable SageMaker Experiments** tracking on all training jobs
- **Version all models** in Model Registry with: training data URI, metrics, approval status

### Model Deployment

- **`PendingManualApproval`** for model registry entries — no automatic prod promotion
- **Enable Model Monitor** on all production endpoints for data drift and model quality
- **Set auto-scaling** — never fixed-instance production endpoints
- **A/B test** new model versions with production variant weights before full cutover

---

## Amazon Bedrock / LLM Standards

### Model Invocation

```python
import boto3, json

bedrock = boto3.client('bedrock-runtime', region_name='eu-west-1')

def invoke_claude(messages: list, max_tokens: int = 1000, system: str = "") -> str:
    body = {
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": max_tokens,
        "system": system,
        "messages": messages,
    }
    response = bedrock.invoke_model(
        modelId="anthropic.claude-3-5-sonnet-20241022-v2:0",
        body=json.dumps(body),
    )
    return json.loads(response['body'].read())['content'][0]['text']
```

### Safety Non-Negotiables

- **Never log full prompts** containing user PII — log token count, latency, model ID only
- **Always implement Bedrock Guardrails** for customer-facing features
- **Always set `max_tokens`** — unbounded generation is a cost and safety risk
- **Always validate structured outputs** against JSON schema before downstream use
- **Never pass user input directly into system prompts** — sanitise to prevent prompt injection

### RAG Pipeline Checklist

- [ ] Chunking strategy documented (chunk size, overlap, splitting method)
- [ ] Embedding model version pinned (re-index if model changes)
- [ ] Retrieval evaluated with RAGAS: faithfulness ≥ 0.7, answer relevance ≥ 0.7
- [ ] Guardrails configured for topic blocking and PII filtering
- [ ] All LLM calls logged to CloudWatch with token count and latency

---

## Responsible AI Controls

| Control | Implementation |
|---------|---------------|
| Bias monitoring | SageMaker Clarify on training data and model predictions |
| Explainability | SHAP values via SageMaker Clarify |
| Hallucination detection | Groundedness check via Bedrock Guardrails |
| PII handling | Amazon Comprehend PII detection before LLM processing |
| Audit trail | All model invocations logged to CloudWatch + S3 |
