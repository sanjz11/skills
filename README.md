# Design Workflow Skills

Technology-agnostic design pipeline skills for the design phase.

## Tuning model

- **Generic skills** — IR, security, data, messaging (no stack names)
- **Adapter skills + references/** — stack-specific patterns
- **config/adr-blueprint.json** — ADR keys + defaults when upload missing
- Bootstrapped at runtime to `src/output_workflow/_internal/_config/`
- **design-technology-base** bundles `technology-registry.json`
- **design-requirements-base** bundles `adr-blueprint.json`
- **design-generic-core** bundles `core-design-model-schema.json`

## Skills

- `design-requirements-base`
- `design-generic-core`
- `design-generic-enricher`
- `design-generic-security`
- `design-generic-data`
- `design-generic-messaging`
- `design-technology-base`
- `design-skill-resolver`
- `design-deliverable`
- `java-adapter`
- `react-native-adapter`
- `mulesoft-adapter`
- `generic-adapter`
- `react-adapter`
- `angular-adapter`
- `nodejs-adapter`
- `python-adapter`
- `dotnet-adapter`

Each folder contains `SKILL.md` loaded from `github.com/sanjz11/skills`.
