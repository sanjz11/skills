---
name: design-requirements-base
description: >-
  Synthesize and merge requirements when ADR or epic uploads are missing or
  partial. Primary goal: always produce requirements_context.json from
  adr-blueprint defaults + registry profile + user uploads. Never fail for lack
  of ADR.
---

# Requirements & ADR Synthesis

## Primary goal

Always produce `src/output_workflow/_internal/requirements_context.json` with a **complete ADR key set** — merging uploaded ADR, epic hints, category defaults, and profile `adrDefaults` — so downstream steps never skip work because ADR was not uploaded.

## Bundled file

`adr-blueprint.json` — ADR keys, defaults, category defaults (in this skill repo folder).

## Bootstrap

1. Load this skill via `skills` tool.
2. Read bundled `adr-blueprint.json`.
3. Write to `src/output_workflow/_internal/_config/adr-blueprint.json`.

Use bootstrapped copy at `_internal/_config/adr-blueprint.json` — not `config/`.

## Success criteria

- [ ] `_internal/_config/adr-blueprint.json` bootstrapped from this skill
- [ ] `requirements_context.json` exists under `_internal/`
- [ ] Every key in `adr-blueprint.json` → `keys` has a resolved value
- [ ] `sources` documents origin per key: `uploaded` | `profile` | `category` | `default` | `inferred`
- [ ] `meta.synthesized` is `true` when adr_blueprint upload was missing or partial
- [ ] User upload values **always win** over defaults
- [ ] `meta.assumptions` lists keys inferred from epic/domain only

## When to use

Load at the **start** of technology context resolution and **before** IR build.

## Inputs

| Source | Path |
|--------|------|
| ADR blueprint | `_internal/_config/adr-blueprint.json` (bootstrap from this skill) |
| Technology registry | `_internal/_config/technology-registry.json` (from design-technology-base skill) |
| Resolved profile | `src/output_workflow/_internal/technology_context.json` (when available) |
| User ADR upload | epic inputs `adr_blueprint` |
| Epic / patterns / domain | Design Phase Inputs |

## Merge procedure (strict order)

For each key in `adr-blueprint.json` → `keys`:

1. **Uploaded ADR** — if `adr_blueprint` parseable and key present → use value (`source: uploaded`)
2. **Profile `adrDefaults`** — from `technology_context.profileId` or pending profile match (`source: profile`)
3. **Category default** — `key.categoryDefaults[architecture_type]` (`source: category`)
4. **Global default** — `key.default` (`source: default`)
5. **Epic inference** — only if epic explicitly states value (`source: inferred`, `needsReview: true`)

Never leave a key undefined.

## Output structure

```json
{
  "adr": {
    "architecture_style": "layered-ddd",
    "api_style": "rest",
    "auth_model": "token-based"
  },
  "sources": {
    "architecture_style": "profile",
    "api_style": "default"
  },
  "inputs": {
    "adr_blueprint": "missing|partial|complete",
    "epic_template": "missing|present",
    "technology_profileId": "java-spring"
  },
  "meta": {
    "ts": "<ISO8601>",
    "v": "1.0",
    "synthesized": true,
    "assumptions": []
  }
}
```

## Fillable ADR template for users

When ADR upload is missing, also write `src/output_workflow/_internal/adr_fillable_template.json` — same keys as blueprint with:
- `value` — resolved synthesized value
- `default` — from blueprint
- `description` — from blueprint
- `userOverride` — `null` (user may replace on UPDATE run)

Users can download, edit `userOverride` fields, and re-upload as `adr_blueprint`.

## Handling missing inputs

| Missing | Action |
|---------|--------|
| adr_blueprint upload | Full synthesis from profile + category + defaults; `meta.synthesized: true` |
| epic_template | Use domain model + technology_notes + registry profile only |
| common_patterns | Skip; note in assumptions |
| target_domain_modelling | Derive bounded contexts later in IR from APIs |
| technology_context not yet written | Resolve profile first from registry + stack inputs |

**Never stop** because ADR is absent.

## Best practices

- Treat synthesized ADR as **first-class input** to IR — same weight as upload when upload missing
- Mark deliverable sections `[SYNTHESIZED ADR]` when upload was absent
- Do not invent compliance (`GDPR`, `HIPAA`) — keep `compliance: none-stated` unless epic states it
- Re-merge on UPDATE when user provides ADR upload — uploaded keys replace synthesized
- Log changed keys in `design_changelog.md`

## Do not

- Block IR or adapter steps waiting for ADR upload
- Drop blueprint keys because epic was silent
- Override user ADR values with defaults

## Completion gate

All blueprint keys resolved; `requirements_context.json` written; sources traceable.
