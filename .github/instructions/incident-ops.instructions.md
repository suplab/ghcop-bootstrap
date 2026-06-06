---
applyTo: "**/runbooks/**, **/oncall/**, **/incidents/**, **/ops/**, **/sre/**"
description: "Operational standards: incident severity classification, runbook format, CloudWatch alarm patterns, on-call responsibilities, and post-incident review process."
---

## Context

This instruction file applies to operational documentation: runbooks, incident reports, on-call playbooks, and observability configuration. Operational documents are production artefacts — they must be tested, reviewed, and maintained with the same rigor as code.

---

## Incident Severity Matrix

| Priority | Definition | Response SLA | Escalation SLA |
|----------|-----------|-------------|---------------|
| **P1** | Complete service outage, data loss risk, or security breach | 15 min initial response | Immediate: Engineering VP, CTO |
| **P2** | Significant service degradation, > 10% users impacted, or revenue impact | 30 min initial response | 1 hour if no progress |
| **P3** | Partial degradation, < 10% users, workaround available | 4 hours | Business hours only |
| **P4** | Minor issue, single user, cosmetic | Next business day | — |

---

## Runbook Format (Required)

```markdown
# Runbook: {Descriptive Title}

## Overview
**System:** {Service name}
**Severity:** P1 / P2 / P3
**Last Reviewed:** {date}
**Owner:** {Team name}

## Symptoms
- {Observable symptom 1 — what the user or monitoring sees}
- {Observable symptom 2}

## Diagnosis Steps

1. **Check CloudWatch dashboard**
   - Navigate to: [{Dashboard Name}]({CloudWatch URL})
   - Look for: {specific metric anomaly}

2. **Check application logs**
   ```
   filter @message like /ERROR/
   | stats count(*) by bin(5m)
   ```

3. **Check database health**
   - RDS Console → {instance} → Monitoring
   - Alert if: CPU > 85%, free storage < 20%, connections > 80% of max

## Resolution Steps

### Scenario A: {Most common cause}
1. {Specific action}
2. {Verify resolution: what metric or log confirms it's fixed}

### Scenario B: {Second most common cause}
1. {Specific action}

## Rollback Procedure
1. {Step to revert the last deployment}
2. {Verification step}

## Escalation Path
If not resolved within {N} minutes:
- {Name/Role} — {contact method}
- {Name/Role} — {contact method}

## Related Resources
- [Architecture Diagram]({link})
- [Previous Incident Reports]({link})
```

---

## CloudWatch Alarm Conventions

### Alarm Naming

`{Project}-{Environment}-{Service}-{Metric}-{Severity}`

Example: `smart-retail-prod-order-api-5xx-rate-p1`

### Standard Alarm Thresholds

| Metric | Namespace | Warning | Critical | Period |
|--------|----------|---------|----------|--------|
| ALB 5xx error rate | AWS/ApplicationELB | > 1% | > 5% | 5 min |
| ECS CPU utilisation | AWS/ECS | > 70% | > 85% | 5 min |
| ECS Memory utilisation | AWS/ECS | > 75% | > 90% | 5 min |
| RDS CPU | AWS/RDS | > 70% | > 85% | 5 min |
| Lambda error rate | AWS/Lambda | > 1% | > 5% | 5 min |
| SQS DLQ messages | AWS/SQS | > 0 | > 10 | 1 min |
| API Gateway 4xx rate | AWS/ApiGateway | > 5% | > 20% | 5 min |

### CloudWatch Logs Insights — Standard Queries

```sql
-- Error rate over time
filter @message like /ERROR/
| stats count(*) as errorCount by bin(5m)
| sort @timestamp desc

-- Slow requests (> 2s)
filter @message like /took/
| parse @message "took *ms" as duration
| filter duration > 2000
| stats count(*), avg(duration) as avgDuration by bin(5m)

-- Exception summary
filter @message like /Exception/
| parse @message "* Exception" as exceptionType
| stats count(*) as frequency by exceptionType
| sort frequency desc
```

---

## Post-Incident Review Process

| Step | Owner | Timeline |
|------|-------|----------|
| Draft RCA document | On-call engineer | Within 24h of resolution |
| Circulate draft | Incident Commander | Within 48h |
| Review meeting | All responders | Within 72h |
| Corrective actions assigned | Tech Lead / Team Lead | At review meeting |
| Corrective actions tracked | Project Tracker | Sprint backlog |
| RCA published | Engineering Manager | Within 1 week |

Post-incident reviews are blameless. The goal is system improvement, not accountability assignment.

---

## On-Call Responsibilities

- Acknowledge PagerDuty alerts within SLA (P1: 15 min, P2: 30 min)
- Post incident declaration in Slack `#incidents` channel within 5 minutes of declaring P1/P2
- Update incident status every 30 minutes for P1, every 60 minutes for P2
- Hand off cleanly at shift end: summarise active incidents, investigations, and outstanding actions
- Complete runbook after resolving an incident if the runbook was missing or incorrect
