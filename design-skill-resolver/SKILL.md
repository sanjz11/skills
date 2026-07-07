---
name: design-skill-resolver
description: >-
  Resolve and load exactly ONE design skill pack from technology_context
  executionContext. Primary goal: dependency-injection style adaptation — the
  agent must never see other technology adapter skills.
---

# Design Skill Resolver

## Primary goal

Resolve **one concrete design skill pack** from `technology_context.json` and execute adaptation using **only that pack**. Other technology adapters must not exist from the agent's perspective.

## Dependency injection model

```
technology_context.executionContext.loadedSkillPack  →  ONE adapter skill
```

| Allowed | Forbidden |
|---------|-----------|
| Load `loadedSkillPack` via `skills` tool | Load any other `*-adapter` skill |
| Curl references from `loadedSkillPack/` folder only | Curl or read other adapter folders |
| Read `technology_context` + IR + registry | Guess stack from epic when context exists |

## Procedure

1. Read `src/output_workflow/_internal/technology_context.json`.
2. Validate:
   - `archetype.id` present
   - `executionContext.loadedSkillPack` present (e.g. `java-adapter`, `react-adapter`)
   - `profile.skill` matches `loadedSkillPack`
3. **Load only** `executionContext.loadedSkillPack` via the `skills` tool.
4. If skill package files not visible, curl from:
   `https://pscode.lioncloud.net/rohranja3/coe-skills/-/raw/main/<loadedSkillPack>/`
5. Load each file in `profile.references` from that pack only.
6. Execute adaptation per loaded pack SKILL.md, `designCapabilities[]`, and **`profile.designContracts[]`**.
7. **Design contracts (mandatory):** for each contract where `mandatory: true` (or `when` condition met), generate file at `path` from IR `source`:
   - `openapi` → OpenAPI 3.1 YAML from `apiOperations`
   - `raml` → RAML 1.0 from `apiOperations` / Experience API layer
   - `asyncapi` → AsyncAPI 2.6+ from `messaging` when events/commands exist
8. Write artifacts under `src/output_workflow/_internal/` per profile `artifacts` and archetype.

## designCapabilities execution

`technology_context.executionContext.designCapabilities[]` lists **what** to design (planner output). The loaded skill pack defines **how** (stack-specific).

| Archetype | Example capabilities | Pack produces |
|-----------|---------------------|---------------|
| backend-service | design-domain-model, design-api-contract | Domain/, Application/, openapi |
| web-ui | design-screens, design-presentation-style | Presentation/, Application/ |
| integration-service | design-integration-flow, design-connectors | Integration/, Messaging/ |
| mobile-ui | design-screens, design-device-capabilities | Presentation/, Application/ |
| data-pipeline | design-pipeline, design-mapping | Data/PipelineDesign (generic-adapter fallback) |

## Gate (mandatory)

If `technology_context` has `error` or missing `loadedSkillPack`:

1. Re-bootstrap registry (`design-technology-base` curl fallback).
2. Re-read or rebuild `technology_context.json`.
3. **Do not** adapt with a different skill than `loadedSkillPack`.
4. **Do not** produce technology-agnostic placeholder designs.

## Success criteria

- [ ] Exactly one skill pack loaded (`loadedSkillPack`)
- [ ] All **mandatory** `profile.designContracts[]` files exist and align with IR
- [ ] Optional `asyncapi` contract when IR messaging non-empty
- [ ] Archetype-specific mandatory folders exist (see agent-contracts expectedArtifactsByProfile)
- [ ] Every JSON artifact includes `meta.techProfile`, `meta.irVersion`, `meta.adapter`, `meta.archetypeId`

## Completion gate

Adaptation complete using single resolved pack; no cross-technology skill leakage.
