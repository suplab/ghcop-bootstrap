---
name: incident-response
description: 'ITIL-aligned P1/P2 incident response: declaration templates, stakeholder communication, war room coordination, and escalation matrix. Use when declaring or managing a production incident.'
---

# Incident Response Skill

ITIL-aligned templates and procedures for P1/P2 incident management.

## Severity Classification

| Priority | Criteria | Response SLA |
|----------|---------|-------------|
| P1 | Complete outage, data loss, security breach | 15 min |
| P2 | > 10% users affected, revenue impact | 30 min |
| P3 | < 10% users, workaround available | 4 hours |
| P4 | Single user, cosmetic | Next business day |

## Declaration Template

```
🔴 INCIDENT DECLARED — P{N}
ID: INC-{YYYYMMDD}-{NNN}
Time: {datetime UTC}
Commander: {name}
Impact: {plain-English description}
Affected: {service list}
War Room: {Slack channel / conference link}
Next Update: {datetime} ({N} min)
```

## Rolling Update Template (every 30 min for P1)

```
📊 INCIDENT UPDATE — INC-{ID} — {datetime}
Status: 🔴 ACTIVE / 🟡 MITIGATED / ✅ RESOLVED
Current Impact: {description}
Investigation: {what we know}
Next Action: {what we're doing now} — Owner: {name} — ETA: {time}
Next Update: {datetime}
```

## Resolution Template

```
✅ INCIDENT RESOLVED — INC-{ID}
Resolved: {datetime UTC}
Duration: {N}h {M}m
Resolution: {one sentence}
Impact Summary: {users affected, data affected}
PIR Scheduled: {date}
Corrective Actions: [see PIR document]
```

## Escalation Matrix

| Time Since Declaration | Action |
|----------------------|--------|
| P1: 15 min | Notify Engineering Manager |
| P1: 30 min | Notify VP Engineering |
| P1: 45 min | Notify CTO, Customer Success |
| P2: 60 min | Notify Engineering Manager if no progress |

## War Room Roles

| Role | Responsibility |
|------|--------------|
| Incident Commander | Coordinates, communicates, tracks progress |
| Technical Lead | Diagnoses root cause, proposes fixes |
| Comms Lead | Drafts stakeholder updates |
| Scribe | Records timeline in real time |
