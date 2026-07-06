---
name: design-deliverable
description: >-
  Produce the shareable design deliverables from IR and internal adapted
  artifacts. Load when consolidating into stakeholder-facing outputs (max 2
  files at workflow root).
---

# Design Deliverable

## Purpose

Create **only** the files stakeholders share externally (maximum 2 files at workflow root).

## Mandatory outputs

1. **`src/output_workflow/consolidated_design.md`** — complete human-readable design:
   - Executive summary
   - Technology stack (from technology_context)
   - Requirements and capabilities
   - Business rules and legacy logic migration (full pseudo-code from adapter)
   - Architecture and layers (generic + adapted)
   - API contract summary (+ path/method summary when OpenAPI exists internally)
   - Security, data, messaging (only sections present in IR)
   - NFR, error handling, observability
   - Mermaid diagrams inline

2. **`src/output_workflow/consolidated_design.json`** — compact machine-readable bundle:
   - `technology` (from technology_context)
   - `coreModelSummary` (key IR fields)
   - `adaptedDesign` (merged internal adapter JSON sections)
   - `apiSummary` when applicable
   - `meta.ts`, `meta.v`

## Rules

1. Read from `src/output_workflow/_internal/` (IR, context, adapter outputs).
2. **Do not** require stakeholders to open internal subfolders — fold essential content into the two files.
3. Keep `_internal/` as workspace-only; deliverables at `src/output_workflow/` root.
4. If OpenAPI exists in `_internal/`, summarize in `.md`; optional `openapi` key in `.json` (not a third file).
5. **UPDATE mode:** patch only sections named in change requirements.

## Validation alignment

Quality checks expect:
- Both deliverables exist and align with IR
- Legacy migration in IR appears in `.md`
- At most two shareable files at workflow root

## Do not

- Create more than 2 shareable files at workflow root
- Omit legacy migration detail from consolidated_design.md
- Contradict IR — IR wins; document conflicts in `meta`
