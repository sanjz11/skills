# OpenAPI design contract (backend profiles)

Generate `Application/openapi.yaml` from IR `apiOperations[]` — OpenAPI **3.1**.

## Rules

1. One path entry per IR `apiOperations[]` item; preserve `operationId`, method, path.
2. Request/response schemas from IR (conceptual types only in IR — expand to JSON Schema in YAML).
3. Document all IR `errors[]` / operation errors as `responses` with consistent error envelope.
4. Security schemes from IR `security` (bearer, apiKey, oauth2 — generic names only).
5. Include `info`, `servers` (placeholder env URLs), `tags` aligned with IR capabilities.
6. Cross-reference in `Application/Design.json` via `openapiRef: openapi.yaml`.

## When IR has no apiOperations

Emit minimal health/readiness contract if epic implies APIs; else document in `meta.assumptions` with `[REVIEW]`.
