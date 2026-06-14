# AI Governance Standard

> Mandatory governance requirements for all AI/ML systems deployed to production. Enforced by `ai-governance-officer` agent and the ARB sign-off gate.

---

## Scope

This standard applies to any system that:
- Uses a machine learning model to make or inform a decision that affects users or business outcomes
- Integrates a generative AI model (LLM, image generation, code generation) in a user-facing or automated workflow
- Exposes AI-generated content without human review

---

## Risk Classification

Classify every AI system before deployment. Classification determines the mandatory controls.

| Risk Tier | Definition | Examples | ARB Required |
|---|---|---|---|
| **Tier 1 — Critical** | Automated decision with legal or financial consequence, no human override | Credit scoring, fraud block, benefits eligibility | Yes — full ARB review |
| **Tier 2 — High** | Significant influence on decision; human reviews but rarely overrides | Loan pre-approval recommendation, claims triage | Yes — ARB checklist |
| **Tier 3 — Medium** | Assistive output; human always reviews before acting | Code suggestion, document summarisation, search ranking | Risk register entry |
| **Tier 4 — Low** | No consequential decision; decorative or informational | Chatbot FAQ, image labelling for internal tools | Self-certification |

---

## Mandatory Artefacts per Tier

### All Tiers — Model Card

Every model deployed to production must have a Model Card at `docs/ai/model-cards/<model-name>.md` containing:

- **Model identity:** name, version, type (classification / regression / generative / embedding)
- **Intended use:** primary use cases and explicitly out-of-scope uses
- **Training data:** sources, date range, preprocessing applied, known biases
- **Performance metrics:** accuracy / F1 / AUC on hold-out set, broken down by demographic segment where applicable
- **Limitations:** known failure modes, edge cases, distribution shift sensitivity
- **Ethical considerations:** protected attribute handling, potential for disparate impact
- **Owner:** team responsible for monitoring and retraining
- **Review date:** maximum 6 months from last update

### Tier 1 & 2 — AI Risk Assessment

Complete `docs/ai/risk-assessments/<model-name>-risk.md` before first deployment:

- STRIDE threat model for the AI system (prompt injection, data poisoning, model extraction, adversarial inputs)
- Fairness analysis: measure and document model performance across protected groups (gender, age, ethnicity where legally permitted)
- Data lineage: traceable path from raw data to training dataset to deployed model
- Human override mechanism: documented process for a human to override or escalate any model decision
- Incident response: who to contact, how to roll back, SLA for response

### Tier 1 — Additional Controls

- Explainability: model must produce a human-readable reason for every decision (SHAP values, LIME, or rule extraction)
- Consent and disclosure: users must be informed that an AI system influences decisions that affect them
- Audit log: every model inference that affects a user decision must be logged with input features, output, confidence score, and timestamp (retained ≥ 7 years for financial decisions)

---

## EU AI Act Alignment

For systems deployed within or affecting EU users:

| AI Act Category | Requirement | Applies When |
|---|---|---|
| **Prohibited** | Must not deploy | Real-time biometric surveillance, social scoring, subliminal manipulation |
| **High-Risk (Annex III)** | Full conformity assessment, technical documentation, human oversight | Credit, employment, education, law enforcement, critical infrastructure |
| **Limited Risk** | Transparency disclosure to users | Chatbots, deepfake generation |
| **Minimal Risk** | No mandatory requirement | Spam filters, AI games |

Confirm classification with legal before deployment of Tier 1 or Tier 2 systems to EU customers.

---

## LLM-Specific Controls

### Prompt Injection Prevention

- Never concatenate user input directly into system prompts without sanitisation
- Validate LLM output before using it in downstream systems (do not trust LLM to produce valid JSON, SQL, or code without parsing and validation)
- Apply output length limits and format validation at the API boundary

```python
# CORRECT — validate LLM output before use
import json
from pydantic import BaseModel, ValidationError

class LLMExtractedOrder(BaseModel):
    customer_id: str
    items: list[str]

raw = llm.complete(prompt)
try:
    parsed = LLMExtractedOrder.model_validate_json(raw)
except (json.JSONDecodeError, ValidationError) as exc:
    logger.warning("LLM output validation failed: %s", exc)
    raise InvalidLLMOutputError from exc

# WRONG — use LLM output directly without validation
order_data = json.loads(llm.complete(prompt))  # no schema validation
```

### Data Minimisation

- Never send PII to external LLM APIs (OpenAI, Anthropic) without explicit user consent and data processing agreement
- For AWS Bedrock: data stays within the AWS account by default — confirm model provider's data handling policy
- Anonymise or pseudonymise inputs before sending to hosted LLM services

### Model Versioning and Pinning

- Always pin the exact model version in production (e.g., `anthropic.claude-sonnet-4-6` not `claude-latest`)
- Test new model versions in staging before promoting to production
- Document version changes in the Model Card with performance comparison

---

## Monitoring and Drift Detection

- **Prediction drift:** alert when output distribution shifts by >10% from baseline (30-day rolling window)
- **Input drift:** alert when input feature distributions shift significantly (PSI > 0.2)
- **Performance degradation:** alert when accuracy/F1 drops >5% from baseline on labelled production samples
- **LLM quality:** sample 5% of LLM outputs for human evaluation monthly; track quality score trend

Review cadence:
- Monthly: drift metrics review
- Quarterly: model card update, fairness re-evaluation
- Annually: full re-training assessment

---

## Retraining Policy

- Define retraining trigger conditions in the Model Card before initial deployment
- Retrain whenever: drift alert fires, performance drops below threshold, training data is updated with >20% new records
- Never retrain in production — use a staging environment with the same data pipeline
- Require A/B test approval before replacing a Tier 1/2 production model

---

## Anti-Patterns

| Anti-Pattern | Correct Alternative |
|---|---|
| Deploy without Model Card | Complete Model Card before any production deployment |
| Use `latest` model alias in production | Pin exact model version string |
| Concatenate user input into system prompts | Sanitise and structure input; use separate message roles |
| Send PII to external LLM APIs | Anonymise or use a self-hosted / VPC-deployed model |
| No human override for Tier 1 decisions | Mandatory human review queue for high-confidence-low-explanation decisions |
| Monitor only technical metrics (latency, error rate) | Also monitor prediction quality, fairness, and drift |
| Undocumented training data | Full data lineage from raw source to training dataset |
