---
name: python-adapter
description: >-
  Map IR to Python / FastAPI internal artifacts. Primary goal: always produce
  stack-specific files from IR + technology_context. Proceed with assumptions
  when IR sections are empty. Load when technology_context.profile.skill is python-adapter.
---

# Python / FastAPI Adapter

## Primary goal

Transform `core_design_model.json` into **complete internal Python / FastAPI design artifacts** under `_internal/` — using `technology_context.profile.skill`, `layerMapping`, and `artifacts` — **even when IR sections are partial or epic uploads were missing**.

## Success criteria

- [ ] Read `src/output_workflow/_internal/core_design_model.json` and `src/output_workflow/_internal/technology_context.json` first
- [ ] Loaded skill matches `technology_context.profile.skill` (python-adapter)
- [ ] Required profile artifacts written under `src/output_workflow/_internal/`
- [ ] `legacyLogicMigration` rendered as **Python** pseudo-code in `.md` files with `# Legacy line N:` traceability
- [ ] Each JSON artifact includes `meta.techProfile`, `meta.irVersion`, `meta.adapter`
- [ ] Empty IR sections: skip file OR write minimal stub with `meta.assumptions` — never fail silently
- [ ] No consolidated deliverables at workflow root

## Inputs

- `src/output_workflow/_internal/core_design_model.json`
- `src/output_workflow/_internal/technology_context.json`
- `config/technology-registry.json` (reference)

## Layer → artifact mapping

| IR layer | Python / FastAPI artifacts |
|----------|---------------------------|
| presentation | APIRouter, Pydantic schema |
| business | Service class |
| data | Repository, ORM model |
| integration | HTTP client, message consumer |
| configuration | Settings module |

## Outputs

- `src/output_workflow/_internal/Application/Design.json`
- `src/output_workflow/_internal/Application/Design.md`
- `src/output_workflow/_internal/Application/openapi.yaml`
- `src/output_workflow/_internal/Security/Security.json`
- `src/output_workflow/_internal/Database/Database.json`
- `src/output_workflow/_internal/Messaging/MessageDesign.json`

## Procedure

1. Read IR and technology context.
2. Load this skill via skills tool.
3. Map each non-empty IR layer per `layerMapping`.
4. Generate openapi/contracts from `apiOperations` when profile includes them.
5. Map `security`, `data`, `messaging` IR sections to folder JSONs when non-empty and category-relevant.
6. Write all artifacts; bump `meta.v` on updates.

## Handling missing or incomplete inputs

You must still produce adapter artifacts. IR is the source of truth.

| Situation | What to do |
|-----------|------------|
| IR section empty | Skip that artifact file OR emit minimal stub documenting omission in artifact `meta.assumptions` |
| apiOperations incomplete | Complete from capabilities using RESTful conventions; flag `[REVIEW]` in meta |
| No legacyLogicMigration | Omit pseudo-code blocks — do not invent legacy |
| technology_context partial | Re-read registry; never guess a different profile |
| Epic never uploaded | Rely entirely on IR + technology_context |

Stack-specific: snake_case throughout; derive routers from apiOperations if epic lacks module layout.

Use `clarify` only if `technology_context.profileId` conflicts with registry or IR stack hints.

## Legacy migration

For each `legacyLogicMigration[]` entry: **Python** pseudo-code in Design/Presentation `.md` with traceability comments.

## Naming

snake_case modules/functions; PascalCase models.

## Do not

- Write `consolidated_design.md` or `consolidated_design.json`
- Load a different adapter skill than `profile.skill`
- Invent APIs or rules absent from IR (infer only with `meta.assumptions`)
- Use a stack that does not match `technology_context.profileId` (python)

## Completion gate

All applicable outputs exist under `_internal/`; assumptions recorded for every inferred design element.
