# Controller pattern (Java / Spring Boot)

Map IR `apiOperations[]` → one `@RestController` per resource group (bounded context).

## Structure

```
@RestController
@RequestMapping("/api/v1/{resource}")
public class {Resource}Controller {
  private final {Resource}Service service;
  // constructor injection
}
```

## Rules

- One controller per aggregate/resource group in IR
- DTOs in `dto` package; `@Valid` on request bodies
- Map IR `errors[]` to `@ExceptionHandler` or problem-detail responses
- OpenAPI annotations optional in design pseudo-code; full spec in `openapi.yaml`

## IR mapping

| IR field | Artifact |
|----------|----------|
| apiOperations.method | HTTP method mapping |
| apiOperations.path | @RequestMapping / @GetMapping etc. |
| apiOperations.requestSchema | Request DTO class |
| apiOperations.responseSchema | Response DTO class |
| security.authorization | Method security annotations (design level) |
