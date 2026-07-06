---
name: mulesoft-adapter
description: >-
  Map Core Design Model IR to MuleSoft Anypoint internal artifacts. Load when technology_context.profile.skill is mulesoft-adapter.

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

- `src/output_workflow/_internal/Integration/IntegrationDesign.json`
- `src/output_workflow/_internal/Integration/IntegrationDesign.md` + `_Diagrams.mmd`
- `src/output_workflow/_internal/Integration/integration-apis.raml` or `openapi.yaml`
- `src/output_workflow/_internal/Application/Design.json` — API catalog cross-reference
- `src/output_workflow/_internal/Messaging/MessageDesign.json` — if IR.messaging non-empty
- `src/output_workflow/_internal/Security/Security.json` — OAuth/policy design

## Legacy migration

Map `legacyLogicMigration` to Process API subflows + DataWeave transforms — not Java.

## Patterns

API-led (system/process/experience), idempotent flows, error handlers, until-successful.
