---
name: java-adapter
description: >-
  Map Core Design Model IR to Java/Spring Boot internal artifacts. Load when technology_context.profile.skill is java-adapter.

# Java / Spring Boot Adapter

Read `technology_context.json` + `core_design_model.json`. Map generic layers using `profile.layerMapping`.

## Layer → artifact mapping

| IR layer | Java artifacts |
|----------|----------------|
| presentation | @RestController, DTO, @Valid request objects |
| business | @Service, domain logic, @Transactional boundaries |
| data | @Repository, JPA Entity, Mapper |
| integration | @FeignClient, @KafkaListener |
| configuration | application.yml design, @Configuration |

## Outputs

- `src/output_workflow/_internal/Application/Design.json` — stack-specific design
- `src/output_workflow/_internal/Application/Design.md` — includes legacy migration pseudo-code in **Java** with traceability
- `src/output_workflow/_internal/Application/openapi.yaml` — OpenAPI 3.1 from `apiOperations`
- `src/output_workflow/_internal/Security/Security.json` — from IR.security
- `src/output_workflow/_internal/Database/Database.json` — from IR.data
- `src/output_workflow/_internal/Messaging/MessageDesign.json` — from IR.messaging (if non-empty)

## Legacy migration

For each `legacyLogicMigration[]` entry: Java pseudo-code in Design.md with `// Legacy line N:` comments.

## Naming

camelCase classes; PascalCase types; packages by bounded context.
