---
name: design-technology-base
description: >-
  Classify application archetype, resolve technology profile, and write
  technology_context.json with executionContext (single loadedSkillPack).
  Bootstraps _internal/_config/ from this skill.
---

# Archetype & Technology Resolver

## Primary goal

1. **Classify** the application archetype (engineering domain).
2. **Resolve** the technology profile from registry + user inputs.
3. **Emit** `technology_context.json` with `executionContext` — exactly **one** `loadedSkillPack` for downstream dependency injection.

## Bundled files (in this skill repo folder)

| File | Purpose |
|------|---------|
| `technology-registry.json` | Archetypes, profiles, aliases, layerMapping, references |
| `agent-contracts.json` | Per-step goals and archetype-aware best practices |

## Bootstrap (required)

1. Load this skill via the `skills` tool.
2. Read bundled JSON; write to `src/output_workflow/_internal/_config/`.
3. If bundled files not visible, curl from `https://raw.githubusercontent.com/sanjz11/skills/main/design-technology-base/`.

## Archetype classification (mandatory — before profile match)

Classify from **all** signals (priority: ADR > epic > user `architecture_type` > registry default):

| archetype.id | When to use |
|--------------|-------------|
| `backend-service` | APIs, services, persistence, domain logic — Java, .NET, Node, Python |
| `web-ui` | Web pages, components, routing — React, Angular, Vue |
| `mobile-ui` | Mobile screens, navigation, device APIs — React Native, Flutter |
| `integration-service` | ESB, iPaaS, message flows — MuleSoft, Camel, IIB |
| `data-pipeline` | ETL, batch, data quality — Informatica, Talend, ADF |

Use `categoryArchetypeMap` in registry when `architecture_type` is set. Override when epic/ADR clearly describe a different domain (e.g. MuleSoft flows with `architecture_type: backend` → `integration-service`).

## Technology profile resolution

1. Match `target_technology_stack` to `profiles.*.aliases`.
2. If no match: use category default profile for classified archetype.
3. Verify profile `category` is compatible with archetype (note mismatch in `meta.assumptions` if not).

## technology_context.json shape (required)

```json
{
  "meta": { "v": "1.0", "ts": "ISO8601", "assumptions": [] },
  "archetype": {
    "id": "backend-service",
    "label": "Backend Services",
    "source": "registry | epic | adr | user"
  },
  "executionContext": {
    "loadedSkillPack": "java-adapter",
    "designCapabilities": [],
    "skillResolution": {
      "mode": "single-pack",
      "resolvedAt": "archetype-technology-resolver"
    }
  },
  "profileId": "java-spring",
  "profile": {
    "skill": "java-adapter",
    "displayName": "...",
    "category": "backend",
    "designContracts": [],
    "layerMapping": {},
    "artifacts": [],
    "references": [],
    "bestPractices": []
  }
}
```

Populate `executionContext.designCapabilities` from `registry.archetypes[archetype.id].designCapabilities`.

Copy `profile.designContracts[]` verbatim from registry profile into `technology_context.profile.designContracts`.

Set `executionContext.loadedSkillPack` = `profile.skill` (exactly one string).

## Success criteria

- [ ] `_config/technology-registry.json` bootstrapped
- [ ] `archetype.id` classified and documented
- [ ] `executionContext.loadedSkillPack` set to single adapter skill name
- [ ] `profile.designContracts[]` copied from registry
- [ ] `profile.skill` matches `loadedSkillPack`
- [ ] No error-only context without curl fallback attempt

## Handling missing inputs

| Situation | What to do |
|-----------|------------|
| Empty stack name | Category default profile for classified archetype |
| Ambiguous epic | Prefer `architecture_type`; document in assumptions |
| ADR conflicts with stack | Note in assumptions; prefer ADR for archetype, user for profile when explicit |

## Extending stacks

1. Add profile + archetype mapping in `technology-registry.json`.
2. Publish adapter skill pack on PSCode.
3. No workflow graph changes — resolver picks the new pack.

## Completion gate

Archetype classified, single skill pack resolved, executionContext written.
