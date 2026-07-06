---
name: angular-adapter
description: >-
  Map Core Design Model IR to Angular design artifacts. Use only in Technology
  Adapter Agent when technology_context.profileId is angular.
---

# Angular Adapter

Read `technology_context.json` + `core_design_model.json`. Map generic layers using `profile.layerMapping`. **Do not** emit server-side framework artifacts.

## Layer → artifact mapping

| IR layer | Angular artifacts |
|----------|-------------------|
| presentation | Component, Module, Routing module, template |
| business | Injectable service, facade |
| data | Repository, HttpClient service, model interface |
| integration | HTTP interceptor, guard |
| configuration | environment.ts design, app config |

## Outputs

- `src/output_workflow/Presentation/PresentationDesign.json` — modules, components, routes, services
- `src/output_workflow/Presentation/PresentationDesign.md` + `_Diagrams.mmd`
- `src/output_workflow/Application/Design.json` — client API contracts and DTO interfaces
- Skip Security.json, Database.json, MessageDesign.json unless IR sections are non-empty and relevant to the web client

## Legacy migration

For each `legacyLogicMigration[]` entry: TypeScript pseudo-code in PresentationDesign.md with traceability comments. Server-side legacy logic: document as backend API dependency.

## Rules

- RxJS observables for async flows where applicable
- camelCase for members; PascalCase for classes and components
- Module-based structure per `technology_context.architecture`
- Reference `apiOperations` for HttpClient endpoint design
