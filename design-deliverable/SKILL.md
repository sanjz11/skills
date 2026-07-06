---
name: design-deliverable
description: >-
  Produce the 1-2 shareable design deliverables from IR and adapter workspace
  artifacts. Use in Design Deliverable Agent — final step before validation.
---

# Design Deliverable Skill

## Purpose

Create **only** the files stakeholders share externally (max 2 files).

## Mandatory outputs

1. **`src/output_workflow/consolidated_design.md`** — complete human-readable design:
   - Executive summary
   - Technology stack (from technology_context)
   - Requirements & capabilities
   - Business rules & legacy logic migration (full detail, pseudo-code from adapter)
   - Architecture & layers (generic + adapted)
   - API contract summary (+ embed or appendix OpenAPI paths when backend)
   - Security, data, messaging (only sections present in IR)
   - NFR, error handling, observability
   - Mermaid diagrams inline

2. **`src/output_workflow/consolidated_design.json`** — compact machine-readable bundle:
   - `technology` (from technology_context)
   - `coreModelSummary` (key IR fields — not full duplicate if huge)
   - `adaptedDesign` (merged adapter JSON sections produced)
   - `apiSummary` when applicable
   - `meta.ts`, `meta.v`

## Rules

1. Read from `src/output_workflow/_internal/` (IR, context, adapter outputs).
2. **Do not** require stakeholders to receive Application/, Presentation/, etc. — fold into consolidated files.
3. Keep `_internal/` as workspace; deliverables live at `src/output_workflow/` root.
4. If OpenAPI exists in `_internal/`, include path/method summary in .md; optional `openapi` key in .json (not a third file).
5. UPDATE mode: patch only sections named in change_requirements.

## Do not

- Create more than 2 shareable files at workflow root
- Omit legacy migration detail from consolidated_design.md
