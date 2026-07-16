---
name: "orbt-web-bff-code-pattern"
description: "Generates ORBT Web/authored BFF Java matching client-reviewed orbt-sales-bff-src: thin Feign to domain, ApiResponse envelope, RBAC, multi-tenant TenantParallelExecutor, Resilience4j on Feign, MapStruct wire adapters, Log4j2, OAuth2/Redis. Use when generating or fixing Web BFF from story packs / ORBT features."
version: 1
created: "2026-07-16"
updated: "2026-07-16"
---
## When to Use
Use when WebBffCodegen / TestAuthor / BuildFix / GapFix generate or fix the authored/modern Web BFF module from a story pack. Apply for any ORBT Web BFF feature so output matches client-reviewed `orbt-sales-bff-src` structure and strategies. Story pack / inventory = WHAT (endpoints, wire fields, domain mappings). This skill = HOW. Prefer this over generic Spring BFF templates when they conflict. Companions: `bff-service-coreframework-guidelines`, `java21-springboot4-best-practices`, `secrets-management-azure-key-vault`, `security-http-headers-spring-boot`.

## Procedure
1. **Package layout (mandatory)**: Root `com.renuity.sales` (or inventory base). Create: `*ServiceApplication`, `client/`, `config/`, `constant/`, `controller/`, `dto/request/`, `dto/response/` (include `ApiResponse<T>`, `ErrorDetail`, `TenantError`), `dto/domain/` (Feign/domain wire DTOs), `exception/`, `function/` (Azure cache invalidation when pack requires), `mapper/`, `service/`, `service/impl/`, `utils/`. NEVER add `model/`, `repository/`, Liquibase, or DataSource config.
2. **Boot / framework**: Parent `orbt-common-parent-lib`. Deps: `core-framework-common`, `core-framework-web`, `core-framework-data`, `core-framework-rbac`, OpenFeign, Resilience4j (circuitbreaker/retry/bulkhead/feign), springdoc-openapi, OAuth2 client+resource-server, Redis (Azure Managed Redis + Entra ID when ADRs require), `jjwt` for token parsing. Exclude logback-classic everywhere. `@SpringBootApplication` + `@EnableFeignClients(basePackages={"com.renuity.core.web.client","com.renuity.<domain>.client"})` + `@EnableCaching` + `@ComponentScan` app root + `com.renuity.core.{common,data,web,rbac}`. Virtual threads on. Context path `/api/sales` (or inventory domain). Spring Cloud Config optional import.
3. **Controllers**: `{Feature}BffController` per feature. `@RestController @RequestMapping("/v1/{resource}") @RequiredArgsConstructor @Validated @Slf4j` + OpenAPI `@Tag`. `@RbacAuthorization` with `@AccessLevel` (ROLE/PAGE/FEATURE+WRITE) per pack. Methods: rich `@Operation`/`@ApiResponses` with `@ExampleObject`, `@Parameter`/`@Schema`, `@Valid` body, `@NotBlank`/`@Pattern` on query params, optional `X-Correlation-Id`. Thin only: log, delegate to `{Feature}BffService`, `ResponseEntity.ok(apiResponse)`. NEVER business logic or Feign calls in controllers.
4. **DTOs**: BFF wire records in `dto.request|response` with `@JsonProperty` + `@Schema` + Jakarta validation. Domain Feign DTOs in `dto.domain` (may use `@Data`/`@Builder` like reference). Envelope `ApiResponse<T>`: status, statusCode, message, data, success, errors, optional partialSuccess + tenantErrors; static `success`/`error`/`partialSuccess`/`errorWithDetails` factories.
5. **Services**: Interface `{Feature}BffService`, impl `{Feature}BffServiceImpl` in `service.impl`. `@Service @RequiredArgsConstructor @Slf4j`, constructor injection. Orchestrate Feign clients + mappers only — no persistence. Multi-tenant reads/writes: `TenantParallelExecutor.execute(TenantContextHolder.getTenantIds(), tenantId -> client.call(tenantId,...))` then `MultiTenantAggregationUtil.aggregate(...)` or private aggregation for writes. Return `ApiResponse.partialSuccess` when some tenants fail. Use `CorrelationIdHolder` for tracing.
6. **Feign clients**: `@FeignClient(name="orbt-<domain>-domain-src", url="${<domain>.domain.newurl}", configuration=FeignConfig.class)`. Per-method `@CircuitBreaker` + `@Retry` + `@Bulkhead` with named instances + **default fallback methods** returning degraded `ApiResponse` (empty list or failure DTO, never throw). Pass `X-Tenant-Id` header on every domain call. Domain paths like `/api/sales/v1/...`. Separate client per domain aggregate when reference does (e.g. `IssueClient` vs `SalesDomainClient`).
7. **Mappers**: MapStruct `@Mapper` interface + hand-maintained or generated impl for BFF↔domain DTO transforms and grid assembly (aggregates, footer totals). No business rules in mappers.
8. **Exceptions**: `GlobalExceptionHandler` `@RestControllerAdvice` → `ResponseEntity<ApiResponse<Void>>` with `ErrorDetail` list. Handle validation, Feign 4xx/5xx, RBAC, `BusinessException`, `ServiceUnavailableException`, `TenantNotFoundException`. MDC `correlation_id` via framework; never invent alternate scheme. `SecurityExceptionHandler` for auth failures when present.
9. **Resilience / tenant**: `MultiTenantResilience4jConfig` — tenant-scoped circuit breaker/rate limiter registry via `TenantContextHolder`. Resilience4j **only on BFF Feign**, never in domain module.
10. **Security / cache**: OAuth2 Azure AD JWT resource server; RBAC annotations on controllers. Redis caching + `CacheInvalidationService` / Azure Function hook when pack requires. Never log tokens/secrets.
11. **Logging**: SLF4J + Log4j2 only (ECS JsonTemplateLayout in prod profiles). Parameterized `@Slf4j` logs at info for operations, warn for partial tenant failures.
12. **Tests**: Controller unit tests `@ExtendWith(MockitoExtension)` + `@Mock` service + `@InjectMocks` controller (no `@WebMvcTest` required if reference uses plain Mockito). Service tests mock Feign + `TenantParallelExecutor` + mapper. Multi-tenant paths: dedicated `*MultiTenantTest` classes. WireMock for Feign integration when needed. Target controller ≥0.90 JaCoCo per workflow gates.
13. **Feature adaptation**: Rename Issue*/SalesSchedule* → `{Feature}*` from inventory. Keep controller/service/client/mapper/test styles. Map domain endpoints from `endpoint_inventory.json` web_bff layer only.

## Pitfalls
- NEVER add JPA, `@Entity`, `JpaRepository`, Liquibase, or `DataSource` in Web BFF
- NEVER put business rules in BFF — delegate to domain via Feign
- NEVER field `@Autowired`; NEVER skip Resilience4j annotations on Feign methods
- NEVER omit `ApiResponse` envelope, `GlobalExceptionHandler`, or RBAC on secured endpoints
- NEVER change authored Web wire shapes — only adapt field names from inventory/contracts
- NEVER default tenant to `1`; use `TenantContextHolder` or explicit tenant params
- NEVER commit secrets; use env placeholders only
