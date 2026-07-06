---
name: mulesoft-adapter
description: >-
  Map Core Design Model IR to MuleSoft Anypoint artifacts. Use only in Technology
  Adapter Agent when profile is mulesoft.
---

# MuleSoft Adapter

Map IR to API-led connectivity artifacts.

## Layer → artifact mapping

| IR layer | MuleSoft artifacts |
|----------|-------------------|
| presentation | Experience API (RAML) |
| business | Process API flows, subflows |
| data | System API |
| integration | Connectors, Transform Message (DataWeave) |
| configuration | API Manager policies, properties |

## Outputs

- `src/output_workflow/Integration/IntegrationDesign.json`
- `src/output_workflow/Integration/IntegrationDesign.md` + `_Diagrams.mmd`
- `src/output_workflow/Integration/integration-apis.raml` or `openapi.yaml`
- `src/output_workflow/Application/Design.json` — API catalog cross-reference
- `src/output_workflow/Messaging/MessageDesign.json` — if IR.messaging non-empty
- `src/output_workflow/Security/Security.json` — OAuth/policy design

## Legacy migration

Map `legacyLogicMigration` to Process API subflows + DataWeave transforms — not Java.

## Patterns

API-led (system/process/experience), idempotent flows, error handlers, until-successful.
