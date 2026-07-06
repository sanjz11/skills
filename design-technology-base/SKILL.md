---
name: design-technology-base
description: >-
  Base technology-aware design patterns for the Design Workflow. Use when resolving
  target stack, selecting enabled design domains, or generating stack-appropriate
  pseudo-code and artifacts. Always read technology_context.json first.
---

# Design Technology Base

## When to use

- **Technology Context Builder** agent (V6) — resolves stack, writes `technology_context.json`
- Technology Adapter loads `profile.skill` / `profile.adapter` from registry

## V6 architecture

```
IR (generic) → Technology Adapter (stack-specific)
```

Generic agents never read adapter skills. Only Technology Adapter Agent loads `adapters/*-adapter` skills.

## Resolution procedure

1. Read `config/technology-registry.json`.
2. Match `target_technology_stack` against `profiles.*.aliases` (case-insensitive).
3. If no match: use `architecture_type` category → `generic-backend` or category default.
4. Set `enabled_domains` from `categories[category].defaultDomains`.
5. Disable `database` for pure frontend/mobile when ADR says BFF-only; disable `presentation` for pure backend/integration.
6. Write `src/output_workflow/technology_context.json`:

```json
{
  "stack": "<user input>",
  "profileId": "<matched profile id>",
  "displayName": "<profile displayName>",
  "category": "<backend|frontend|mobile|integration|cloud|fullstack|data>",
  "enabled_domains": ["application", "..."],
  "profile": {
    "pseudoCodeLanguage": "...",
    "legacySourceLanguages": ["..."],
    "skill": "...",
    "patterns": ["..."],
    "contractFormats": ["openapi"]
  },
  "meta": { "ts": "<ISO8601>", "v": "1.0" }
}
```

## Pseudo-code rules

- Never hardcode Java unless `profileId` is `java-spring`.
- Use the profile's `pseudoCodeLanguage` for all business-logic migration blocks.
- Legacy excerpts use languages from `legacySourceLanguages`; prefer the language in source docs.

## Domain output paths

| Domain | Directory | Primary JSON |
|--------|-----------|--------------|
| application | Application/ | Design.json |
| messaging | Messaging/ | MessageDesign.json |
| security | Security/ | Security.json |
| database | Database/ | Database.json |
| presentation | Presentation/ | PresentationDesign.json |
| integration | Integration/ | IntegrationDesign.json |
| cloud | Cloud/ | CloudDesign.json |

## Workflow graph routing (V6)

V6 uses a **single linear pipeline** — no per-stack graph branching. The Technology Adapter decides which artifact folders to emit based on IR content and `technology_context.category`.

- **backend/fullstack**: OpenAPI 3.1 primary
- **integration**: RAML or OpenAPI + AsyncAPI for events
- **frontend/mobile**: OpenAPI/GraphQL client contracts; component/state design in Presentation/

## Extending for new technologies

Add one entry to `technology-registry.json` + one skill file. No workflow graph changes required.
