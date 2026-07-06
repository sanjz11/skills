---
name: design-generic-core
description: >-
  Build the technology-independent Core Design Model (IR). Use for Core Design
  Model Agent only. Never emit Java, React, MuleSoft, or any stack-specific terms.
---

# Generic Core Design Skill

## Purpose

Produce `src/output_workflow/core_design_model.json` — the **intermediate representation** consumed by enrichers and the Technology Adapter.

## Schema

Read `config/core-design-model-schema.json` for structure.

## Rules (mandatory)

1. **Zero technology names** — no Controller, Service, Screen, Flow, @Annotation, Redux, DataWeave
2. Use **generic layers**: presentation, business, data, integration, configuration
3. Assign each capability/operation to a layer in `layers.*`
4. Business rules: BR-ID, business language, pre/post conditions — no pseudo-code in IR
5. Legacy logic: store in `legacyLogicMigration[]` with `sourceLanguage`, `sourceExcerpt`, `businessIntent`, `targetLayer: business` — adapter renders pseudo-code later
6. API operations in `apiOperations[]`: method, path, operationId, summary, requestSchema, responseSchema, errors
7. Data in `dataEntities[]` and `data.entities[]` — conceptual types only (string, integer, uuid, datetime)
8. DDD as `boundedContexts[]` — generic terms: aggregate, entity, value-object, domain-event

## Layer assignment guide

| Concept | Layer |
|---------|-------|
| User-facing interaction, API surface, UI concern | presentation |
| Business rules, workflows, calculations | business |
| Persistence, entities, queries | data |
| External systems, events, connectors | integration |
| Env, feature flags, deployment params | configuration |

## Output files

- `src/output_workflow/core_design_model.json` (compact JSON)
- `src/output_workflow/core_design_model.md` (human-readable IR documentation + Mermaid)

## Do not

- Generate openapi.yaml, Design.json, or stack artifacts — Adapter does that
- Reference technology_context except in meta.notes if needed
