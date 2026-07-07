---
name: java-adapter
description: >-
  Map IR to Java / Spring Boot internal artifacts with full DDD domain model.
  Primary goal: always produce stack-specific files from IR + technology_context.
  Proceed with assumptions when IR sections are empty. Load when
  technology_context.profile.skill is java-adapter.
---

# Java / Spring Boot Adapter

## Primary goal

Transform `core_design_model.json` into **complete internal Java / Spring Boot design artifacts** under `_internal/` — including **DDD bounded contexts, aggregates, entities, value objects, domain events, and repositories** — using `technology_context.profile.skill`, `layerMapping`, and `artifacts`.

## Bootstrap fallback (references)

If `references/*` files are not visible next to `SKILL.md`, fetch from:

`https://raw.githubusercontent.com/sanjz11/skills/main/java-adapter/references/<filename>`

Use `command_line` (`curl -fsSL`) before proceeding without templates.

## Success criteria

- [ ] Read `src/output_workflow/_internal/core_design_model.json` and `technology_context.json` first
- [ ] `technology_context` has valid `profile.skill` (not error-only) — if error, re-bootstrap registry per `design-technology-base` before adapting
- [ ] **Domain layer complete:** `Domain/DomainDesign.json` maps every IR `boundedContexts[]` entry to aggregates, entities, value objects, domain events
- [ ] Application layer maps controllers/services/repositories to IR capabilities and apiOperations
- [ ] `legacyLogicMigration` rendered as **Java** pseudo-code in `.md` files with `// Legacy line N:` traceability
- [ ] Each JSON artifact includes `meta.techProfile`, `meta.irVersion`, `meta.adapter`
- [ ] No consolidated deliverables at workflow root

## Inputs

- `src/output_workflow/_internal/core_design_model.json`
- `src/output_workflow/_internal/technology_context.json`
- `src/output_workflow/_internal/_config/technology-registry.json`

## Layer → artifact mapping

| IR layer | Java / Spring Boot artifacts |
|----------|---------------------------|
| boundedContexts | `@AggregateRoot`, domain entities, value objects, domain events |
| presentation | `@RestController`, DTO, `@Valid` request objects |
| business | `@Service`, application services, `@Transactional` boundaries |
| data | `@Repository`, JPA Entity, Mapper, read-model projections |
| integration | `@FeignClient`, `@KafkaListener`, outbox publisher |
| configuration | `application.yml` design, `@Configuration`, feature flags |

## Outputs (all required when IR section non-empty)

- `src/output_workflow/_internal/Domain/DomainDesign.json`
- `src/output_workflow/_internal/Domain/DomainDesign.md`
- `src/output_workflow/_internal/Application/Design.json`
- `src/output_workflow/_internal/Application/Design.md`
- `src/output_workflow/_internal/Application/openapi.yaml` — **mandatory** OpenAPI 3.1 from IR `apiOperations`
- `src/output_workflow/_internal/Messaging/asyncapi.yaml` — when IR `messaging.events` or `messaging.commands` non-empty
- `src/output_workflow/_internal/Messaging/MessageDesign.json` (when IR.messaging non-empty)

## Design contracts procedure (mandatory)

1. Load `references/contract-openapi-pattern.md`; generate `Application/openapi.yaml` — every IR `apiOperations[]` entry as path+method; errors as responses.
2. Set `Application/Design.json` → `openapiRef: openapi.yaml`.
3. When IR messaging present: load `references/contract-asyncapi-pattern.md`; generate `Messaging/asyncapi.yaml`.
4. Do not invent operations not in IR; document gaps in artifact `meta.assumptions`.

## DDD procedure (mandatory)

1. For each IR `boundedContexts[]`: define package, aggregates (with root), entities, value objects, invariants.
2. Map IR `capabilities` and `workflows` to **application services**; cite `businessRuleIds` on enforcing methods.
3. Map IR `dataEntities` to JPA entities inside correct aggregate or read-model package.
4. Map IR `domainEvents` / `messaging.events` to Spring events or outbox records.
5. Controllers expose only `apiOperations` — delegate to application services, never embed business rules.
6. Write `DomainDesign.md` with aggregate diagram (Mermaid) and aggregate boundary table.

## Reference patterns (load or fetch)

- `references/ddd-pattern.md`
- `references/controller-pattern.md`
- `references/repository-pattern.md`
- `references/Design-json-shape.json`

Render IR into reference JSON/Markdown shapes.

## Best practices (Java / Spring Boot)

- Constructor injection only; avoid field injection
- OpenAPI 3.1 generated from IR apiOperations
- Transactional boundaries on application service layer, not controllers
- One repository per aggregate root
- Reference other aggregates by ID only
- Map every `businessRules[]` entry to a domain or application enforcement point

## Handling missing inputs

| Situation | What to do |
|-----------|------------|
| IR `boundedContexts` empty | Infer one bounded context per major capability cluster; document in `meta.assumptions` |
| IR section empty | Skip file OR minimal stub with `meta.assumptions` |
| technology_context.error | Stop and re-bootstrap — do not emit generic layered design |

## Do not

- Write consolidated deliverables
- Omit Domain layer — DDD is mandatory for Java profile
- Invent APIs absent from IR

## Completion gate

`Domain/`, `Application/`, and applicable `Security/`/`Database/`/`Messaging/` folders populated; DDD traceability to IR ids complete.
