---
name: design-generic-data
description: >-
  Enrich IR with technology-agnostic data design. Primary goal: model entities
  and persistence from domain/inputs or inferred from APIs — never skip because
  epic omitted database section.
---

# Generic Data (IR enrichment)

## Primary goal

Populate `data`, `dataEntities`, and relationships in the IR with conceptual schema, persistence approach, migrations, performance notes, and backup strategy — **from domain model and APIs when epic lacks a database section** — with every inference documented.

## Success criteria

- [ ] `dataEntities[]` aligns with `apiOperations` and `boundedContexts` when those exist
- [ ] `data.architecture`, `data.entities`, `data.relationships` populated or empty with assumption
- [ ] Conceptual types only — no SQL/DDL/vendor syntax
- [ ] `meta.assumptions[]` for every entity or pattern not directly stated in inputs
- [ ] IR updated; `meta.v` bumped; no `Database.json` emitted

**IR file:** `src/output_workflow/_internal/core_design_model.json`

## When to use

Load when requirements imply persistence **or** when APIs/entities imply stored data (default for backend/fullstack).

## Domain responsibilities

### Data architecture
Storage type (relational/document/etc.), topology, HA, scalability — conceptual.

### Schema design
Entities, attributes, relationships, indexing intent, normalization/denormalization.

### Migrations
Versioning, rollback, zero-downtime concepts.

### Performance
Access patterns, caching, capacity notes.

### Backup / recovery
RTO/RPO when ADR states; generic backup strategies otherwise.

## Handling missing or incomplete inputs

Data enrichment **must proceed** when APIs or entities exist without a database chapter in epic.

| Situation | What to do |
|-----------|------------|
| No domain model upload | Derive entities from `apiOperations` request/response schemas and resource names |
| Epic silent on DB type | Default `relational` for CRUD APIs; assumption with `[REVIEW]` |
| No migration requirements | Include minimal `migrations.strategy: "incremental"` baseline |
| No RTO/RPO in ADR | Omit numeric targets; note "TBD with operations" in assumption |
| Frontend/mobile only | Minimal client-side store entities only; defer server schema |
| No persistence needed | Leave `data` sparse; assumption: "stateless/BFF-only per category" |

Use `clarify` when ADR implies both SQL and document store without choosing.

## What you can do

- Design conceptual ERD, entities, relationships, indexing intent
- Cross-link `dataEntities` and `boundedContexts`
- Document migration and backup at concept level

## What you cannot do

- Emit SQL, MongoDB commands, or `Database.json`
- Recommend a specific database vendor as the only option
- Invent entities unrelated to capabilities/APIs

## IR output structure

```json
{
  "dataEntities": [{ "id": "ENT-001", "name": "", "attributes": [], "boundedContext": "" }],
  "data": {
    "architecture": { "storageType": "relational", "topology": "", "ha": {}, "scalability": {} },
    "entities": [],
    "relationships": [],
    "persistence": { "accessPatterns": [], "indexing": [], "caching": {} },
    "migrations": { "strategy": "", "versioning": {}, "rollback": {} },
    "performance": { "optimization": [] },
    "backup": { "strategy": [], "rto": "", "rpo": "" }
  }
}
```

## Completion gate

Entities consistent with APIs; assumptions documented for all inferred schema decisions.
