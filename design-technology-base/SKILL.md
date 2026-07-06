---
name: design-technology-base
description: >-
  Resolve target technology profile from registry and user inputs. Load when
  writing technology_context.json. Defines stack resolution, enabled domains,
  and workspace artifact layout — no design content.
---

# Design Technology Base

## When to use

Load when resolving the target stack and writing `technology_context.json`.

Read `config/technology-registry.json` and user stack inputs before writing output.

## Architecture principle

```
Requirements → technology context → generic IR → stack adaptation → deliverables
```

Generic IR skills never load stack adapter skills. Only the stack adaptation step loads `*-adapter` skills from `technology_context.profile.skill`.

## Resolution procedure

1. Read `config/technology-registry.json`.
2. Match `target_technology_stack` against `profiles.*.aliases` (case-insensitive).
3. If no match: use `architecture_type` category → `generic-backend` or category default.
4. Set `enabled_domains` from category and ADR (e.g. omit `database` for BFF-only frontend; omit `presentation` for pure backend/integration).
5. Write `src/output_workflow/_internal/technology_context.json`:

```json
{
  "stack": "<user input>",
  "profileId": "<matched profile id>",
  "displayName": "<profile displayName>",
  "category": "<backend|frontend|mobile|integration|cloud|fullstack|data>",
  "enabled_domains": ["application"],
  "profile": {
    "pseudoCodeLanguage": "...",
    "legacySourceLanguages": ["..."],
    "skill": "<adapter-skill-name>",
    "adapter": "<adapter-id>",
    "patterns": ["..."],
    "layerMapping": {},
    "artifacts": [],
    "contractFormats": ["openapi"]
  },
  "meta": { "ts": "<ISO8601>", "v": "1.0" }
}
```

Copy `layerMapping`, `artifacts`, `framework`, `namingConvention` from the matched registry profile.

## Pseudo-code rules

- Use `profile.pseudoCodeLanguage` from technology_context for business-logic migration — never assume a language.
- Legacy excerpts use `legacySourceLanguages` from the profile or source documents.

## Workspace artifact layout

Internal artifacts live under `src/output_workflow/_internal/`:

| Domain | Directory (under `_internal/`) | Primary JSON |
|--------|-------------------------------|--------------|
| application | Application/ | Design.json |
| messaging | Messaging/ | MessageDesign.json |
| security | Security/ | Security.json |
| database | Database/ | Database.json |
| presentation | Presentation/ | PresentationDesign.json |
| integration | Integration/ | IntegrationDesign.json |
| cloud | Cloud/ | CloudDesign.json |

Shareable deliverables live at `src/output_workflow/` root only (consolidated files).

## Category guidance

- **backend / fullstack:** OpenAPI 3.1 primary for APIs
- **integration:** RAML or OpenAPI + AsyncAPI for events when IR.messaging is non-empty
- **frontend / mobile:** client contracts + presentation design under Presentation/

## Extending for new technologies

Add one entry to `technology-registry.json` + one adapter skill. No workflow graph changes required.
