---
name: "lpapi-legacy-wire-bff-code-pattern"
description: "Generates ORBT LPAPI/frozen legacy-wire BFF Java matching lpapi-sales-bff-src: byte-exact LPAPI wire preservation, thin Feign to domain, wire adapters, Resilience4j, Log4j2, OAuth2/Redis. Use when generating or fixing LegacyWire/LPAPI BFF from story packs with frozen API contracts."
version: 1
created: "2026-07-16"
updated: "2026-07-16"
---
## When to Use
Use when LegacyWireBffCodegen / TestAuthor / BuildFix / GapFix generate or fix the frozen/legacy LPAPI wire BFF module from a story pack. Apply when inventory marks `legacy_bff`, `lpapi`, or `parity == "frozen"`. Story pack frozen `*__API_contract.md` + parity oracles = WHAT (exact paths, field names, casing, nullability). This skill = HOW (Spring Boot structure mirroring `lpapi-sales-bff-src` repo). Prefer this over authored Web BFF patterns when wire is frozen. Companions: `bff-service-coreframework-guidelines`, `java21-springboot4-best-practices`, `secrets-management-azure-key-vault`, `security-http-headers-spring-boot`.

## Procedure
1. **Package layout (mandatory)**: Root `com.renuity.lpapi` (or inventory base). Create: `*Application`, `client/`, `config/`, `controller/` (or `adapter/` when pack uses legacy servlet-style naming), `dto/legacy/` (frozen wire shapes), `dto/domain/` (Feign/domain DTOs), `exception/`, `mapper/` (wire↔domain adapters), `service/`, `service/impl/`, `utils/`. Artifact id `lpapi-sales-bff-src`, app name `lpapi-sales-bff`. NEVER add `model/`, `repository/`, Liquibase, or DataSource.
2. **Boot / framework**: Parent `orbt-common-parent-lib`. Same enterprise stack as Web BFF: `core-framework-common|web|data`, OpenFeign, Resilience4j, springdoc (only where not conflicting with frozen wire), OAuth2, Redis, `jjwt`. Exclude logback-classic. `@SpringBootApplication` + `@EnableFeignClients` + `@ComponentScan` app + `com.renuity.core.{common,data,web}`. Virtual threads on. Context path `/api/lpapi` (or exact path from frozen contract). Deploy target: Azure App Service `app-lpapi-sales-bff-orbt-{env}-eus2-01`, container image `lpapi-sales-bff`.
3. **Frozen wire first**: Before coding, load frozen API contract from pack (`*__API_contract.md`, parity oracle, golden fixtures). When `parity == "frozen"`: JSON field names, HTTP paths, query param names, status codes, and response nesting MUST match contract byte-for-byte. Use `@JsonProperty` exact legacy names even when non-idiomatic. Do NOT wrap frozen responses in `ApiResponse` unless contract specifies it. Prefer legacy DTO classes in `dto.legacy` over modern records when contract predates records.
4. **Controllers / adapters**: `{Feature}LpapiController` or legacy route names from contract. `@RestController` with **exact** `@RequestMapping` from frozen doc. Thin: validate inputs per contract, delegate to `{Feature}LpapiBffService`, return legacy wire type directly or `ResponseEntity` with contract status. Add SLF4J logging + Jakarta validation on adapters; do NOT add OpenAPI annotations that alter documented wire. Skip RBAC annotations only when contract/legacy clients do not send auth headers; otherwise mirror Web BFF RBAC.
5. **Services**: `{Feature}LpapiBffService` + impl. Orchestration only: map legacy request → domain Feign call → map domain response → legacy wire. Handle multi-tenant via `TenantParallelExecutor` when contract supports tenant headers; aggregate per contract (some LPAPI endpoints are single-tenant). No domain business logic.
6. **Feign clients**: Same as Web BFF — `@FeignClient` to domain service, `X-Tenant-Id`, Resilience4j annotations + fallbacks. Domain paths from inventory domain layer, NOT legacy LPAPI paths.
7. **Mappers**: Dedicated legacy↔domain mappers. Preserve legacy null handling (empty string vs null vs omitted field) per golden fixtures. When contract marks field `readOnly` on response, never populate from domain if legacy sent empty.
8. **Exceptions**: Map domain/Feign failures to **legacy error shapes** when contract defines them; otherwise use minimal legacy error body (not modern `ApiResponse` unless contract requires). Log correlation id; never expose stack traces in body.
9. **Resilience / security**: Resilience4j on Feign only. OAuth2/Redis same as Web BFF when ADRs require. Never log legacy session tokens or API keys.
10. **Logging**: SLF4J + Log4j2 only. Log operation + tenant + correlation; never log full legacy payloads containing PII when prod profile disables it.
11. **Tests**: Golden-fixture tests mandatory when `parity == "frozen"` — deserialize expected JSON from contract/oracle, invoke controller/service, assert byte-exact JSON (or canonical JSON comparison). Unit-test mappers for edge null/empty cases. Mock Feign for service tests. WireMock for integration. JaCoCo controller ≥0.90.
12. **Feature adaptation**: Implement only endpoints in `endpoint_inventory.json` for `legacy_bff` / `lpapi` layer. If layer absent, write `SKIP_LEGACY_BFF.md` and exit. Rename feature prefixes from reference patterns; never invent new legacy fields.

## Pitfalls
- NEVER modernize frozen wire (no ApiResponse wrapper, camelCase renames, or REST cleanup) when parity is frozen
- NEVER add JPA, repositories, or domain business rules in LPAPI BFF
- NEVER use Web BFF DTOs (`dto.response.ApiResponse` grid shapes) for frozen LPAPI endpoints
- NEVER skip golden-fixture / parity tests when contract marks frozen
- NEVER field `@Autowired`; NEVER omit Feign resilience fallbacks
- NEVER commit secrets; use env placeholders only
