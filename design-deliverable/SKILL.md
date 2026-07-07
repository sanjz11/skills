---
name: design-deliverable
description: >-
  Produce consolidated stakeholder deliverables. Primary goal: always output
  two self-contained files from whatever internal artifacts exist. Fill gaps
  from IR with documented assumptions.
---

# Design Deliverable

## Primary goal

Always produce **exactly two** shareable files at `src/output_workflow/` root — `consolidated_design.md` and `consolidated_design.json` — self-contained and consistent, built from whatever exists in `_internal/`.

## Success criteria

- [ ] `consolidated_design.md` exists — complete human-readable design
- [ ] `consolidated_design.json` exists — valid JSON aligned with `.md`
- [ ] `meta.adapterStatus`: `complete` | `partial` | `missing` (honest assessment)
- [ ] No third shareable file at workflow root
- [ ] Stack-specific sections inlined when adapter artifacts exist

## Mandatory sections in consolidated_design.md

1. Executive summary
2. Technology stack (from `technology_context`)
3. Requirements and scope
4. **Domain model (DDD)** — bounded contexts, aggregates, entities, value objects (from IR + `Domain/DomainDesign` when Java)
5. Business rules (full)
6. Legacy migration (full pseudo-code from adapter `.md` files)
7. Architecture + Mermaid diagrams
8. APIs / contracts
9. Security, data, messaging (when in IR or adapter artifacts)
10. **Presentation style** (frontend) — design tokens, layout, component styles from `PresentationStyle.*`
11. **Integration design** (MuleSoft) — API-led layers, flows, connectors, error handlers from `Integration/*`
12. NFR and deployment

## consolidated_design.json shape

Include: `technology`, `coreModelSummary`, `adaptedDesign`, `domainModel`, `apiSummary`, `meta`.

Set `meta.adapterStatus` and list `meta.missingArtifacts[]` when adapter step did not produce expected files.

## Handling missing adapter artifacts

| Situation | What to do |
|-----------|------------|
| Adapter artifacts missing | Set `adapterStatus: missing`; consolidate from IR; flag `[ADAPTER GAP]` prominently |
| Domain/DomainDesign missing (Java) | Include IR `boundedContexts` in deliverable; flag gap |
| PresentationStyle missing (React) | Include minimal token table from epic; flag gap |
| IntegrationDesign missing (MuleSoft) | Do not claim MuleSoft-specific design; flag gap |

## Do not

- Hide adapter failures — validation downstream depends on honest `adapterStatus`
- Say "see `_internal/` folder" — inline essential content

## Completion gate

Two root files exist; `adapterStatus` accurate; DDD and stack-specific sections present when artifacts exist.
