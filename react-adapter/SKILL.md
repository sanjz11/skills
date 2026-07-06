---
name: react-adapter
description: >-
  Map Core Design Model IR to React (web) design artifacts. Use only in Technology
  Adapter Agent when technology_context.profileId is react.
---

# React (Web) Adapter

Read `technology_context.json` + `core_design_model.json`. Map generic layers using `profile.layerMapping`. **Do not** emit server-side framework artifacts.

## Layer → artifact mapping

| IR layer | React artifacts |
|----------|-----------------|
| presentation | Page, Component, Route, layout |
| business | Custom hook, Context provider, use-case module |
| data | API client, store slice, query cache |
| integration | API layer / BFF client, interceptors |
| configuration | env config, build config |

## Outputs

- `src/output_workflow/Presentation/PresentationDesign.json` — pages, routes, components, state model
- `src/output_workflow/Presentation/PresentationDesign.md` + `_Diagrams.mmd`
- `src/output_workflow/Application/Design.json` — client layer only (API client, DTOs, error mapping)
- Skip Security.json, Database.json, MessageDesign.json unless IR sections are non-empty and relevant to the web client

## Legacy migration

For each `legacyLogicMigration[]` entry: TypeScript/JSX pseudo-code in PresentationDesign.md with traceability comments. Server-side legacy logic: document as backend API dependency, not server framework code.

## Rules

- TypeScript/JSX for client business logic
- Reference `apiOperations` for API client method design
- camelCase for functions/variables; PascalCase for components
- Component composition and state-management patterns per `technology_context.patterns`
