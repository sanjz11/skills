---
name: design-generic-messaging
description: >-
  Add generic messaging section to Core Design Model IR. Technology-blind.
  Use in Core Design Model Agent when enriching IR.messaging.
---

# Generic Messaging (IR)

Add to `core_design_model.json` → `messaging`:

- `events`, `commands` — schemas (generic JSON)
- `patterns` — pub/sub, point-to-point
- `reliability` — delivery guarantees, idempotency, DLQ

Only when epic/ADR require async/events. Leave empty otherwise.
