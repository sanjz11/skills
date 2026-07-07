---
name: react-adapter
description: >-
  Map IR to React (Web) internal artifacts including routes, hooks, and
  presentation style tokens. Primary goal: always produce stack-specific files
  from IR + technology_context. Load when technology_context.profile.skill is
  react-adapter.
---

# React (Web) Adapter

## Primary goal

Transform `core_design_model.json` into **complete internal React (Web) design artifacts** — pages, routes, hooks, API clients, **and visual style/CSS token design** — under `_internal/`.

## Bootstrap fallback (references)

If `references/*` files are not visible next to `SKILL.md`, fetch from:

`https://pscode.lioncloud.net/rohranja3/coe-skills/-/raw/main/react-adapter/references/<filename>`

Use `command_line` (`curl -fsSL`) before proceeding without templates.

## Success criteria

- [ ] Read IR and `technology_context.json` first; abort if `technology_context.error` — re-bootstrap registry first
- [ ] `Presentation/PresentationDesign.json` — routes, pages, components, flow families
- [ ] **`Presentation/PresentationStyle.json`** — design tokens, component styles, per-view layout/CSS guidance
- [ ] **`Presentation/PresentationStyle.md`** — human-readable style guide with CSS variable table
- [ ] `Application/Design.json` — hooks, contexts, stores, API clients, interceptors
- [ ] `legacyLogicMigration` as **TypeScript/JSX** pseudo-code in `.md` files
- [ ] Each JSON includes `meta.techProfile`, `meta.irVersion`, `meta.adapter`

## Outputs (all required for frontend profile)

- `src/output_workflow/_internal/Presentation/PresentationDesign.json`
- `src/output_workflow/_internal/Presentation/PresentationDesign.md`
- `src/output_workflow/_internal/Presentation/PresentationStyle.json`
- `src/output_workflow/_internal/Presentation/PresentationStyle.md`
- `src/output_workflow/_internal/Application/Design.json`
- `src/output_workflow/_internal/Application/client-api.openapi.yaml` — when IR `apiOperations` non-empty (OpenAPI 3.1 client contract)

## Design contracts procedure

1. Load `references/contract-client-api-pattern.md`.
2. When IR `apiOperations` non-empty: generate `Application/client-api.openapi.yaml` for APIs the UI consumes.
3. Set `Application/Design.json` → `clientApiContractRef: client-api.openapi.yaml`.
4. UI-only epics with no APIs: skip contract file; note in `meta.assumptions`.

## Presentation style procedure (mandatory)

1. Load `references/presentation-style-pattern.md` and `PresentationStyle-json-shape.json`.
2. Build token catalog: colors, typography, spacing, radius, elevation.
3. For each view/page: document layout pattern and state visuals (loading, empty, error, success).
4. Bind shared components to CSS variables / token refs.
5. Document `cssStrategy` (default: CSS variables + CSS Modules when epic silent).
6. Support RTL token variant when epic/ADR mentions bidirectional layout.

## Layer mapping

| IR layer | React artifacts |
|----------|-----------------|
| presentation | Page, Component, Route, layout |
| business | Custom hook, Context provider |
| data | API client, store slice, query cache |
| integration | API layer / auth refresh client |
| configuration | env config, design tokens |

## Reference patterns

- `references/page-component-pattern.md`
- `references/presentation-style-pattern.md`
- `references/PresentationDesign-json-shape.json`
- `references/PresentationStyle-json-shape.json`

## Best practices

- Pages from routes in IR; one page component per connected view
- Hooks for business logic; no API calls directly in presentational components
- Loading-empty-error-success quartet on every data-bearing view
- Style tokens centralized — no hardcoded hex in component specs

## Do not

- Skip style artifacts — visual design is part of frontend adaptation
- Write consolidated deliverables
- Use react-native patterns unless `profileId` is `react-native`

## Completion gate

Presentation (design + style) and Application artifacts exist; style covers all routes in PresentationDesign.
