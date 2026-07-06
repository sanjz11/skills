---
name: design-generic-enricher
description: >-
  Enrich Core Design Model IR with cross-cutting concerns (security, data,
  messaging). Technology-blind. Use for Security, Data, and Messaging Enricher agents.
---

# Generic Enricher Skill

## Purpose

Read `src/output_workflow/core_design_model.json`, enrich a **single cross-cutting section**, write IR back (bump `meta.v` patch).

## Rules

1. **Never** add technology-specific terms
2. Only add sections supported by epic, ADR, domain model, and existing IR
3. If epic has no messaging requirements, leave `messaging.events` empty — do not invent
4. Preserve all existing IR fields verbatim
5. Append changelog entry to `src/output_workflow/design_changelog.md`

## Security enricher adds

`security.authentication`, `security.authorization`, `security.roles`, `security.policies` — generic (RBAC, MFA requirement, token-based, etc.)

## Data enricher adds

`data.entities`, `data.relationships`, `data.persistence`, `data.migrations` — conceptual schema, not PostgreSQL DDL

## Messaging enricher adds

`messaging.events`, `messaging.commands`, `messaging.patterns`, `messaging.reliability` — generic pub/sub, delivery guarantees

## Do not

- Emit MessageDesign.json, Security.json, Database.json — Adapter maps from IR
