# /threat-model — STRIDE Threat Model for a Service

Produce a structured STRIDE threat model for a service or bounded context.

## Usage

```
/threat-model "OrderService — accepts orders from web frontend, calls PaymentService and InventoryService, writes to PostgreSQL, publishes OrderPlaced events to Kafka"
```

## What This Command Does

1. Activates the `security-auditor` and `architect` agents
2. Identifies trust boundaries: client → service, service → dependencies, service → data stores
3. Applies STRIDE analysis across each trust boundary:
   - **S**poofing — can an attacker impersonate a user or service?
   - **T**ampering — can data be modified in transit or at rest?
   - **R**epudiation — can an actor deny performing an action?
   - **I**nformation Disclosure — can an attacker read data they shouldn't?
   - **D**enial of Service — can an attacker make the service unavailable?
   - **E**levation of Privilege — can an attacker gain permissions beyond their role?
4. Rates each threat: **Critical / High / Medium / Low** (CVSS-aligned)
5. Proposes a mitigation for each threat
6. Produces a risk register entry for `docs/security/threat-models/<service>.md`

## Output Format

The command produces:

### System Description
- Service name, technology stack, external actors, trust boundaries diagram (text)

### STRIDE Analysis Table

| Threat ID | Category | Component | Threat Description | Likelihood | Impact | Risk | Mitigation |
|---|---|---|---|---|---|---|---|
| T-001 | Spoofing | API Gateway | Attacker forges JWT token | Low | High | High | Validate JWT signature with JWKS; verify iss, aud, exp |

### Risk Register Summary
- Count of Critical / High / Medium / Low threats
- Top 3 threats requiring immediate attention

### Mitigations Backlog
- List of recommended security controls not yet implemented, formatted as tickets

## Tips

- The more detail you provide about integrations, the more accurate the threat model
- Run this command before ARB review for new services (`/review-architecture`)
- Update the threat model when: adding new integrations, changing auth mechanisms, or after a security incident
- Store the output in `docs/security/threat-models/<service-name>.md`
