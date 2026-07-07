---
name: design-generic-core
description: >-
  Build archetype-aware technology-independent Core Design Model (IR). Primary
  goal: one complete stack-agnostic design model shaped by application archetype.
  Proceed even when uploads are partial — document assumptions.
---

# Archetype-Aware Core Design (IR)

## Primary goal

Produce a **complete, technology-independent** Core Design Model — `core_design_model.json` and `core_design_model.md` — shaped by the classified **application archetype** in `technology_context.json`.

## Archetype-first procedure (mandatory)

1. Read `technology_context.json` → `archetype.id`.
2. Read `inputs_manifest.json` → consume `resolved.epic`, `resolved.domain_model`, `resolved.patterns`, `resolved.integration_spec`, `resolved.ux_spec`.
3. Read `_config/core-design-model-schema.json` → `archetypeRules[archetype.id]`.
3. Set `meta.archetypeId` and `meta.archetypeSource` in IR.
4. Populate **required** sections for that archetype; leave de-emphasized sections empty with reason in assumptions.

## IR shape by archetype

| archetype.id | Primary IR sections | DDD (boundedContexts) |
|--------------|---------------------|------------------------|
| `backend-service` | capabilities, businessRules, apiOperations, data | **Mandatory** — full aggregates |
| `web-ui` | userExperience (screens, navigation, journeys), capabilities | Optional unless epic implies domain model |
| `mobile-ui` | userExperience + device/offline in integrations | Optional |
| `integration-service` | integrationFlows, integrations, messaging | Optional — prefer flows over aggregates |
| `data-pipeline` | dataPipelines, dataEntities, integrations | Not required |

### userExperience (web-ui / mobile-ui)

```json
"userExperience": {
  "journeys": [{ "id", "name", "actorRef", "steps" }],
  "screens": [{ "id", "name", "purpose", "entryPoints", "capabilityRefs" }],
  "navigation": { "structure", "routes" },
  "stateModel": { "globalState", "screenState" },
  "accessibility": []
}
```

Use generic terms: screen, view, journey — not React/Angular component names.

### integrationFlows (integration-service)

```json
"integrationFlows": [{
  "id", "name", "trigger", "steps",
  "sourceConnectors", "targetConnectors",
  "transformations", "errorHandling", "reliability"
}]
```

### dataPipelines (data-pipeline)

```json
"dataPipelines": [{
  "id", "name", "sources", "targets",
  "mappings", "transformations", "dataQualityRules", "schedule"
}]
```

### boundedContexts (backend-service only — mandatory)

Full DDD per context: aggregates, entities, valueObjects, domainEvents. Map every capability and dataEntity to a context.

## Success criteria

- [ ] `meta.archetypeId` matches `technology_context.archetype.id`
- [ ] All `archetypeRules[archetypeId].required` sections populated
- [ ] No stack-specific terminology in IR
- [ ] `designCapabilities` from executionContext traceable in IR sections
- [ ] Cross-cutting skills applied when archetype implies security/data/messaging

## Bootstrap

Read bundled `core-design-model-schema.json`; write to `_config/`. Curl fallback:
`https://pscode.lioncloud.net/rohranja3/coe-skills/-/raw/main/design-generic-core/core-design-model-schema.json`

## Rules (mandatory)

1. **Zero technology names** in IR
2. **Do not force backend DDD** on web-ui, mobile-ui, or integration archetypes
3. Business rules: BR-ID, business language — no pseudo-code in IR
4. Legacy: excerpts + business intent only in `legacyLogicMigration`

## Completion gate

IR matches archetypeRules for classified archetype; assumptions documented for every gap.
