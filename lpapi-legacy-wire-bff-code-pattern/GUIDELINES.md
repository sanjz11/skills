# LPAPI BFF Layer — Scaffolding & Codegen Guidelines (NON-NEGOTIABLE)

**Audience:** Scaffold / LegacyWireBffCodegen / TestAuthor / BuildFix / GapFix agents  
**Scope:** LPAPI / frozen legacy-wire BFF layer **only**  
**Service name:** `lpapi-sales-bff`  
**Repo / module root:** `lpapi-sales-bff-src`  
**Pattern skill (HOW):** [`lpapi-legacy-wire-bff-code-pattern/SKILL.md`](../skills/lpapi-legacy-wire-bff-code-pattern/SKILL.md)  
**Primary sources of truth (both required):**
1. **Story pack zip** — WHAT to build for the LPAPI BFF layer  
2. **ADRs zip → `_adr_index`** — HOW / platform alignment for the LPAPI BFF layer  

Agents must consult **both**. Neither alone is enough.

---

## 1. Hard scope rules

1. Generate code **only** for the **LPAPI BFF layer** (`legacy_bff` / `lpapi` / `parity == "frozen"` in inventory).
2. Do **not** generate Domain, Web BFF, common-framework, or any other surface in this pass.
3. Do **all** work **inside** the `lpapi-sales-bff-src` repo / module root only.
4. The running service name is **`lpapi-sales-bff`** — never invent alternate service names.
5. **NEVER** create a nested service module such as:
   - `sales-lpapi-bff-service`
   - `lpapi-bff-service`
   - `*-lpapi-bff-service`
   - any second Maven module / nested Boot app under `lpapi-sales-bff-src`
6. `lpapi-sales-bff-src` **is** the LPAPI BFF service. Put `pom.xml`, `src/`, and application code at that root (or the single module root resolved by inventory) — do not nest another service directory inside it.
7. If inventory has no LPAPI / legacy_bff endpoints: write `SKIP_LEGACY_BFF.md` under inventory and **exit**. Do not invent endpoints.

---

## 2. Naming & identity (must match)

| Item | Required value |
|------|----------------|
| Repo / module | `lpapi-sales-bff-src` |
| Maven `artifactId` | `lpapi-sales-bff-src` |
| Spring app name | `lpapi-sales-bff` |
| Container image | `lpapi-sales-bff` |
| Main class | `com.renuity.sales.LpapiSalesBffApplication` |
| Base package | `com.renuity.sales` |
| Pattern skill | `lpapi-legacy-wire-bff-code-pattern` |

Forbidden names for new modules/dirs/apps: `sales-lpapi-bff-service`, `lpapi-bff-service`, `sales-schedule-lpapi-bff` (as a nested copy inside this repo).

---

## 3. Sources of truth (Story pack zip + `_adr_index`)

**Both are mandatory sources of truth** for LPAPI BFF scaffolding and codegen:

| Source of truth | Location | Governs |
|-----------------|----------|---------|
| **Story pack zip** | Workflow **StoryPack** input | **WHAT** — endpoints, frozen API contracts, ACs, field names, parity, LPAPI wire shapes |
| **`_adr_index`** | Inside the **ADRs zip** (open this file first) | **HOW** — stack, security, logging, Feign, deploy, LPAPI-BFF-relevant ADR paths |

Do not invent endpoints, routes, DTO fields, or stack choices that contradict either source.  
Do not use only ADRs without the story pack, and do not use only the story pack without `_adr_index` when ADRs are supplied.

### Precedence when reading

| Priority | Source | Use for |
|----------|--------|---------|
| 1 | **Story pack zip** (entry manifest / `_STORY.md`, frozen `*__API_contract.md`, LPAPI/legacy BFF build contracts, ACs) + derived `_inventory/*` | **WHAT** to build (LPAPI endpoints, fields, parity, acceptance) |
| 2 | ADRs zip → **`_adr_index`** first, then only the ADR paths it lists for **LPAPI / BFF / legacy wire** | **HOW** stack, security, logging, Feign, deploy |
| 3 | Skill `lpapi-legacy-wire-bff-code-pattern` | **HOW** packages, controllers, Feign, frozen wire coding style |
| 4 | Optional guidelines upload (this document) | Hard scope/naming constraints for this layer |

**Conflict resolution (LPAPI BFF only):**
- Story pack vs ADR on **wire/contract (WHAT):** story pack (frozen contract) wins.
- Story pack vs ADR on **platform stack (HOW):** `_adr_index` → listed ADR paths win.
- Package/layout style: skill wins unless story pack or ADR explicitly mandates otherwise.
- This guidelines file wins on **scope/naming** (single repo `lpapi-sales-bff-src`, no nested `*-lpapi-bff-service`).

### Story pack zip procedure (mandatory)

1. Open the **Story pack zip** supplied as workflow input (SOURCE OF TRUTH for **WHAT**).
2. Start at the pack entry manifest (usually `_STORY.md` or the index named in the pack).
3. Load **only** documents needed for the **LPAPI / legacy / frozen wire** surface:
   - LPAPI / legacy BFF build contract(s)
   - Frozen `*__API_contract.md` (or equivalent)
   - Functional ACs scoped to LPAPI / mobile / legacy wire
   - Parity oracles / golden fixtures when present
4. Ignore Domain-only and Web-BFF-only contracts in this pass except where they define the **domain Feign targets** the LPAPI BFF must call.
5. Emit / consume inventories under the single `outputRoot`, then implement **exactly** the LPAPI / `legacy_bff` endpoints listed there.
6. Never invent LPAPI endpoints or fields that are absent from the story pack / inventory.

### ADR alignment procedure (mandatory)

1. Open the ADRs zip and locate **`_adr_index`** (or `_adr_index.md` / `_adr_index.json` — use whatever filename exists in the zip).
2. Read `_adr_index` completely. It is the **only** ADR entry point — do not browse random ADR files first.
3. From `_adr_index`, select **only** entries that apply to:
   - LPAPI BFF / legacy wire / mobile frozen API
   - BFF Feign / Resilience4j / logging / security as used by BFF
4. Follow the **paths listed in `_adr_index`** for those entries. Ignore Domain-only, Web-UI-only, or unrelated surface ADRs unless `_adr_index` explicitly ties them to LPAPI BFF.
5. Record applied ADR ids/paths in inventory / decision notes when required by the workflow.
6. If `_adr_index` and the skill conflict on package layout: **skill wins for LPAPI package tree**; ADR wins for platform stack (Java version, parent POM, secrets, observability) unless the **story pack** frozen contract overrides wire-related choices.

---

## 4. Project structure (CREATE / UPDATE)

Agents **must** follow the tree in `lpapi-legacy-wire-bff-code-pattern/SKILL.md`. Summary:

```
lpapi-sales-bff-src/                    ← THIS is the service root (do not nest another service)
├── pom.xml                             ← artifactId: lpapi-sales-bff-src
├── Dockerfile                          ← image: lpapi-sales-bff
├── settings.xml / devops files as needed
└── src/
    ├── main/java/com/renuity/sales/
    │   ├── LpapiSalesBffApplication.java
    │   ├── client/
    │   ├── controller/
    │   ├── dto/domain|request|response/
    │   ├── exception/
    │   ├── service/ + impl/
    │   ├── util/
    │   └── time/                       ← optional
    ├── main/resources/application*.yml + log4j2.xml
    └── test/java/com/renuity/sales/...
```

### Scaffolding rules

- CREATE: fill `src/` **inside** `lpapi-sales-bff-src`; reuse existing devops (`.github`, Dockerfile) when present.
- UPDATE: modify existing packages in place; never add a sibling `*-service` module.
- Single Boot application only: one `@SpringBootApplication` class.
- No multi-module parent that reintroduces Domain/Web BFF under this repo unless inventory/`module_plan.json` already defines it **outside** this guideline’s LPAPI-only pass (this agent still only touches LPAPI files).

---

## 5. Code generation rules (LPAPI only)

1. **Frozen wire first** — HTTP paths, `@JsonProperty` names, status codes, and bodies must match frozen `*__API_contract.md` / golden fixtures when `parity == "frozen"`.
2. Controllers return **raw legacy types** (lists, plain strings). Do **not** wrap HTTP responses in modern Web-BFF `ApiResponse` unless the frozen contract requires it.
3. Thin controllers → services orchestrate Feign → domain; map domain → legacy wire in service (or mapper if skill/ADR requires).
4. Feign clients call **domain** URLs/paths, never LPAPI paths.
5. Resilience4j (`@CircuitBreaker`, `@Retry`, `@Bulkhead`) on Feign methods.
6. Exception handler returns **legacy plain-string** error bodies (400/401 semantics from contract).
7. No JPA, repositories, Liquibase, or domain business rules in this repo.
8. Constructor injection only; SLF4J + Log4j2 only.
9. Tests: golden fixtures when frozen; mirror package layout under `src/test/java`.

Implement **exactly** the LPAPI endpoints in `endpoint_inventory.json` for this layer — nothing else.

---

## 6. Forbidden actions (explicit)

| Do not | Do instead |
|--------|------------|
| Create `sales-lpapi-bff-service` or `lpapi-bff-service` | Use `lpapi-sales-bff-src` as the only service root |
| Nest a second Boot module inside the repo | Put code under `src/main/java/com/renuity/sales/` |
| Generate Domain or Web BFF in this pass | Skip those layers / leave to their agents |
| Modernize LPAPI routes to `/api/sales/v1/...` | Keep frozen `/api/SalesApi/...` (or contract paths) |
| Ignore story pack zip and invent endpoints/fields | Always open story pack first for **WHAT** (LPAPI contracts/ACs) |
| Ignore `_adr_index` and pick random ADRs | Always start from `_adr_index`, LPAPI paths only |
| Use only ADRs or only story pack | Use **both** sources of truth together |
| Copy `sales-schedule-lpapi-bff` as a nested project | Mirror its **patterns** inside `lpapi-sales-bff-src` |

---

## 7. Agent checklist (before finishing)

- [ ] Only LPAPI / legacy_bff inventory endpoints implemented  
- [ ] All files under `lpapi-sales-bff-src` (no nested `*-lpapi-bff-service`)  
- [ ] `artifactId` / app name = `lpapi-sales-bff-src` / `lpapi-sales-bff`  
- [ ] Package tree matches `lpapi-legacy-wire-bff-code-pattern` skill  
- [ ] Story pack zip read; LPAPI/legacy contracts + ACs used as **WHAT** source of truth  
- [ ] `_adr_index` read; only LPAPI-BFF-relevant ADR paths applied (**HOW** source of truth)  
- [ ] Frozen wire parity preserved (paths, JSON names, error strings) per story pack  

- [ ] Feign → domain with Resilience4j; no JPA  
- [ ] Tests added for controllers/services (and golden fixtures if frozen)  
- [ ] No Domain / Web BFF / duplicate service scaffolding created  

---

## 8. Skill reference

Full HOW procedure and package tree:

- Skill id: `github.com/sanjz11/skills/lpapi-legacy-wire-bff-code-pattern`
- Local: `skills/lpapi-legacy-wire-bff-code-pattern/SKILL.md`

Upload **this guidelines file** as workflow **OptionalGuidelines** (or attach alongside the story pack) so Scaffold + LegacyWireBffCodegen agents apply these constraints on every CREATE/UPDATE run for the LPAPI BFF layer.
