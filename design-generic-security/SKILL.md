---
name: design-generic-security
description: >-
  Enrich the Core Design Model IR with technology-agnostic security design.
  Load during IR build when epic/ADR require authentication, authorization,
  roles, API gateway patterns, or security policies. No vendor or framework names.
---

# Generic Security (IR enrichment)

## When to use

Load this skill when building or patching `core_design_model.json` and requirements (epic, ADR, patterns, domain model) imply security concerns.

**Output target:** enrich `security` on the IR only — do **not** create `Security.json` or stack-specific security artifacts (the technology adapter maps IR later).

**IR file:** `src/output_workflow/_internal/core_design_model.json`

---

## Primary goal

Produce comprehensive, technology-agnostic security content in the IR — authentication, authorization, roles, API surface protection, and policies — implementable with any identity provider or security stack.

---

## Responsibilities

### 1. Authentication design
- Generic patterns: token-based, session-based, certificate-based
- Identity provider patterns: federated identity, SSO (conceptual)
- MFA requirements
- Token management and refresh strategies
- Password policies and credential management
- **No** vendor names (Keycloak, Cognito, Auth0, etc.)

### 2. Authorization design
- Models: RBAC, ABAC, PBAC
- Permission structures and access control
- Resource-based authorization
- Policy-based access control
- Authorization decision points
- **No** framework-specific syntax

### 3. Roles design
- Role definitions and hierarchies
- Role assignment patterns
- Role permissions and constraints
- Default roles and lifecycle
- Dynamic roles when required
- Express roles in **business terms**

### 4. API gateway / edge security
- Rate limiting and throttling (generic)
- API key management patterns
- Request/response transformation for security
- Circuit breaker patterns for security resilience
- **No** gateway product configuration

### 5. Security policies
- Compliance requirements when stated in ADR (GDPR, HIPAA, PCI-DSS, SOC2)
- Threat mitigation strategies
- Security monitoring and auditing requirements
- Incident response procedures (high level)

---

## What you can do

- Design generic authentication and authorization models with clear permission structures
- Define role hierarchies using business terminology
- Specify MFA, token lifecycle, federated identity, and SSO at a conceptual level
- Document API edge security: rate limits, throttling, API keys
- Generate policy and compliance notes grounded in ADR/epic
- Create threat mitigation and audit/logging requirements
- Populate IR `security` with compact, valid JSON structures
- Preserve all existing IR fields; bump `meta.v` patch on change

## What you cannot do

- Specify concrete products ("use Keycloak", "use AWS Cognito")
- Write production configs, code, or deployment scripts
- Recommend vendors or make technology selection decisions
- Emit `Security.json`, `Security.md`, or files outside the IR
- Invent security requirements not supported by epic/ADR
- Design controls that violate least privilege or defense-in-depth
- Assume existing infrastructure without documenting assumptions in `meta.assumptions`

---

## How to apply (procedure)

1. Read epic, ADR, common patterns, and domain modelling.
2. Read existing `core_design_model.json` if present.
3. Enrich **only** `security` (and related `nfr` / `observability` cross-refs if needed).
4. Ground every decision in inputs; cite ADR for cross-cutting choices.
5. If epic has no security requirements, leave `security` minimal or empty — do not invent.
6. Write updated IR; bump `meta.v`.

---

## IR output structure

Map domain expertise into `core_design_model.json` → `security`:

```json
{
  "security": {
    "authentication": {
      "patterns": ["token-based"],
      "mfa": {},
      "tokenLifecycle": {},
      "identityFederation": {},
      "credentialPolicy": {}
    },
    "authorization": {
      "model": "RBAC",
      "permissions": [],
      "resourceRules": [],
      "policyEnforcement": {}
    },
    "roles": [
      { "id": "ROLE-001", "name": "", "permissions": [], "hierarchy": [] }
    ],
    "apiGateway": {
      "rateLimiting": {},
      "apiKeys": {},
      "transformations": {}
    },
    "policies": [
      { "id": "POL-001", "type": "compliance|threat|audit", "summary": "", "requirements": [] }
    ]
  }
}
```

Use compact JSON (short keys where clear, no redundant nesting). Align roles with `requirements.actors` and `businessRules` where applicable.

---

## Input grounding

Ground every decision in:
- Epic template
- ADR blueprint
- Common patterns
- Target domain modelling

When ADR conflicts with epic, follow ADR and document in `meta.assumptions`.
