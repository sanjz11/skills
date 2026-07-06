---
name: generic-adapter
description: >-
  Fallback adapter mapping IR to generic layered design artifacts when no specific
  stack adapter matches. Use in Technology Adapter Agent as default.
---

# Generic Adapter

When profile is `generic-backend` or no specific adapter skill exists.

## Outputs

- `src/output_workflow/Application/Design.json` — layers mapped to generic names from `layerMapping`
- `src/output_workflow/Application/Design.md`
- `src/output_workflow/Application/openapi.yaml` — from apiOperations
- Optional Security, Database, Messaging JSONs if IR sections populated

Use pseudocode for legacy migration. No framework-specific annotations.
