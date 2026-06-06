---
mode: "agent"
description: "Incident to RCA workflow: Detection → Triage → War Room → Resolution → Post-Incident Review → Corrective Actions"
tools: ['read', 'edit', 'search', 'githubRepo']
---

## Incident to RCA Workflow

This workflow manages the full incident lifecycle from detection through resolution to root cause analysis and preventive action.

---

## Phase 1 — Incident Handler: Detection & Declaration

**Agent:** `@incident-handler`

**Trigger:** Alert fired, user report received, or anomalous monitoring signal

1. Classify severity against the severity matrix (P1–P4)
2. Declare the incident with standard template in `#incidents` Slack channel
3. Open the war room (Slack channel or conference bridge)
4. Assign roles: Incident Commander, Technical Lead, Comms Lead, Scribe
5. Post first stakeholder update within 15 minutes (P1) or 30 minutes (P2)

**P1 Declaration checklist:**
- [ ] Severity confirmed (P1 = complete outage or data loss risk)
- [ ] War room opened
- [ ] On-call tech lead paged
- [ ] VP Engineering notified
- [ ] Customer Success notified
- [ ] First external status update drafted (if customer-visible)

---

## Phase 2 — Ops Engineer: Triage & Diagnosis

**Agent:** `@ops-engineer`

1. Pull CloudWatch dashboard for the affected service
2. Run CloudWatch Logs Insights queries to identify error patterns
3. Check recent deployments (CodeDeploy, CDK, Lambda)
4. Check for upstream dependency failures (third-party APIs, shared services)
5. Check AWS Service Health Dashboard for platform issues
6. Document findings in war room with timestamps

**Diagnostic queries:**
```sql
-- Error rate spike
filter @message like /ERROR/
| stats count(*) by bin(1m)
| sort @timestamp desc

-- Recent exceptions
filter @message like /Exception/
| parse @message "* Exception" as type
| stats count(*) by type
| sort count desc
```

**Output before Phase 3:**
- [ ] Root cause hypothesis formed (or multiple hypotheses ranked)
- [ ] Affected components identified
- [ ] Impact scope quantified (% users, data at risk)

---

## Phase 3 — Incident Handler: Mitigation & Resolution

**Agent:** `@incident-handler`

1. Execute the resolution action (rollback, hotfix, config change, circuit breaker)
2. Monitor CloudWatch for recovery signal (error rate returning to baseline)
3. Run smoke tests against the affected service
4. Confirm recovery with the original reporter
5. Post resolution notification in `#incidents` and war room
6. Update external status page if applicable

**Resolution checklist:**
- [ ] Error rate returned to baseline
- [ ] Smoke tests passing
- [ ] No secondary incidents triggered
- [ ] Resolution notification posted
- [ ] War room closed

---

## Phase 4 — RCA Agent: Root Cause Analysis

**Agent:** `@rca-agent`

**Trigger:** Within 24 hours of incident resolution

1. Reconstruct the incident timeline from logs, monitoring data, and war room notes
2. Apply 5-Whys analysis to reach the systemic root cause
3. Identify contributing factors (separate from root cause)
4. Classify the root cause: Code Defect / Infrastructure / Process / Third-Party / Human Error
5. Produce SMART corrective actions with owners and due dates
6. Document what went well and what to improve

**RCA Document:**
Stored at: `docs/incidents/INC-{YYYYMMDD}-{NNN}-rca.md`

**Output before Phase 5:**
- [ ] Incident timeline complete (minute-by-minute)
- [ ] 5-Whys analysis complete with root cause identified
- [ ] Corrective actions table with owners and due dates
- [ ] "What went well" section completed

---

## Phase 5 — Project Tracker: Corrective Action Tracking

**Agent:** `@project-tracker`

1. Create GitHub Issues or Jira tickets for each corrective action
2. Assign to sprint backlog with priority (P1 corrective actions = next sprint)
3. Add to risk register if the issue represents an ongoing systemic risk
4. Schedule 30-day review to confirm corrective actions are implemented and effective

**Corrective Action Priority:**
- P1 (prevent recurrence of this exact incident): next sprint
- P2 (improve detection or response): within 2 sprints
- P3 (hygiene and resilience): backlog, scheduled

**Workflow Complete when:**
- [ ] RCA document published and linked from incident record
- [ ] All corrective actions tracked in project backlog
- [ ] Stakeholders notified of RCA findings and actions
- [ ] 30-day review scheduled
