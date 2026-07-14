---
name: "orbt-domain-service-code-pattern"
description: "Generates ORBT domain-service Java matching client-reviewed orbt-poc-sales-api: CQRS controllers, ApiResponse envelope, MapStruct, native JPA queries with projections, Liquibase, Log4j2, core-framework multi-tenant. Use when generating or fixing Domain-layer Spring Boot from story packs / ORBT features, or mirroring sales-api patterns for any feature."
version: 1
created: "2026-07-14"
updated: "2026-07-14"
---
## When to Use
Use when DomainCodegen / Scaffold / GapFix / TestAuthor generate or fix a Domain module from a story pack. Apply for any ORBT domain feature so output matches client-reviewed `orbt-poc-sales-api` structure and strategies. Story pack / inventory = WHAT (endpoints, tables, SP SQL, fields). This skill = HOW. Prefer this over generic Spring templates when they conflict. Companions: `domain-service-coreframework-guidelines`, `java21-springboot4-best-practices`, `legacy-sql-to-modern-jpa-transformation`.

## Procedure
1. **Package layout (mandatory)**: Root from inventory/ADR (e.g. `com.orbt.<domain>`). Create exactly: `*DomainServiceApplication`, `config/` (JpaConfig + OpenApiConfig only — no SecurityConfig), `constants/`, `controller/`, `dto/request/`, `dto/response/` (include `ApiResponse<T>`), `exception/`, `mapper/`, `model/` (not entity/), `repository/`, `repository/projection/`, `service/`, `service/impl/`, `utils/`.
2. **Boot / framework**: Parent `orbt-common-parent-lib`. Deps: `core-framework-common|data|domain` only. NEVER add `core-framework-web` or `jjwt`. Exclude logback-classic everywhere. `@SpringBootApplication(exclude={HibernateJpaAutoConfiguration, DataSourceAutoConfiguration})`. `@ComponentScan` domain root + `com.renuity.core.{domain,common,data}`. `@EnableCaching`. Virtual threads on. Context path `/api/<domain>`. OpenApiConfig `@Profile({"dev","qat"})` only.
3. **CQRS controllers**: `{Feature}CommandController` (writes) + `{Feature}QueryController` (reads). Shared `@RequestMapping("/v1/{resource}")`. Class: `@Slf4j @Validated @RestController` + OpenAPI `@Tag`. Methods: `@Operation`/`@ApiResponses`, `@Valid` body, param constraints, optional `X-Tenant-Id`. Thin only: resolve tenant, log key=value, delegate, `ResponseEntity.ok(ApiResponse.success(...))`. NEVER put business logic in controllers.
4. **DTOs**: Java records in `dto.request|response`. EVERY field: `@JsonProperty` + `@Schema`. Jakarta validation on request (`@NotNull`/`@Min`/`@NotBlank`/`@Pattern`/`@Valid` nested). Envelope always `ApiResponse<T>` with status, statusCode, message, data, success, errors + static success/error factories.
5. **Services**: Interface in `service`, impl in `service.impl`. `@Service @Slf4j`, constructor injection, `private final` fields. Writes `@Transactional`; reads `@Transactional(readOnly=true)`. Pass `tenantId` as method arg. Structured logs; never tokens/secrets. Domain events via record + `@TransactionalEventListener(AFTER_COMMIT)` when pack requires.
6. **Repositories / JPA**: `@Repository` extends `JpaRepository`. Complex/legacy SQL → `@Query(nativeQuery=true)` text blocks with CTE `params AS (SELECT :tenantId ::INTEGER AS tenant_id, ...)`, named `@Param`, `@QueryHints` timeout, interface projections under `repository.projection`. DML: `@Modifying` + `@Transactional(propagation=MANDATORY)`. ALWAYS tenant bind + WHERE tenant_id. NEVER empty stub repos when pack SQL/SP exists. NEVER raw user ORDER BY concat — whitelist/CASE.
7. **Entities (`model`)**: `@Entity @Table(schema,name) @Column(snake_case)`. `@Getter @Setter` — NEVER `@Data`. equals/hashCode on `@Id` only. Relationships `FetchType.LAZY`. `tenant_id` nullable=false when tenant-scoped.
8. **MapStruct**: `@Mapper(componentModel="spring")`. Prefer Projection → ResponseDto on hot paths. No hand-written mappers.
9. **Exceptions**: Domain `RuntimeException` + `public static final String ERROR_CODE`. `GlobalExceptionHandler` `@RestControllerAdvice` returns `ResponseEntity<ApiResponse<Void>>`. Use MDC `correlation_id` (framework); never invent alternate MDC. Never HTTP 200 for errors; never stack traces in body. Empty lists return SUCCESS (not 404).
10. **Tenant**: Header or `TenantContextHolder`; fail-fast if missing/invalid — NEVER default to `1`. SQL tenant predicate AND routing DataSource (not either alone). Clear ThreadLocal in finally if set manually.
11. **Liquibase**: `db/changelog/db.changelog-master.xml` + `changesets/NNN-*.xml`. Additive; preConditions onFail=MARK_RAN; explicit rollback; schemaName set. ddl-auto: validate.
12. **Logging**: SLF4J + Log4j2 only (prod ECS JsonTemplateLayout). Never Hibernate SQL DEBUG/TRACE in prod.
13. **Tests**: Controller `@WebMvcTest` + TestSecurityConfig/TestRedisConfig + `@MockitoBean`. Service `@ExtendWith(MockitoExtension) @Mock @InjectMocks`. Repo IT: Testcontainers PostgreSQL — NEVER H2.
14. **Feature adaptation**: Rename LeadIssue* → {Feature}* from inventory. Keep CQRS/packages/annotations/envelope/test styles. Replace SQL with pack/SP-derived SQL via JPA transform skills.

## Pitfalls
- NEVER add core-framework-web, jjwt, Logback, H2, or Resilience4j in domain
- NEVER field @Autowired; NEVER @Data on entities; NEVER cache JPA entities
- NEVER skip unread SP/SQL or leave repository TODOs/stubs
- NEVER omit ApiResponse envelope, CQRS Command/Query split, MapStruct, or GlobalExceptionHandler
- NEVER expose Swagger UI outside dev/qat profiles
- NEVER commit secrets; use env placeholders only