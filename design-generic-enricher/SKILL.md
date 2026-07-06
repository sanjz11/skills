---
name: design-generic-enricher
description: >-
  Shared rules for IR cross-cutting enrichment. Primary goal: patch IR safely
  when enriching security, data, or messaging. Always proceed with documented
  assumptions when inputs are incomplete.
---

# Generic IR Enrichment (shared rules)

## Primary goal

Safely patch `src/output_workflow/_internal/core_design_model.json` with technology-blind cross-cutting content — preserving existing fields, documenting every gap-filled decision, and never blocking because an upload is missing.

## Success criteria

- [ ] Only owned IR sections modified; all other fields preserved verbatim
- [ ] `meta.v` patch incremented
- [ ] Every inferred value has `meta.assumptions[]` entry
- [ ] No stack-specific artifact files created
- [ ] Empty sections are valid when requirements truly omit that domain (with assumption explaining why)

## Section ownership

| Skill | IR sections |
|-------|-------------|
| design-generic-security | `security` |
| design-generic-data | `data`, `dataEntities`, relationships |
| design-generic-messaging | `messaging`, links to `integrations` |

## Workflow modes

**CREATE:** build from whatever requirements exist in workspace and context.

**UPDATE:** read IR first; patch only named changes; append `src/output_workflow/_internal/design_changelog.md`.

## Handling missing or incomplete inputs

Enrichment must **still run** when epic/ADR are partial:

1. Read existing IR — prior steps may already have capabilities, APIs, actors.
2. Infer cross-cutting needs from `apiOperations`, `capabilities`, category, and ADR fragments.
3. Apply **category baselines** when silent:
   - **backend/fullstack API:** token auth + RBAC skeleton, relational data entity stubs, optional async only if integrations imply events
   - **frontend/mobile:** client-relevant auth flows only; defer server DB detail
   - **integration:** messaging/event patterns prioritized over presentation
4. Document each baseline in `meta.assumptions` with `needsReview: true` when confidence is medium or low.
5. Use `clarify` only when choosing between mutually exclusive architectures (e.g. event-driven vs sync-only).

**Never:** invent compliance mandates, named vendors, or production secrets.

## Do not

- Emit `Security.json`, `Database.json`, `MessageDesign.json` — adapter maps from IR
- Contradict ADR — ADR wins; document conflicts in `meta.assumptions`
- Wipe existing IR content during enrichment

## Completion gate

IR written, `meta.v` bumped, assumptions complete for every non-input-backed field.
