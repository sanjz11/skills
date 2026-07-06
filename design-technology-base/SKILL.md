---
name: design-technology-base
description: >-
  Resolve technology profile and write technology_context.json. Primary goal:
  always produce a valid profile from registry + inputs. Fall back to category
  defaults when stack name is ambiguous or missing.
---

# Design Technology Base

## Primary goal

Always write a valid `src/output_workflow/_internal/technology_context.json` that resolves **profileId**, **adapter skill**, **layerMapping**, and **artifacts** — even when the user stack string is vague, misspelled, or partially specified.

## Success criteria

- [ ] `technology_context.json` exists under `_internal/`
- [ ] `profileId` and `profile.skill` match an entry in `config/technology-registry.json`
- [ ] `layerMapping`, `artifacts`, `framework`, `namingConvention` copied from profile
- [ ] `meta.v` set; `meta.assumptions` explains any fuzzy match or fallback
- [ ] No design artifacts or IR created in this step

## When to use

Load when resolving target stack before IR build or adaptation.

## Procedure

1. Read `config/technology-registry.json`.
2. Match `target_technology_stack` to `profiles.*.aliases` (case-insensitive, substring tolerant).
3. If no match: map `architecture_type` → category default profile (`generic-backend` or registry default).
4. Set `enabled_domains` from category + ADR hints.
5. Write `src/output_workflow/_internal/technology_context.json` — include from profile: `layerMapping`, `artifacts`, `references`, `bestPractices`, `adrDefaults`.
6. Bump `meta.v` on UPDATE when profile unchanged.

## Output structure

```json
{
  "stack": "<user input or best resolved label>",
  "profileId": "<registry id>",
  "displayName": "<profile displayName>",
  "category": "<backend|frontend|mobile|integration|cloud|fullstack|data>",
  "enabled_domains": ["application"],
  "profile": {
    "pseudoCodeLanguage": "...",
    "legacySourceLanguages": ["..."],
    "skill": "<adapter-skill-name>",
    "adapter": "<adapter-id>",
    "patterns": [],
    "layerMapping": {},
    "artifacts": [],
    "contractFormats": ["openapi"]
  },
  "meta": { "ts": "<ISO8601>", "v": "1.0", "assumptions": [] }
}
```

## Handling missing or incomplete inputs

| Situation | What to do |
|-----------|------------|
| Empty `target_technology_stack` | Use `architecture_type` → category default profile; assumption: "stack not specified" |
| Ambiguous alias (matches multiple) | Prefer exact alias > category match > `defaultProfile` in registry |
| `architecture_type` only | Select default profile for that category |
| `technology_notes` only | Parse for framework hints; match aliases in notes |
| ADR specifies different stack | Note conflict in `meta.assumptions`; prefer user stack unless ADR explicitly overrides |
| Registry unreachable | Use `generic-backend` profile fields from local copy if present in workspace |

**Never:** hardcode a stack without registry reference; skip writing the JSON file.

## Workspace layout

- Internal artifacts: `src/output_workflow/_internal/{Application,Presentation,...}/`
- Shareable deliverables: `src/output_workflow/consolidated_design.{md,json}` only

## Architecture principle

```
Requirements → technology context → generic IR → stack adaptation → deliverables
```

Generic IR skills never load `*-adapter` skills. Only stack adaptation loads `technology_context.profile.skill`.

## Category guidance (artifact emphasis)

| Category | Primary contracts | Typical internal folders |
|----------|-------------------|---------------------------|
| backend / fullstack | OpenAPI 3.1 | Application/, Security/, Database/, Messaging/ |
| integration | RAML or OpenAPI + AsyncAPI when IR.messaging is non-empty | Integration/, Messaging/ |
| frontend / mobile | OpenAPI or GraphQL client contracts | Presentation/, Application/ (client only) |
| cloud / data | OpenAPI or infrastructure contracts per profile | Cloud/ or data platform folders |

Stack adaptation chooses which folders to emit from IR content and `enabled_domains` — not from pipeline branching.

## Extending the registry

1. Add a profile block to `config/technology-registry.json` (`aliases`, `layerMapping`, `adrDefaults`, `references`, `bestPractices`).
2. Add an adapter skill folder with `SKILL.md` and `references/`.
3. Point `profile.skill` and `profile.references` at the new adapter.

Orchestration steps stay the same; quality tuning is **registry + skills + references** only.

## Completion gate

Valid JSON on disk, profile resolvable, assumptions documented for any fuzzy match.
