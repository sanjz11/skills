---
name: "lpapi-legacy-wire-bff-code-pattern"
description: "Generates ORBT LPAPI/frozen legacy-wire BFF Java matching lpapi-sales-bff-src create-mode layout: byte-exact LPAPI wire, thin Feign to domain, Resilience4j, Log4j2. Use when generating or fixing LegacyWire/LPAPI BFF from story packs with frozen API contracts."
version: 2
created: "2026-07-16"
updated: "2026-07-16"
---
## When to Use
Use when LegacyWireBffCodegen / Scaffold / TestAuthor / BuildFix / GapFix generate or fix the frozen/legacy LPAPI wire BFF for **CREATE** or UPDATE. Apply when inventory marks `legacy_bff`, `lpapi`, or `parity == "frozen"`. Golden references: repo identity **`lpapi-sales-bff-src`** (deploy/image) + proven Java layout from **`sales-schedule-lpapi-bff`** (frozen `/api/SalesApi/*` wire). Story pack frozen `*__API_contract.md` + parity oracles = WHAT. This skill = HOW. Companions: `bff-service-coreframework-guidelines`, `java21-springboot4-best-practices`, `secrets-management-azure-key-vault`, `security-http-headers-spring-boot`.

## CREATE-mode project structure (mandatory)

Scaffold **exactly** this tree under the LPAPI BFF module root (standalone repo `lpapi-sales-bff-src`).  
**Service name is `lpapi-sales-bff`.** Do **not** create nested modules/dirs named `sales-lpapi-bff-service`, `lpapi-bff-service`, or any second Boot app inside this repo — `lpapi-sales-bff-src` **is** the service.

```
lpapi-sales-bff-src/                         # SERVICE ROOT (lpapi-sales-bff) — do not nest another *-service
├── pom.xml                                  # artifactId: lpapi-sales-bff-src
├── Dockerfile                               # multi-stage Maven 21 → JRE; image name lpapi-sales-bff
├── settings.xml                             # enterprise Maven (never commit secrets; use env)
├── .dockerignore
├── .gitattributes
├── .trivyignore
├── CODEOWNERS
├── CONTRIBUTING.md
├── SECURITY.md
├── README.md
├── .github/
│   ├── dependabot.yml
│   └── workflows/
│       ├── build-image.yml                  # image-name: lpapi-sales-bff
│       ├── deploy-image.yml                 # app-lpapi-sales-bff-orbt-{env}-eus2-01
│       ├── rollback.yml
│       ├── pr-validation.yml
│       ├── codeql.yml
│       └── owasp_zap.yml
└── src/
    ├── main/
    │   ├── java/com/renuity/sales/
    │   │   ├── LpapiSalesBffApplication.java
    │   │   ├── client/
    │   │   │   └── {Feature}LpapiDomainClient.java
    │   │   ├── controller/
    │   │   │   └── {Feature}LpapiBFFController.java
    │   │   ├── dto/
    │   │   │   ├── domain/                  # Feign / domain shapes only
    │   │   │   ├── request/                 # frozen LPAPI request wire
    │   │   │   └── response/                # frozen LPAPI response wire
    │   │   ├── exception/
    │   │   │   ├── LpapiBadRequestException.java
    │   │   │   ├── LpapiUnauthorizedException.java
    │   │   │   └── {Feature}LpapiBffExceptionHandler.java
    │   │   ├── service/
    │   │   │   ├── {Feature}LpapiBffService.java
    │   │   │   └── impl/
    │   │   │       └── {Feature}LpapiBffServiceImpl.java
    │   │   ├── time/                        # optional: tenant timezone helpers
    │   │   └── util/                        # LegacyLpapiDateFormats, wire helpers
    │   └── resources/
    │       ├── application.yml
    │       ├── application-dev.yml
    │       ├── application-qat.yml
    │       ├── application-uat.yml
    │       ├── application-prod.yml
    │       └── log4j2.xml
    └── test/java/com/renuity/sales/
        ├── controller/{Feature}LpapiBFFControllerTest.java
        ├── service/{Feature}LpapiBffServiceImplTest.java
        ├── dto/request/…Test.java
        └── util/…Test.java
```

### Naming / identity (CREATE)
| Item | Value |
|------|--------|
| Maven `artifactId` | `lpapi-sales-bff-src` |
| `spring.application.name` / service name | `lpapi-sales-bff` (never `sales-lpapi-bff-service` / `lpapi-bff-service`) |
| Main class | `com.renuity.sales.LpapiSalesBffApplication` |
| Base package | `com.renuity.sales` (do **not** invent `com.renuity.lpapi` unless inventory/ADR overrides) |
| Container image | `lpapi-sales-bff` |
| Azure App Service | `app-lpapi-sales-bff-orbt-{env}-eus2-01` |
| HTTP routes | **Exact** frozen paths from contract (e.g. `/api/SalesApi/GetSalesSchedCal`) — usually **no** `server.servlet.context-path` |
| ADR HOW entry | Always open ADRs zip **`_adr_index`** first; follow only paths for LPAPI/legacy BFF |

### Packages allowed vs forbidden
- **Required:** `client`, `controller`, `dto.domain`, `dto.request`, `dto.response`, `exception`, `service`, `service.impl`, `util` (+ `time` when pack needs TZ).
- **Optional:** `config` only for Feign/Resilience4j beans when not inherited from framework.
- **Forbidden:** `model/`, `repository/`, Liquibase, DataSource, JPA entities, Web-BFF `ApiResponse` on **HTTP** responses when wire is frozen.

## Procedure
1. **Scaffold tree first**: In CREATE mode, create the full structure above **inside** `lpapi-sales-bff-src` before feature code. Keep devops files; fill `src/` here only. Never add a nested `*-lpapi-bff-service` module. For ADRs: open `_adr_index` in the ADRs zip first, then only LPAPI-BFF paths listed there.
2. **Boot / framework**: Prefer parent `orbt-common-parent-lib` when ADRs/enterprise settings require it (standalone sales LPAPI repo). Deps: web + validation + OpenFeign + Resilience4j + Log4j2 (exclude logback) + springdoc (docs only; must not change wire). Add OAuth2/Redis/`core-framework-*` only when ADRs require — frozen mobile clients may not use RBAC headers. `@SpringBootApplication(scanBasePackages = "com.renuity")` + `@EnableFeignClients` + `@EnableCaching`. Virtual threads on when Boot 4 / parent enables them.
3. **Frozen wire first**: Load `*__API_contract.md` + golden fixtures. When `parity == "frozen"`: JSON field names (often PascalCase via `@JsonProperty("SchedDate")`), HTTP paths, status codes, and nesting MUST match byte-for-byte. Prefer Lombok `@Data`/`@Builder` DTOs in `dto.request|response` for legacy wire (not modern records) when contract predates records.
4. **Controllers**: `{Feature}LpapiBFFController`. `@RestController @RequestMapping("/api/SalesApi")` (or exact prefix from contract). Thin: validate, call service, **unwrap** any internal `ApiResponse` / domain envelope and return **raw** legacy type (`List<SalesSchedDTO>`, plain `String`, etc.). Preserve legacy response headers (e.g. `Content-Length`) when contract requires. Do not wrap HTTP body in modern `ApiResponse` unless contract says so.
5. **Services**: `{Feature}LpapiBffService` + `impl`. Orchestration only: map legacy request → Feign domain call → map domain → legacy wire. No business rules / no persistence.
6. **Feign clients**: `{Feature}LpapiDomainClient` — `@FeignClient` to domain URL property (e.g. `${sales.domain.url}`), domain paths (`/api/v1/...` or inventory), **not** LPAPI paths. Annotate with `@CircuitBreaker` + `@Retry` + `@Bulkhead`. Prefer named fallbacks when aligning with Web BFF enterprise style.
7. **Exceptions**: `{Feature}LpapiBffExceptionHandler` scoped to controller package. Return **plain JSON string** bodies for 400/401 matching frozen semantics (e.g. `"Invalid Token, Please Regenerate Token and Try Again!"`). Use `LpapiBadRequestException` / `LpapiUnauthorizedException`. Never expose stack traces.
8. **Util / time**: Put legacy date/format parsers in `util/` (e.g. `LegacyLpapiDateFormats`); tenant TZ helpers in `time/` when pack requires.
9. **Config / resources**: `application.yml` with `spring.application.name: lpapi-sales-bff`, Feign timeouts, Resilience4j instances, domain base URL. Profile yml files + `log4j2.xml`. No context-path unless contract requires one.
10. **Tests**: Golden-fixture / byte-exact JSON when frozen. Controller + service + util unit tests mirroring package layout under `src/test/java`. JaCoCo controller ≥0.90.
11. **Feature adaptation**: Implement only `legacy_bff` / `lpapi` inventory endpoints. If layer absent → `SKIP_LEGACY_BFF.md` and exit. Rename `{Feature}` prefixes; never invent new legacy fields.

## Pitfalls
- NEVER create `sales-lpapi-bff-service`, `lpapi-bff-service`, or any nested Boot module inside `lpapi-sales-bff-src` — this repo **is** `lpapi-sales-bff`
- NEVER generate Domain or Web BFF code in an LPAPI-only pass
- NEVER modernize frozen wire (no ApiResponse HTTP wrapper, camelCase renames, or REST cleanup) when parity is frozen
- NEVER add JPA, repositories, or domain business rules in LPAPI BFF
- NEVER use Web BFF grid/`ApiResponse` shapes on frozen LPAPI HTTP responses
- NEVER skip golden-fixture tests when contract marks frozen
- NEVER put LPAPI routes behind a modern `/api/sales/v1` context unless the frozen contract says so
- NEVER ignore `_adr_index` and pick random ADR files — index first, LPAPI paths only
- NEVER field `@Autowired`; NEVER omit Feign resilience annotations
- NEVER commit secrets; use env placeholders only
