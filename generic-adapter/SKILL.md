---
name: generic-adapter
description: >-
  Fallback adapter mapping IR to generic layered artifacts. Load when profile is generic-backend or no specific adapter matches.

# Generic Adapter

When profile is `generic-backend` or no specific adapter skill exists.

## Outputs

- `src/output_workflow/_internal/Application/Design.json` — layers mapped to generic names from `layerMapping`
- `src/output_workflow/_internal/Application/Design.md`
- `src/output_workflow/_internal/Application/openapi.yaml` — from apiOperations
- Optional Security, Database, Messaging JSONs if IR sections populated

Use pseudocode for legacy migration. No framework-specific annotations.
