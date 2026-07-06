---
name: design-generic-data
description: >-
  Enrich the Core Design Model IR with technology-agnostic data and persistence
  design. Load during IR build when epic/ADR require schema, storage, migration,
  performance, or backup/recovery. No database vendor or DDL syntax.
---

# Generic Data (IR enrichment)

## When to use

Load this skill when building or patching `core_design_model.json` and requirements imply persistent data, schema, migrations, or data architecture.

**Output target:** enrich `data`, `dataEntities`, and related IR sections — do **not** create `Database.json` or vendor-specific DDL (the technology adapter maps IR later).

**IR file:** `src/output_workflow/_internal/core_design_model.json`

---

## Primary goal

Analyze application data requirements and produce technology-agnostic data design in the IR — architecture concepts, conceptual schema, migration approach, performance considerations, and backup/recovery — implementable on any database platform.

---

## Responsibilities

### 1. Data architecture
- Generic storage types: relational, document, key-value, graph, columnar, time-series
- Topology: single instance, primary-replica, multi-master, sharding (conceptual)
- Deployment: on-premise, cloud, hybrid, managed service (generic)
- High availability and replication patterns
- Scalability: horizontal/vertical, partitioning
- **No** vendor-specific implementations

### 2. Schema design
- Conceptual entities/collections, attributes, relationships
- Generic data types and constraints (string, uuid, datetime, decimal)
- Normalization (relational) or denormalization (document) strategies
- Indexing strategies (primary, secondary, composite, unique — generic)
- Referential integrity patterns
- Align with `dataEntities[]` and `boundedContexts[]`

### 3. Migration strategies
- Versioning and rollout approaches
- Rollback and testing patterns
- Zero-downtime migration concepts
- **No** Flyway/Liquibase/vendor tool syntax

### 4. Performance optimization
- Query optimization patterns (generic)
- Indexing for access patterns
- Caching strategies
- Connection pooling (conceptual)
- Capacity planning guidance (not instance sizing)

### 5. Backup and recovery
- Backup types: full, incremental, differential, continuous
- RTO/RPO requirements when stated in ADR
- Disaster recovery patterns
- Retention and archival strategies
- Point-in-time recovery (conceptual)

---

## What you can do

- Design conceptual schemas and entity relationships
- Map entities to bounded contexts and `apiOperations`
- Define generic indexing and access-pattern notes
- Document migration, HA, scaling, and backup strategies at concept level
- Specify transaction/consistency models (ACID vs BASE) when relevant
- Add multi-tenancy patterns when ADR requires
- Populate IR `data` with compact JSON
- Preserve existing IR fields; bump `meta.v` on change

## What you cannot do

- Generate SQL DDL, MongoDB commands, or proprietary syntax
- Recommend specific database vendors without neutral comparison
- Design schemas without grounding in epic/domain model
- Emit `Database.json`, `Database.md`, or files outside the IR
- Specify exact cloud instance types or hardware
- Invent data requirements not in epic/ADR
- Over-index or denormalize without justified access patterns

---

## How to apply (procedure)

1. Read epic, ADR, patterns, domain modelling, and existing IR.
2. Cross-reference `dataEntities[]`, `apiOperations[]`, and `businessRules`.
3. Enrich `data` and align or extend `dataEntities[]`.
4. Add only sections requirements support.
5. Write updated IR; bump `meta.v`.

---

## IR output structure

```json
{
  "dataEntities": [
    { "id": "ENT-001", "name": "", "attributes": [], "boundedContext": "" }
  ],
  "data": {
    "architecture": {
      "storageType": "relational",
      "topology": "",
      "ha": {},
      "scalability": {}
    },
    "entities": [],
    "relationships": [],
    "persistence": {
      "accessPatterns": [],
      "indexing": [],
      "caching": {},
      "consistency": ""
    },
    "migrations": {
      "strategy": "",
      "versioning": {},
      "rollback": {}
    },
    "performance": {
      "optimization": [],
      "capacityNotes": ""
    },
    "backup": {
      "strategy": [],
      "rto": "",
      "rpo": "",
      "retention": {}
    }
  }
}
```

Use conceptual types only. Compact JSON; references/IDs for repeated structures.

---

## Input grounding

Ground every decision in epic, ADR, common patterns, and target domain modelling. Align entities and boundaries with domain model.
