---
name: design-generic-core
description: >-
  Build the technology-independent Core Design Model (IR). Primary goal: one
  complete stack-agnostic design model from available inputs. Proceed even when
  uploads are partial — document assumptions.
---

# Generic Core Design

## Primary goal

Produce a **complete, technology-independent** Core Design Model — `core_design_model.json` and `core_design_model.md` — that captures capabilities, APIs, business rules, data, security, messaging, NFRs, and legacy intent from **whatever inputs are available**, without framework or vendor names.

## Success criteria

- [ ] `src/output_workflow/_internal/core_design_model.json` exists and matches schema intent
- [ ] `src/output_workflow/_internal/core_design_model.md` exists with readable structure and Mermaid where useful
- [ ] Every generic layer (`presentation`, `business`, `data`, `integration`, `configuration`) is populated or explicitly empty with reason in `meta.assumptions`
- [ ] `apiOperations` covers every API implied by inputs (or documents assumed APIs with `[REVIEW]`)
- [ ] `businessRules` use BR-IDs; `legacyLogicMigration` has excerpts without pseudo-code
- [ ] Cross-cutting skills applied when relevant: security, data, messaging
- [ ] `meta.v` set; `meta.assumptions` records every gap-filled decision

## When to use

Load when creating or updating the canonical IR **before** stack-specific adaptation.

Read `config/core-design-model-schema.json` first.

## Procedure

1. Read all available inputs (epic, ADR, patterns, domain model) and existing workspace IR.
2. Load `design-generic-security`, `design-generic-data`, `design-generic-messaging` when requirements or category imply those domains.
3. Extract capabilities, actors, flows, business rules, APIs, data, security, messaging, NFR, errors, deployment.
4. Assign elements to generic layers.
5. Write JSON (compact) and MD (readable).
6. Bump `meta.v` on updates.

## Rules (mandatory)

1. **Zero technology names** in IR
2. Business rules: BR-ID, business language — no pseudo-code in IR
3. Legacy: `legacyLogicMigration[]` with `sourceLanguage`, `sourceExcerpt`, `businessIntent` only
4. Conceptual data types only (string, uuid, datetime, etc.)
5. Preserve existing IR fields on patch; copy verbatim what change requirements do not name

## Handling missing or incomplete inputs

You must still deliver a complete IR. Missing uploads, empty epic sections, or partial ADR are **not** reasons to stop.

| Situation | What to do |
|-----------|------------|
| Epic upload missing | Infer scope from ADR, domain model, stack category, and chat context |
| No APIs listed | Derive minimal REST surface from capabilities and entities; flag `[REVIEW]` |
| No business rules | Extract implicit rules from flows and legacy excerpts |
| No legacy section | Leave `legacyLogicMigration` empty — do not invent legacy |
| No security/messaging in epic | Apply category baseline via enrichment skills OR leave empty with assumption explaining omission |
| Critical ambiguity | One `clarify` question; then proceed with documented assumption |

Record every inference in `meta.assumptions[]` with `id`, `topic`, `assumption`, `reason`, `confidence`, `impact`, `needsReview`.

## Layer assignment

| Concept | Layer |
|---------|-------|
| User-facing interaction, API surface | presentation |
| Business rules, workflows | business |
| Persistence, entities | data |
| External systems, events | integration |
| Env, feature flags | configuration |

## Do not

- Emit openapi.yaml, Design.json, consolidated deliverables, or stack artifacts
- Hardcode a programming language or framework in IR
- Block completion because an optional upload was not provided

## Completion gate

Before finishing, confirm all success criteria pass. Any gap-filled section must have a matching `meta.assumptions` entry.
