---
name: design-generic-security
description: >-
  Enrich IR with technology-agnostic security. Primary goal: populate security
  from requirements or category baselines — never skip because epic omitted
  security. Document every assumption.
---

# Generic Security (IR enrichment)

## Primary goal

Populate `core_design_model.json` → `security` with complete, technology-agnostic authentication, authorization, roles, API protection, and policies — **even when epic/ADR do not mention security explicitly** — using category-appropriate baselines and documented assumptions.

## Success criteria

- [ ] `security.authentication`, `security.authorization`, `security.roles`, `security.policies` present or explicitly empty with assumption
- [ ] Roles align with `requirements.actors` when actors exist
- [ ] API protection reflects `apiOperations` when APIs exist
- [ ] No vendor or framework names (Keycloak, OAuth product names, etc.)
- [ ] Every baseline or inferred control recorded in `meta.assumptions[]`
- [ ] IR file updated; `meta.v` bumped; no `Security.json` emitted

**IR file:** `src/output_workflow/_internal/core_design_model.json`

## When to use

Load during IR build when epic/ADR imply security **or** when the system exposes APIs, user accounts, or sensitive data (apply baseline even if epic is silent).

## Domain responsibilities

### Authentication
Token/session/certificate patterns; MFA; token lifecycle; federated identity (conceptual); credential policy.

### Authorization
RBAC, ABAC, PBAC; permissions; resource rules; policy enforcement points.

### Roles
Business-term role definitions, hierarchies, assignment patterns.

### API / edge security
Rate limiting, throttling, API keys, request hardening (generic).

### Policies
Compliance **only when stated in ADR**; threat mitigation; audit/logging requirements.

## Handling missing or incomplete inputs

Security enrichment **must not be skipped** for backend/fullstack APIs without an explicit epic section.

| Situation | Baseline to apply |
|-----------|-------------------|
| Epic silent on auth | Token-based API auth + RBAC with default roles `anonymous`, `authenticated`, `admin` — `[REVIEW]` |
| No roles in domain model | Derive from actors and API operations |
| No compliance in ADR | Omit compliance policies; do not invent GDPR/HIPAA |
| Public API only | Document public vs protected routes in `authorization.resourceRules` |
| ADR mandates MFA | Include MFA in authentication; do not downgrade |

Use `clarify` only for mutually exclusive models (e.g. ABAC vs simple RBAC) when ADR is silent and API count > 10.

Record assumptions:

```json
{
  "id": "ASM-SEC-001",
  "topic": "authentication",
  "assumption": "Bearer token API auth assumed",
  "reason": "Epic lists REST APIs but no auth section",
  "confidence": "medium",
  "impact": "Implementation may need IdP selection",
  "needsReview": true
}
```

## What you can do

- Design full auth/authz models, roles, API edge controls, audit requirements
- Populate compact JSON under `security`
- Preserve all other IR fields

## What you cannot do

- Name products (Keycloak, Cognito, Auth0)
- Write configs, code, or `Security.json`
- Invent compliance mandates not in ADR
- Skip security for API backends without documenting intentional public exposure

## IR output structure

```json
{
  "security": {
    "authentication": { "patterns": [], "mfa": {}, "tokenLifecycle": {} },
    "authorization": { "model": "RBAC", "permissions": [], "resourceRules": [] },
    "roles": [{ "id": "ROLE-001", "name": "", "permissions": [] }],
    "apiGateway": { "rateLimiting": {}, "apiKeys": {} },
    "policies": [{ "id": "POL-001", "type": "", "summary": "", "requirements": [] }]
  }
}
```

## Completion gate

`security` populated or consciously empty with assumption; `meta.assumptions` complete.
