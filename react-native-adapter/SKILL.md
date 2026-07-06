---
name: react-native-adapter
description: >-
  Map Core Design Model IR to React Native design artifacts. Use only in Technology
  Adapter Agent when profile is react-native or category is mobile.
---

# React Native Adapter

Map IR layers using `profile.layerMapping`. **Do not** emit Spring/Java artifacts.

## Layer → artifact mapping

| IR layer | React Native artifacts |
|----------|------------------------|
| presentation | Screen, Component, Navigation stack |
| business | Custom hook, use-case module |
| data | API client, repository, AsyncStorage/SQLite adapter |
| integration | API layer / BFF client |
| configuration | env config, app.json design |

## Outputs

- `src/output_workflow/Presentation/PresentationDesign.json` — screens, navigation, components, state
- `src/output_workflow/Presentation/PresentationDesign.md` + `_Diagrams.mmd`
- `src/output_workflow/Application/Design.json` — **client layer only** (API client, DTOs, error mapping)
- Skip Security.json, Database.json, MessageDesign.json unless IR sections are non-empty and relevant to mobile

## Rules

- TypeScript/JSX pseudo-code for client business logic only
- Server-side legacy SP migration: document as backend API dependency, not Java service code
- Reference `apiOperations` for API client method design
