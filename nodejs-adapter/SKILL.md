---
name: nodejs-adapter
description: >-
  Map Core Design Model IR to Node.js / NestJS design artifacts. Use only in
  Technology Adapter Agent when technology_context.profileId is nodejs.
---

# Node.js / NestJS Adapter

Read `technology_context.json` + `core_design_model.json`. Map generic layers using `profile.layerMapping`.

## Layer → artifact mapping

| IR layer | Node.js / NestJS artifacts |
|----------|----------------------------|
| presentation | Route, Controller, DTO class, validation pipe |
| business | Injectable service, domain logic |
| data | Repository, TypeORM/Prisma entity, mapper |
| integration | HTTP client module, message consumer |
| configuration | ConfigModule, env schema |

## Outputs

- `src/output_workflow/Application/Design.json` — stack-specific layered design
- `src/output_workflow/Application/Design.md` — includes legacy migration pseudo-code in **TypeScript** with traceability
- `src/output_workflow/Application/openapi.yaml` — OpenAPI 3.1 from `apiOperations`
- `src/output_workflow/Security/Security.json` — from IR.security
- `src/output_workflow/Database/Database.json` — from IR.data
- `src/output_workflow/Messaging/MessageDesign.json` — from IR.messaging (if non-empty)

## Legacy migration

For each `legacyLogicMigration[]` entry: TypeScript pseudo-code in Design.md with `// Legacy line N:` comments.

## Naming

camelCase for functions/variables; PascalCase for classes and DTOs. NestJS module boundaries by bounded context.
