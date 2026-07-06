---
name: design-generic-core
description: >-
  Build the technology-independent Core Design Model (IR). Load when creating
  the canonical design model before any stack-specific adaptation. Never emit
  framework, language, or vendor names in the IR.
---

# Generic Core Design

## Purpose

Produce `src/output_workflow/_internal/core_design_model.json` — the **intermediate representation** consumed by cross-cutting enrichment skills and the technology adapter.

Also produce `src/output_workflow/_internal/core_design_model.md` for human-readable IR documentation.

## Schema

Read `config/core-design-model-schema.json` for structure.

## Rules (mandatory)

1. **Zero technology names** — no framework classes, annotations, UI libraries, or vendor products
2. Use **generic layers**: presentation, business, data, integration, configuration
3. Assign each capability/operation to a layer in `layers.*`
4. Business rules: BR-ID, business language, pre/post conditions — **no pseudo-code** in IR
5. Legacy logic: `legacyLogicMigration[]` with `sourceLanguage`, `sourceExcerpt`, `businessIntent`, `targetLayer: business` — adapter renders pseudo-code later
6. `apiOperations[]`: method, path, operationId, summary, requestSchema, responseSchema, errors
7. `dataEntities[]` — conceptual types only (string, integer, uuid, datetime)
8. `boundedContexts[]` — generic terms: aggregate, entity, value-object, domain-event

## Cross-cutting enrichment

When requirements apply, also load and apply:
- `design-generic-security` → `security`
- `design-generic-data` → `data`, `dataEntities`
- `design-generic-messaging` → `messaging`

Apply all loaded skills in **one IR pass**; preserve fields; bump `meta.v` on updates.

## Layer assignment

| Concept | Layer |
|---------|-------|
| User-facing interaction, API surface, UI concern | presentation |
| Business rules, workflows, calculations | business |
| Persistence, entities, queries | data |
| External systems, events, connectors | integration |
| Env, feature flags, deployment params | configuration |

## Output files

- `src/output_workflow/_internal/core_design_model.json` (compact JSON)
- `src/output_workflow/_internal/core_design_model.md` (readable + Mermaid)

## Do not

- Generate openapi.yaml, Design.json, or stack artifacts — adapter step does that
- Emit consolidated deliverables
- Reference a specific stack except optional notes in `meta.assumptions`
