# RAML design contract (MuleSoft)

Generate `Integration/integration-apis.raml` from IR `apiOperations[]` and Experience API layer in `IntegrationDesign.json`.

## Rules

1. RAML 1.0 — `#%RAML 1.0` header, title, version, baseUri placeholder.
2. Map synchronous IR `apiOperations` → Experience API resources (e.g. `/health`, business resources).
3. Types from IR request/response shapes; error responses from IR `errorModel`.
4. Cross-reference in `IntegrationDesign.json` via `ramlRef: integration-apis.raml`.
5. Process/System API contracts may be separate RAML fragments referenced from IntegrationDesign when IR has data/system layers.

## Async companion

When IR `messaging.events` or `messaging.commands` non-empty: also produce `Messaging/asyncapi.yaml` (AsyncAPI 2.6+).
