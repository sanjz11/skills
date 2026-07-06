---
name: design-generic-enricher
description: >-
  Shared rules for enriching the Core Design Model IR with cross-cutting
  concerns. Use alongside design-generic-security, design-generic-data, and
  design-generic-messaging when patching the IR.
---

# Generic IR Enrichment (shared rules)

## Purpose

Patch `src/output_workflow/_internal/core_design_model.json` with technology-blind cross-cutting content. Each domain skill (security, data, messaging) owns its section; this skill defines **common rules**.

## Rules

1. **Never** add technology-specific terms to the IR
2. Only add sections supported by epic, ADR, domain model, and existing IR
3. If requirements do not apply, leave the section empty — do not invent
4. Preserve all existing IR fields verbatim unless patching a named change
5. Bump `meta.v` patch on every IR update
6. Document assumptions in `meta.assumptions[]` when information is incomplete

## Workflow modes

**CREATE:** build from requirements inputs.

**UPDATE:** read existing IR first; patch only what change requirements name; copy everything else verbatim; append `src/output_workflow/_internal/design_changelog.md`.

## Section ownership

| Skill | IR sections |
|-------|-------------|
| design-generic-security | `security`, related policy notes |
| design-generic-data | `data`, `dataEntities`, relationships |
| design-generic-messaging | `messaging`, event/command links to `integrations` |

## Do not

- Emit stack-specific artifact files (`Security.json`, `Database.json`, `MessageDesign.json`) — technology adapter maps from IR
- Emit consolidated deliverables
- Contradict ADR; ADR wins — document conflicts in `meta.assumptions`
