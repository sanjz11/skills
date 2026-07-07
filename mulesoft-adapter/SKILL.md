---
name: mulesoft-adapter
description: >-
  Map IR to MuleSoft Anypoint artifacts with API-led layering, connectors,
  DataWeave transforms, and error handlers. Load when
  technology_context.profile.skill is mulesoft-adapter.
---

# MuleSoft Adapter

## Primary goal

Transform `core_design_model.json` into **complete MuleSoft Anypoint design artifacts** — Experience/Process/System APIs, connectors, DataWeave transforms, error handlers, MUnit scope — under `_internal/`.

## Bootstrap fallback (references)

If `references/*` files are not visible next to `SKILL.md`, fetch from:

`https://raw.githubusercontent.com/sanjz11/skills/main/mulesoft-adapter/references/<filename>`

Use `command_line` (`curl -fsSL`) before proceeding without templates.

## Success criteria

- [ ] Read IR and `technology_context.json`; abort if error-only context — re-bootstrap registry first
- [ ] **API-led design:** Experience API (RAML), Process flows/subflows, System API when IR data layer present
- [ ] **Every process flow** has explicit error handler, retry policy aligned with IR `errors[]` and `messaging.reliability`
- [ ] **Connectors** mapped from IR `integrations[]` with config property names
- [ ] DataWeave 2.0 transforms for each IR workflow transformation step
- [ ] `legacyLogicMigration` as **DataWeave / flow pseudo-code** with `# Legacy line N:` traceability
- [ ] MUnit test list per main flow

## Outputs (required when IR sections apply)

- `src/output_workflow/_internal/Integration/IntegrationDesign.json`
- `src/output_workflow/_internal/Integration/IntegrationDesign.md`
- `src/output_workflow/_internal/Integration/integration-apis.raml`
- `src/output_workflow/_internal/Application/Design.json`
- `src/output_workflow/_internal/Messaging/MessageDesign.json`
- `src/output_workflow/_internal/Security/Security.json`

## API-led procedure (mandatory)

1. Load `references/api-led-pattern.md` and `connector-error-handler-pattern.md`.
2. Map synchronous `apiOperations` → Experience API RAML resources (e.g. `/health`).
3. Map IR `flows` / `workflows` → Process API main flows; step clusters → subflows.
4. Map IR `dataEntities` / persistence reads → System API resources (when applicable).
5. Map IR `integrations[]` → connector configs with environment properties.
6. Map IR `errors[]` → error types + `on-error-propagate` / `on-error-continue` + selective retry.
7. Map IR `messaging.commands` → Anypoint MQ/JMS/VM listeners in Process layer.
8. Document API Manager policies (client ID, SLA) in Security.json when ADR requires.

## Reference patterns

- `references/api-led-pattern.md`
- `references/connector-error-handler-pattern.md`
- `references/IntegrationDesign-json-shape.json`

## Best practices (enforced)

- Experience / Process / System separation — no business orchestration in Experience layer
- DataWeave for all transforms; no Java custom transformers unless IR requires
- Error handler on **every** flow and subflow
- Correlation ID propagation across async and sync paths
- Externalize paths, queues, credentials — never embed in flow logic
- Filename preservation and archive routes from IR business rules reflected in file connector config

## Handling missing inputs

| Situation | What to do |
|-----------|------------|
| IR.integration empty | Experience API from apiOperations only; Process from workflows |
| IR.messaging empty | Skip MessageDesign.json with assumption |
| technology_context.error | Re-bootstrap — do not emit generic "background-processing-unit" design |

## Do not

- Produce technology-agnostic integration prose without MuleSoft artifact names
- Write consolidated deliverables
- Expose business CRUD RAML when IR/ADR limits sync surface to health probe only

## Completion gate

Integration folder complete with RAML, flows, connectors, error catalog, and MUnit list; artifacts use MuleSoft terminology throughout.
