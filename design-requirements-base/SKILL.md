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

Always produce `requirements_context.json` with a **complete ADR key set**, and **`inputs_manifest.json`** that classifies every uploaded document by **content** (not UI slot) — so downstream steps use the right material even when users mis-place files.

## Input classification (mandatory when uploads exist)

Upload slots (`epic_template`, `adr_blueprint`, `common_patterns`, `target_domain_modelling`) are **hints only**.

1. Load `references/input-classification-pattern.md` (curl fallback if not bundled).
2. `read_file` every non-empty upload from **all four slots**.
3. Classify each file: `epic` | `adr` | `domain_model` | `patterns` | `integration_spec` | `ux_spec` | `unknown`.
4. Write `src/output_workflow/_internal/inputs_manifest.json` with `uploads[]`, `resolved{}`, `slotCoverage`, `assumptions`.
5. Use `resolved.*` buckets for ADR merge and IR — never ignore a file because it is in the "wrong" slot.

| Example mis-placement | Handling |
|---------------------|----------|
| Epic in `target_domain_modelling` | Classify as `epic`; use for capabilities and flows |
| Domain glossary in `epic_template` | Classify as `domain_model`; use for entities / contexts |
| ADR in `common_patterns` | Classify as `adr`; merge into requirements_context |

## Bundled file

`adr-blueprint.json` — ADR keys, defaults, category defaults (in this skill repo folder).

## Bootstrap

1. Load this skill via `skills` tool.
2. Read bundled `adr-blueprint.json`.
3. Write to `src/output_workflow/_internal/_config/adr-blueprint.json`.

Use bootstrapped copy at `_internal/_config/adr-blueprint.json` — not `config/`.

### Bootstrap fallback

If `adr-blueprint.json` is not visible next to `SKILL.md`:

`curl -fsSL https://pscode.lioncloud.net/rohranja3/coe-skills/-/raw/main/design-requirements-base/adr-blueprint.json`

Write result to `_internal/_config/adr-blueprint.json` before synthesizing requirements.

## Success criteria

- [ ] `_internal/_config/adr-blueprint.json` bootstrapped from this skill
- [ ] `inputs_manifest.json` exists when any upload slot is non-empty
- [ ] Every uploaded file appears in `uploads[]` with `classifiedAs` and `confidence`
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
| User uploads (any slot) | Classify via `inputs_manifest.json` → `resolved.*` |
| User ADR content | `resolved.adr` + `adr_blueprint` slot (content wins over slot) |
| Epic / patterns / domain | `resolved.epic`, `resolved.patterns`, `resolved.domain_model` |

## Merge procedure (strict order)

For each key in `adr-blueprint.json` → `keys`:

1. **Classified ADR** — parse all files in `inputs_manifest.resolved.adr` (`source: uploaded`)
2. **ADR slot upload** — if not already consumed from manifest (`source: uploaded`)
3. **Profile `adrDefaults`** — from `technology_context.profileId` or pending profile match (`source: profile`)
4. **Category default** — `key.categoryDefaults[architecture_type]` (`source: category`)
5. **Global default** — `key.default` (`source: default`)
6. **Epic inference** — from `resolved.epic` when epic explicitly states value (`source: inferred`, `needsReview: true`)

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
    "inputs_manifest": "present",
    "classifiedEpicFiles": 1,
    "classifiedDomainFiles": 0,
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

| Missing / mis-placed | Action |
|---------------------|--------|
| adr_blueprint slot empty | Check `resolved.adr` in manifest; synthesize if still empty |
| epic_template slot empty | Use `resolved.epic` from manifest |
| domain doc in wrong slot | Use `resolved.domain_model` — IR step consumes manifest |
| common_patterns slot empty | Use `resolved.patterns` if classified elsewhere |
| All slots empty | Synthesize ADR; note in assumptions; do not stop |
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
