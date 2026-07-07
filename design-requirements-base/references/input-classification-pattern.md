# Input classification pattern

Upload slots are **hints only**. Classify every file by **content**, not by which slot the user selected.

## Slots (UI labels)

| Slot | Preferred content | Also accept when mis-placed |
|------|-------------------|----------------------------|
| `epic_template` | Epic, user stories, scope, capabilities, flows | ADR, domain model, patterns |
| `adr_blueprint` | ADR keys, architecture decisions, NFR matrix | Epic, standards doc |
| `common_patterns` | Cross-cutting patterns, coding standards, NFR baselines | ADR excerpts, integration patterns |
| `target_domain_modelling` | Domain glossary, entities, bounded contexts, ERD | Epic scope, data dictionary |

## Classification types

| `classifiedAs` | Signals in document |
|----------------|-------------------|
| `epic` | Capabilities, user stories, acceptance criteria, in-scope/out-of-scope, workflows |
| `adr` | ADR- IDs, architecture decision, technology choices, auth/cache/messaging decisions |
| `domain_model` | Entities, aggregates, bounded context, glossary, relationships, ERD, ubiquitous language |
| `patterns` | Pattern catalog, standards, reusable components, NFR baselines, error-handling conventions |
| `integration_spec` | Source/target systems, message formats, batch schedules, ETL mappings |
| `ux_spec` | Screens, wireframes, journeys, navigation, page inventory |
| `unknown` | Cannot classify — still ingest; note in assumptions |

## Procedure

1. List every non-empty upload from all four slots.
2. `read_file` each uploaded document (all files in multi-file slots).
3. Assign `classifiedAs` per file from content signals (slot is a weak hint only).
4. Build `resolved` buckets — a file may contribute to multiple buckets when content spans types.
5. Write `inputs_manifest.json` before requirements synthesis or IR build.

## `inputs_manifest.json` shape

```json
{
  "meta": { "v": "1.0", "ts": "ISO8601" },
  "uploads": [
    {
      "slot": "target_domain_modelling",
      "fileRef": "path/from/runtime",
      "fileName": "E-003-epic.md",
      "classifiedAs": "epic",
      "confidence": "high",
      "signals": ["user stories", "capabilities", "acceptance criteria"]
    }
  ],
  "resolved": {
    "epic": ["path/to/classified-epic.md"],
    "adr": [],
    "domain_model": ["path/to/domain.pdf"],
    "patterns": [],
    "integration_spec": [],
    "ux_spec": [],
    "unknown": []
  },
  "slotCoverage": {
    "epic_template": "misplaced|matched|empty",
    "adr_blueprint": "matched|empty",
    "common_patterns": "empty",
    "target_domain_modelling": "misplaced"
  },
  "assumptions": []
}
```

## Consumption rules

| Downstream step | Read from |
|-----------------|-----------|
| ADR synthesis | `resolved.adr` first; epic may supply ADR hints |
| Archetype classifier | all `resolved.*` + stack inputs |
| IR epic content | `resolved.epic` + relevant parts of `unknown` |
| IR domain model | `resolved.domain_model` |
| IR patterns / NFR | `resolved.patterns` |
| Integration / UX IR | `resolved.integration_spec`, `resolved.ux_spec` |

## Mis-placement examples

| User action | Correct handling |
|-------------|------------------|
| Epic uploaded to `target_domain_modelling` | `classifiedAs: epic`; use for capabilities and flows |
| Domain PDF in `epic_template` | `classifiedAs: domain_model`; use for bounded contexts / entities |
| ADR in `common_patterns` | `classifiedAs: adr`; merge into requirements_context |
| Same doc in two slots | Deduplicate by fileRef; classify once |

Never ignore a file because it is in the "wrong" slot.
