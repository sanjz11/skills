---
name: python-adapter
description: >-
  Map Core Design Model IR to Python / FastAPI design artifacts. Use only in
  Technology Adapter Agent when technology_context.profileId is python.
---

# Python / FastAPI Adapter

Read `technology_context.json` + `core_design_model.json`. Map generic layers using `profile.layerMapping`.

## Layer → artifact mapping

| IR layer | Python / FastAPI artifacts |
|----------|----------------------------|
| presentation | APIRouter, Pydantic schema, request validator |
| business | Service class, domain logic |
| data | Repository, SQLAlchemy/ORM model, mapper |
| integration | HTTP client, message consumer |
| configuration | Settings module, pydantic-settings |

## Outputs

- `src/output_workflow/Application/Design.json` — stack-specific layered design
- `src/output_workflow/Application/Design.md` — includes legacy migration pseudo-code in **Python** with traceability
- `src/output_workflow/Application/openapi.yaml` — OpenAPI 3.1 from `apiOperations`
- `src/output_workflow/Security/Security.json` — from IR.security
- `src/output_workflow/Database/Database.json` — from IR.data
- `src/output_workflow/Messaging/MessageDesign.json` — from IR.messaging (if non-empty)

## Legacy migration

For each `legacyLogicMigration[]` entry: Python pseudo-code in Design.md with `# Legacy line N:` comments.

## Naming

snake_case for modules, functions, and variables; PascalCase for Pydantic models and service classes.
