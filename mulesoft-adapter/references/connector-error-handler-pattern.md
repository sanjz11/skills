# MuleSoft — connectors, error handlers, and API-led mapping

## API-led layering (mandatory)

| IR layer | MuleSoft artifact | Responsibility |
|----------|-------------------|----------------|
| presentation / apiOperations | **Experience API** (RAML) | External contract, policies, consumer-facing errors |
| business / workflows | **Process API** flows + subflows | Orchestration, branching, compensation |
| data / entities | **System API** | Canonical entity access, no orchestration |
| integration | **Connectors** | Object store, SFTP, VM/JMS, HTTP requester — name per IR integration |
| configuration | **Properties + API Manager policies** | Client ID enforcement, SLA, spike control when NFR requires |

## Every Process flow must declare

1. **Main flow** — trigger (listener/scheduler), correlation id propagation
2. **Subflows** — one per IR workflow step cluster
3. **Error handler** — `on-error-propagate` vs `on-error-continue` per error catalog row
4. **Retry policy** — only for IR-approved transient errors (`retryable: true`)
5. **Transform Message** — DataWeave 2.0 with input/output MIME types
6. **Until-successful / Try** scopes where IR reliability section specifies retry

## Connector mapping from IR

| IR integration kind | Typical connector | Config keys |
|---------------------|-------------------|-------------|
| message-channel | Anypoint MQ / JMS / VM | destination, ack mode |
| file-storage | SFTP / File / S3 | path, filename pattern |
| file-transfer | SFTP outbound | destination path, rename strategy |
| operational-check | HTTP Request + health subflow | url, timeout |

## Error catalog → Mule error type

Map each IR `errors[]` entry to:

- `errorType` namespace and identifier
- `retryable` flag
- `archiveRoute` (error folder) when file-centric
- Logging category and correlation fields (never log secrets)

## RAML requirements

- `/health` as synchronous resource when IR includes health apiOperation
- Async commands documented in `Messaging/MessageDesign.json` and referenced from Process API — not as CRUD RAML resources unless IR explicitly requires

## DataWeave pseudo-code in IntegrationDesign.md

Use DataWeave 2.0 syntax in legacy migration blocks:

```dataweave
%dw 2.0
output application/xml
---
// trace: BR-005
{
  catalog: payload.products map (p) -> { ... }
}
```

## MUnit

List one MUnit test per Process flow covering: happy path, selective retry, non-retryable failure, empty-input edge case when IR business rule requires it.
