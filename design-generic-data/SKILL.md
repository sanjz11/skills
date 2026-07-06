---
name: design-generic-data
description: >-
  Add generic data section to Core Design Model IR. Technology-blind.
  Use in Core Design Model Agent when enriching IR.data.
---

# Generic Data (IR)

Add to `core_design_model.json` → `data`:

- `entities`, `relationships` — conceptual schema
- `persistence` — storage approach (generic)
- `migrations` — strategy (generic)

Use conceptual types only (string, uuid, datetime). No PostgreSQL/MongoDB DDL.
