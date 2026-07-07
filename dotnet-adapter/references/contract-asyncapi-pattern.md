# AsyncAPI design contract (messaging)

Generate `Messaging/asyncapi.yaml` when IR `messaging.events` or `messaging.commands` is non-empty.

## Rules

1. AsyncAPI 2.6+ — `asyncapi: 2.6.0` header.
2. Map IR `messaging.commands` → `channels` with `publish` or `subscribe` as appropriate.
3. Map IR `messaging.events` → event channels with payload schemas from domain events.
4. Document delivery semantics from IR `messaging.reliability` (at-least-once, etc.).
5. Cross-reference in `Messaging/MessageDesign.json` via `asyncApiRef: asyncapi.yaml`.
