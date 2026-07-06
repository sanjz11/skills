---
name: design-deliverable
description: >-
  Produce consolidated stakeholder deliverables. Primary goal: always output
  two self-contained files from whatever internal artifacts exist. Fill gaps
  from IR with documented assumptions.
---

# Design Deliverable

## Primary goal

Always produce **exactly two** shareable files at `src/output_workflow/` root — `consolidated_design.md` and `consolidated_design.json` — self-contained and consistent, built from whatever exists in `_internal/`, without requiring stakeholders to open subfolders.

## Success criteria

- [ ] `consolidated_design.md` exists — complete human-readable design
- [ ] `consolidated_design.json` exists — valid JSON aligned with `.md`
- [ ] No third shareable file at workflow root
- [ ] Stack, APIs, business rules, legacy migration (if in IR) appear in `.md`
- [ ] Sections missing from internal artifacts are either omitted with note in `meta.assumptions` or filled from IR with `[REVIEW]` flag
- [ ] `.md` and `.json` do not contradict IR (IR wins on conflict)

## Mandatory outputs

### 1. `consolidated_design.md`

Executive summary, technology stack, requirements, business rules, legacy migration (full pseudo-code from adapter), architecture, APIs, security/data/messaging (if in IR), NFR, Mermaid diagrams.

### 2. `consolidated_design.json`

`technology`, `coreModelSummary`, `adaptedDesign`, `apiSummary`, `meta`.

## Procedure

1. Read all of `src/output_workflow/_internal/`.
2. Merge IR + adapted artifacts; IR is semantic source of truth.
3. Fold internal folder content into the two files.
4. UPDATE: patch only sections named in change requirements.

## Handling missing or incomplete inputs

You must still ship both deliverables.

| Situation | What to do |
|-----------|------------|
| Adapter artifacts missing | Consolidate from IR + `technology_context` only; note gap in `meta.assumptions` |
| IR section empty | Omit section or include "Not in scope per assumptions" with reference to `meta.assumptions` |
| OpenAPI missing | Build API summary from IR `apiOperations` |
| Legacy pseudo-code missing | Include IR excerpts + note adapter gap; flag `[REVIEW]` |
| Partial internal tree | Include what exists; never reference paths stakeholders cannot access |

Use `clarify` only if deliverable purpose (audience, compliance packaging) is unknown and would change document structure materially.

## Do not

- Create more than 2 shareable root files
- Omit legacy detail that exists in internal `.md` files
- Say "see `_internal/` folder" — inline essential content

## Completion gate

Two root files exist, cross-validated, assumptions documented for any synthesized content.
