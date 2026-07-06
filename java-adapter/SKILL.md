---
name: java-adapter
description: >-
  Map IR to Java / Spring Boot internal artifacts. Primary goal: always produce
  stack-specific files from IR + technology_context. Proceed with assumptions
  when IR sections are empty. Load when technology_context.profile.skill is java-adapter.
---

# Java / Spring Boot Adapter

## Primary goal

Transform `core_design_model.json` into **complete internal Java / Spring Boot design artifacts** under `_internal/` — using `technology_context.profile.skill`, `layerMapping`, and `artifacts` — **even when IR sections are partial or epic uploads were missing**.

## Success criteria

- [ ] Read `src/output_workflow/_internal/core_design_model.json` and `src/output_workflow/_internal/technology_context.json` first
- [ ] Loaded skill matches `technology_context.profile.skill` (java-adapter)
- [ ] Required profile artifacts written under `src/output_workflow/_internal/`
- [ ] `legacyLogicMigration` rendered as **Java** pseudo-code in `.md` files with `// Legacy line N:` traceability
- [ ] Each JSON artifact includes `meta.techProfile`, `meta.irVersion`, `meta.adapter`
- [ ] Empty IR sections: skip file OR write minimal stub with `meta.assumptions` — never fail silently
- [ ] No consolidated deliverables at workflow root

## Inputs

- `src/output_workflow/_internal/core_design_model.json`
- `src/output_workflow/_internal/technology_context.json`
- `config/technology-registry.json` (reference)

## Layer → artifact mapping

| IR layer | Java / Spring Boot artifacts |
|----------|---------------------------|
| presentation | @RestController, DTO, @Valid request objects |
| business | @Service, domain logic, @Transactional boundaries |
| data | @Repository, JPA Entity, Mapper |
| integration | @FeignClient, @KafkaListener |
| configuration | application.yml design, @Configuration |

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

Stack-specific: If IR.security empty → minimal Security.json stub + assumption. If IR.messaging empty → skip MessageDesign.json.

Use `clarify` only if `technology_context.profileId` conflicts with registry or IR stack hints.

## Legacy migration

For each `legacyLogicMigration[]` entry: **Java** pseudo-code in Design/Presentation `.md` with traceability comments.

## Naming

camelCase classes; PascalCase types; packages by bounded context.

## Reference patterns (load via skills tool)

Registry profile `java-spring` lists:

- `references/controller-pattern.md`
- `references/repository-pattern.md`
- `references/Design-json-shape.json`

Load each reference file from this adapter skill folder. **Render IR into reference JSON/Markdown shapes** — tune stack output by editing references in the skills repository, not orchestration prompts.

## Best practices (Java / Spring Boot)

- Constructor injection only; avoid field injection
- OpenAPI 3.1 generated from IR apiOperations
- Transactional boundaries on service layer not controllers

## Tuning strategy

| Change | Edit here | Do not edit |
|--------|-----------|-------------|
| Output file shape | `references/*-shape.json` | IR schema |
| Layer mapping labels | `technology-registry.json` → layerMapping | generic skills |
| Stack conventions | This SKILL.md + references | step orchestration prompts |
| ADR defaults | registry `adrDefaults` | adr-blueprint keys list |

Future phases: `implementationSkill` + `testingSkill` in registry (see `config/agent-contracts.json`).

## Do not

- Write `consolidated_design.md` or `consolidated_design.json`
- Load a different adapter skill than `profile.skill`
- Invent APIs or rules absent from IR (infer only with `meta.assumptions`)
- Use a stack that does not match `technology_context.profileId` (java-spring)

## Completion gate

All applicable outputs exist under `_internal/`; assumptions recorded for every inferred design element.
