---
name: design-technology-base
description: >-
  Resolve technology profile and write technology_context.json. Bundles
  technology-registry.json. Bootstraps _internal/_config/ from this skill — no
  project workspace required.
---

# Design Technology Base

## Primary goal

Bootstrap platform config from **this skill package**, then write valid `technology_context.json` — even when the user stack string is vague or misspelled.

## Bundled files (in this skill repo folder)

| File | Purpose |
|------|---------|
| `technology-registry.json` | Stack profiles, aliases, layerMapping, references, adrDefaults |
| `agent-contracts.json` | Per-step goals and best practices |

## Bootstrap (required — no project workspace)

Reunity has **no config workspace**. Config lives in skills on GitHub.

**Before resolving the stack:**

1. Load this skill via the `skills` tool.
2. Read bundled `technology-registry.json` from this skill package.
3. Write it to `src/output_workflow/_internal/_config/technology-registry.json`.
4. Read bundled `agent-contracts.json` and write to `src/output_workflow/_internal/_config/agent-contracts.json`.

All later steps read from `src/output_workflow/_internal/_config/` — never from `config/`.

### Bootstrap fallback (when bundled JSON not visible)

Reunity may expose only `SKILL.md` to `read_file`. If sibling files are missing from the skill package snapshot:

1. Fetch with `command_line`:
   - `curl -fsSL https://raw.githubusercontent.com/sanjz11/skills/main/design-technology-base/technology-registry.json`
   - `curl -fsSL https://raw.githubusercontent.com/sanjz11/skills/main/design-technology-base/agent-contracts.json`
2. Write fetched content to `_internal/_config/` paths above.
3. **Never** write error-only `technology_context.json` until both fetches fail.

If fetch fails, retry once; then write `technology_context.json` with `error` **and** `meta.bootstrapAttempted: true` documenting URLs tried.

If `_config/` files already exist (UPDATE run), refresh only when registry skill version changed or change_requirements says so.

## Success criteria

- [ ] `_internal/_config/technology-registry.json` bootstrapped from this skill
- [ ] `technology_context.json` exists under `_internal/`
- [ ] `profileId` and `profile.skill` match a registry entry
- [ ] `layerMapping`, `artifacts`, `references`, `bestPractices`, `adrDefaults` copied from profile
- [ ] `meta.assumptions` explains any fuzzy match

## Procedure

1. Bootstrap config (above).
2. Read `_internal/_config/technology-registry.json`.
3. Match `target_technology_stack` to `profiles.*.aliases` (case-insensitive).
4. If no match: map `architecture_type` → category default profile.
5. Set `enabled_domains` from category + ADR hints.
6. Write `_internal/technology_context.json` with full profile fields.
7. Bump `meta.v` on UPDATE when profile unchanged.

## Handling missing inputs

| Situation | What to do |
|-----------|------------|
| Empty stack name | Category default profile from bootstrapped registry |
| Registry file missing in run | Bootstrap from this skill first — do not guess without registry |
| ADR/epic imply different stack | Note in `meta.assumptions`; prefer user unless ADR overrides |

## Category guidance

| Category | Primary contracts | Typical folders |
|----------|-------------------|-----------------|
| backend / fullstack | OpenAPI 3.1 | Application/, Security/, Database/ |
| integration | RAML / OpenAPI + AsyncAPI | Integration/, Messaging/ |
| frontend / mobile | Client API contracts | Presentation/, Application/ |

## Extending stacks

1. Edit `technology-registry.json` in this skill on GitHub (add profile + references).
2. Add adapter skill with `references/`.
3. Push skills repo — no orchestration changes.

## Completion gate

Config bootstrapped, context written, profile traceable to registry.
