---
name: design-generic-security
description: >-
  Add generic security section to Core Design Model IR. Technology-blind.
  Use in Core Design Model Agent when enriching IR.security.
---

# Generic Security (IR)

Add to `core_design_model.json` → `security`:

- `authentication` — patterns (token, session, MFA requirements)
- `authorization` — RBAC/ABAC, permission model
- `roles` — role definitions and hierarchies
- `policies` — compliance, threat mitigation (generic)

Only add what epic/ADR require. No vendor names (no Keycloak, Cognito).
