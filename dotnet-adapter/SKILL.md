---
name: dotnet-adapter
description: >-
  Map Core Design Model IR to .NET/ASP.NET Core internal artifacts. Load when technology_context.profile.skill is dotnet-adapter.

# .NET / ASP.NET Core Adapter

Read `technology_context.json` + `core_design_model.json`. Map generic layers using `profile.layerMapping`.

## Layer → artifact mapping

| IR layer | .NET artifacts |
|----------|----------------|
| presentation | ApiController, DTO, FluentValidation |
| business | Service, MediatR handler, domain logic |
| data | Repository, Entity, EF Core DbContext mapping |
| integration | HttpClient, message consumer |
| configuration | appsettings.json design, IOptions |

## Outputs

- `src/output_workflow/_internal/Application/Design.json` — stack-specific layered design
- `src/output_workflow/_internal/Application/Design.md` — includes legacy migration pseudo-code in **C#** with traceability
- `src/output_workflow/_internal/Application/openapi.yaml` — OpenAPI 3.1 from `apiOperations`
- `src/output_workflow/_internal/Security/Security.json` — from IR.security
- `src/output_workflow/_internal/Database/Database.json` — from IR.data
- `src/output_workflow/_internal/Messaging/MessageDesign.json` — from IR.messaging (if non-empty)

## Legacy migration

For each `legacyLogicMigration[]` entry: C# pseudo-code in Design.md with `// Legacy line N:` comments.

## Naming

PascalCase for classes, methods, and properties; camelCase for local variables and JSON serialization defaults.
