---
name: design-deliverable
description: >-
  Produce consolidated stakeholder deliverables. Primary goal: always output
  two self-contained files from IR + single-pack adapter artifacts. Archetype-
  appropriate sections; honest adapterStatus.
---

# Design Deliverable

## Primary goal

Always produce **exactly two** shareable files at `src/output_workflow/` root — self-contained, archetype-appropriate, with honest `meta.adapterStatus`.

## Archetype-aware sections

Include sections matching `technology_context.archetype.id`:

| Archetype | Mandatory deliverable emphasis |
|-----------|-------------------------------|
| `backend-service` | DDD domain model, APIs, persistence, messaging |
| `web-ui` | User journeys, screens, navigation, presentation style |
| `mobile-ui` | Screens, navigation, device/offline concerns, style |
| `integration-service` | Integration flows, connectors, transforms, error handling |
| `data-pipeline` | Pipeline design, mappings, DQ rules, scheduling |

## consolidated_design.json meta (required)

```json
"meta": {
  "archetypeId": "...",
  "loadedSkillPack": "...",
  "adapterStatus": "complete | partial | missing",
  "missingArtifacts": [],
  "missingContracts": [],
  "designContracts": [
    { "id": "...", "format": "openapi|raml|asyncapi", "path": "...", "operationCount": 0, "status": "present|missing|n/a" }
  ]
}
```

Copy `loadedSkillPack` from `technology_context.executionContext`. Populate `designContracts` from `technology_context.profile.designContracts` and verify files exist under `_internal/`. List mandatory missing paths in `missingContracts[]`.

## Mandatory sections in consolidated_design.md

1. Executive summary
2. Application archetype + technology stack
3. Requirements and scope
4. **Archetype primary model** (DDD / UX / integration flows / data pipelines)
5. Business rules
6. Legacy migration (pseudo-code from adapter `.md`)
7. Architecture + Mermaid
8. Stack-specific inlined design from adapter artifacts
9. **Design contracts** — format, path, operation/event count per `profile.designContracts`
10. NFR and deployment

## Handling missing adapter artifacts

Set `adapterStatus: missing`; flag `[ADAPTER GAP]`; consolidate from IR. Do not claim stack-specific design without adapter artifacts.

## Completion gate

Two root files; `archetypeId` + `loadedSkillPack` + `adapterStatus` accurate.
