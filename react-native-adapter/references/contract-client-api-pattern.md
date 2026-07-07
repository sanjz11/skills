# Client API contract (frontend / mobile profiles)

Generate `Application/client-api.openapi.yaml` — OpenAPI 3.1 describing **APIs the client consumes** (not server implementation).

## Rules

1. Derive from IR `apiOperations[]` and `Application/Design.json` API client bindings.
2. Document auth headers, error handling, and correlation IDs the client must send.
3. Cross-reference in `Application/Design.json` via `clientApiContractRef: client-api.openapi.yaml`.
4. When epic is UI-only with no backend APIs: omit file and note in `meta.assumptions`.

Optional: `Application/graphql.schema.graphql` when ADR `api_style` is graphql (document in assumptions if not generated).
