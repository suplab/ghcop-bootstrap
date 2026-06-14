# Runbook: [SERVICE NAME] — [SCENARIO NAME]

**Service:** `<service-name>`
**Severity:** P1 / P2 / P3
**Last reviewed:** YYYY-MM-DD
**Owner:** `<team-name>`
**Escalation:** `<pagerduty-rotation-name>`

---

## 1. Overview

Brief description of the scenario this runbook addresses: what alert fired, what symptom is visible to users, and what the likely cause is.

**Example alert that triggers this runbook:**
```
CRITICAL: order-service — Error Rate > 5% for 5 minutes
```

**User impact:** [e.g. "Customers cannot place orders via checkout"]

---

## 2. Quick Diagnosis (First 5 Minutes)

Run these commands immediately. Each check takes < 30 seconds.

### Check 1 — Service health

```bash
# ECS / Fargate
aws ecs describe-tasks --cluster prod-cluster --tasks $(aws ecs list-tasks --cluster prod-cluster --service-name order-service --query 'taskArns[*]' --output text) --query 'tasks[*].{id:taskArn,status:lastStatus,health:healthStatus}'

# Kubernetes
kubectl get pods -n order-service -l app=order-service
kubectl top pods -n order-service
```

Expected: `RUNNING` / `HEALTHY`. If tasks are `STOPPED` or pods are `CrashLoopBackOff` → go to Section 4 (Restart Procedure).

### Check 2 — Error rate in logs

```bash
# CloudWatch Logs Insights
aws logs start-query \
  --log-group-name /ecs/order-service \
  --start-time $(date -d '10 minutes ago' +%s) \
  --end-time $(date +%s) \
  --query-string 'fields @timestamp, @message | filter @message like /ERROR/ | stats count() as error_count by bin(1m)'
```

### Check 3 — Database connectivity

```bash
# Test from a running task
aws ecs execute-command --cluster prod-cluster \
  --task <task-arn> \
  --container order-service \
  --command "pg_isready -h $DB_HOST -p 5432 -U $DB_USER" \
  --interactive
```

### Check 4 — Downstream dependency health

```bash
# Check each dependency (payment-service, inventory-service)
curl -s https://payment-service.internal/actuator/health | jq .status
curl -s https://inventory-service.internal/actuator/health | jq .status
```

---

## 3. Decision Tree

```
Service not responding?
├── Tasks/pods stopped or crash-looping → Section 4 (Restart)
├── Tasks running but high error rate
│   ├── DB errors in logs → Section 5 (Database)
│   ├── Downstream 5xx errors → Section 6 (Upstream/Dependency)
│   └── OOM errors → Section 7 (Memory)
└── Intermittent errors → Section 8 (Rollback)
```

---

## 4. Restart Procedure

**Prerequisites:** Confirm with on-call lead before restarting a P1 service.

```bash
# ECS — force new deployment (rolling restart)
aws ecs update-service \
  --cluster prod-cluster \
  --service order-service \
  --force-new-deployment

# Monitor until stable
aws ecs wait services-stable \
  --cluster prod-cluster \
  --services order-service
```

```bash
# Kubernetes — rolling restart
kubectl rollout restart deployment/order-service -n order-service
kubectl rollout status deployment/order-service -n order-service
```

**Success criteria:** Error rate drops to < 0.1% within 5 minutes of restart completing.

---

## 5. Database Issues

### Symptoms
- Logs contain: `HikariPool ... Connection is not available`, `FATAL: remaining connection slots are reserved`

### Actions

```bash
# Check active connections
psql -h $DB_HOST -U $DB_USER -c "
SELECT count(*), state, wait_event_type, wait_event
FROM pg_stat_activity
WHERE datname = 'orderdb'
GROUP BY state, wait_event_type, wait_event
ORDER BY count DESC;"

# Kill idle connections if pool is exhausted
psql -h $DB_HOST -U $DB_USER -c "
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'orderdb'
  AND state = 'idle'
  AND state_change < NOW() - INTERVAL '10 minutes';"
```

---

## 6. Upstream Dependency Issues

If a downstream service is returning 5xx:
1. Check the downstream service runbook
2. Verify circuit breaker is open in Resilience4j metrics: `/actuator/metrics/resilience4j.circuitbreaker.state`
3. If circuit breaker is open: the service is degraded gracefully — no action needed until upstream recovers
4. If circuit breaker is closed but errors persist: check network connectivity, security group rules, and DNS resolution

---

## 7. Memory / OOM Issues

### Symptoms
- Tasks/pods killed with exit code 137 (OOM)
- CloudWatch: `MemoryUtilization` > 90% for sustained period

### Actions

```bash
# Check heap dump (if `-XX:+HeapDumpOnOutOfMemoryError` configured)
aws s3 ls s3://your-bucket/heap-dumps/ --recursive | sort | tail -5

# Increase memory temporarily (ECS)
aws ecs update-service \
  --cluster prod-cluster \
  --service order-service \
  --task-definition order-service:NEXT_VERSION_WITH_MORE_MEMORY
```

---

## 8. Rollback Procedure

Only roll back if the issue is directly caused by the latest release.

```bash
# ECS — roll back to previous task definition
PREVIOUS_TD=$(aws ecs describe-services \
  --cluster prod-cluster \
  --services order-service \
  --query 'services[0].taskDefinition' --output text | \
  sed 's/:[0-9]*$/:PREV_VERSION/')

aws ecs update-service \
  --cluster prod-cluster \
  --service order-service \
  --task-definition $PREVIOUS_TD
```

```bash
# Kubernetes — roll back to previous revision
kubectl rollout undo deployment/order-service -n order-service
kubectl rollout status deployment/order-service -n order-service
```

---

## 9. Post-Incident Actions

- [ ] Confirm error rate is back to normal (< 0.1%)
- [ ] Notify stakeholders that the incident is resolved
- [ ] Create an incident record in PagerDuty / Jira
- [ ] Schedule a blameless post-mortem within 48 hours for P1, 5 business days for P2
- [ ] Open an RCA using `/rca "incident details"` and track corrective actions
- [ ] Update this runbook if any steps were unclear or incorrect

---

## 10. Related Resources

- CloudWatch Dashboard: `https://console.aws.amazon.com/cloudwatch/...`
- Service Logs: CloudWatch Log Group `/ecs/order-service`
- Metrics: Grafana dashboard `order-service-overview`
- Pact Consumer Tests: `https://pact-broker.internal/pacts/...`
- RCA Template: `/rca "P1 — order-service down"`
