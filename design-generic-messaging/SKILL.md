---
name: design-generic-messaging
description: >-
  Enrich the Core Design Model IR with technology-agnostic messaging and
  event-driven design. Load during IR build when epic/ADR require async,
  events, commands, or pub/sub. No broker vendor names or configs.
---

# Generic Messaging (IR enrichment)

## When to use

Load this skill when building or patching `core_design_model.json` and requirements imply asynchronous communication, events, commands, or message-driven integration.

**Output target:** enrich `messaging` and related `integrations` — do **not** create `MessageDesign.json` or broker-specific configs (the technology adapter maps IR later).

**IR file:** `src/output_workflow/_internal/core_design_model.json`

If epic/ADR have **no** async or event requirements, leave `messaging` empty — do not invent.

---

## Primary goal

Produce technology-agnostic messaging design in the IR — architecture patterns, message schemas, producer/consumer patterns, reliability guarantees, and observability — implementable on any messaging platform.

---

## Responsibilities

### 1. Messaging architecture
- Broker patterns: topics, queues, exchanges (generic)
- Message patterns: pub/sub, point-to-point, request/reply
- Event sourcing and CQRS where ADR/epic require
- Routing and filtering strategies
- **No** Kafka/RabbitMQ/Service Bus-specific settings

### 2. Message schemas
- Domain events and integration events
- Commands and queries
- Headers and metadata (correlation, trace, routing)
- Schema versioning and backward compatibility
- Serialization format requirements (JSON Schema concepts)
- **No** Avro/Protobuf vendor bindings

### 3. Producer / consumer patterns
- Producer and consumer group concepts
- Partitioning and ordering requirements
- Scaling and load distribution (generic)
- Consumer lifecycle patterns

### 4. Reliability patterns
- Delivery semantics: at-least-once, at-most-once, exactly-once
- Idempotency and deduplication
- Dead letter queue (DLQ) strategies
- Retry policies and exponential backoff
- **No** platform-specific retry configs

### 5. Monitoring and observability
- Throughput, latency, error rate metrics
- Tracing and correlation (correlation IDs)
- Alerting thresholds (generic)
- Message lifecycle documentation

---

## What you can do

- Design pub/sub, point-to-point, request/reply, event sourcing, CQRS (when required)
- Define event/command schemas with versioning rules
- Specify producer/consumer, partitioning, and ordering requirements
- Document delivery guarantees, idempotency, DLQ, and retry patterns
- Add correlation and tracing requirements to `observability` when needed
- Populate IR `messaging` with compact JSON
- Link events to `boundedContexts`, `apiOperations`, and `integrations`
- Preserve existing IR; bump `meta.v` on change

## What you cannot do

- Generate broker-specific configs (Kafka topics, RabbitMQ exchanges, etc.)
- Recommend specific messaging platforms or vendors
- Design sync REST APIs (messaging/async focus only)
- Emit `MessageDesign.json` or files outside the IR
- Invent events not grounded in epic/ADR
- Specify auth for brokers (defer to `security` skill)
- Define infrastructure provisioning or network topology

---

## How to apply (procedure)

1. Read epic, ADR, patterns, domain modelling, and existing IR.
2. Confirm async/event requirements exist; otherwise skip or leave empty.
3. Enrich `messaging`; cross-link `integrations[]` for external systems.
4. Write updated IR; bump `meta.v`.

---

## IR output structure

```json
{
  "messaging": {
    "architecture": {
      "patterns": ["pub-sub"],
      "eventSourcing": {},
      "cqrs": {},
      "routing": {}
    },
    "events": [
      { "id": "EVT-001", "name": "", "schema": {}, "version": "1.0", "source": "" }
    ],
    "commands": [
      { "id": "CMD-001", "name": "", "schema": {}, "version": "1.0" }
    ],
    "patterns": {
      "producers": [],
      "consumers": [],
      "partitioning": {},
      "ordering": {}
    },
    "reliability": {
      "delivery": "at-least-once",
      "idempotency": {},
      "dlq": {},
      "retry": {}
    },
    "monitoring": {
      "metrics": [],
      "tracing": {},
      "alerting": {}
    }
  }
}
```

Compact JSON; use IDs for repeated payload shapes.

---

## Input grounding

Ground every decision in epic, ADR, common patterns, and target domain modelling. Cite ADR for cross-cutting messaging choices.
