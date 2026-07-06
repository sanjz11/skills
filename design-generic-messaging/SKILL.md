---
name: design-generic-messaging
description: >-
  Enrich IR with technology-agnostic messaging. Primary goal: model events and
  reliability when async is implied — or explicitly document sync-only with
  assumption when epic is silent.
---

# Generic Messaging (IR enrichment)

## Primary goal

Populate `messaging` in the IR with events, commands, patterns, and reliability — when epic/ADR require async communication — **or** explicitly record a sync-only assumption when no async signals exist. Never leave `messaging` ambiguous.

## Success criteria

- [ ] `messaging` is either fully specified **or** empty with `meta.assumptions` explaining sync-only decision
- [ ] Events/commands link to `boundedContexts` and `integrations` when present
- [ ] Delivery semantics and idempotency documented when events exist
- [ ] No broker vendor names or configs
- [ ] IR updated; `meta.v` bumped; no `MessageDesign.json` emitted

**IR file:** `src/output_workflow/_internal/core_design_model.json`

## When to use

Load when epic/ADR mention events, queues, pub/sub, integration flows, or saga/choreography — **or** when `integrations[]` imply async handoff.

## Domain responsibilities

### Architecture
Pub/sub, point-to-point, request/reply, event sourcing, CQRS (when ADR requires).

### Schemas
Events, commands, headers, versioning, compatibility.

### Producers / consumers
Groups, partitioning, ordering requirements.

### Reliability
Delivery guarantees, idempotency, DLQ, retry.

### Observability
Metrics, tracing, correlation IDs.

## Handling missing or incomplete inputs

| Situation | What to do |
|-----------|------------|
| Epic silent on messaging | If sync REST only → empty `messaging` + assumption "sync-only architecture" |
| Integration list mentions external system | Add integration events for handoff points; `[REVIEW]` |
| ADR says "event-driven" | Full event catalog from capabilities and bounded contexts |
| No event schemas in epic | Define minimal schemas from entity lifecycle (Created/Updated/Deleted) |
| Unclear sync vs async | Default sync; one `clarify` if integration count > 3 |

**Do not** invent event storms or topics without business trigger in capabilities/APIs.

## What you can do

- Design full event/command models with reliability and monitoring
- Link messaging to `integrations[]`
- Document correlation and tracing requirements

## What you cannot do

- Kafka/RabbitMQ/Service Bus specific configs
- Emit `MessageDesign.json`
- Design messaging when system is purely request/response without documenting why

## IR output structure

```json
{
  "messaging": {
    "architecture": { "patterns": [], "eventSourcing": {}, "routing": {} },
    "events": [{ "id": "EVT-001", "name": "", "schema": {}, "version": "1.0" }],
    "commands": [],
    "patterns": { "producers": [], "consumers": [], "partitioning": {} },
    "reliability": { "delivery": "at-least-once", "idempotency": {}, "dlq": {}, "retry": {} },
    "monitoring": { "metrics": [], "tracing": {} }
  }
}
```

## Completion gate

Messaging either populated with traceable events or explicitly sync-only with assumption.
