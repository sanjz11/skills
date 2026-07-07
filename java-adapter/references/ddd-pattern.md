# Java / Spring Boot — DDD mapping pattern

Map IR `boundedContexts[]` into a **Domain** layer that downstream Java artifacts reference.

## Per bounded context

| IR element | Java artifact | Notes |
|------------|---------------|-------|
| boundedContext | `package` root | e.g. `com.example.salesoperations` |
| aggregates[] | `@AggregateRoot` classes | One class per aggregate; enforce invariants inside root |
| entities[] | JPA `@Entity` or domain entity | Belongs to exactly one aggregate unless read-model |
| valueObjects[] | immutable record/class | No identity; equality by value |
| domainEvents[] | Spring ApplicationEvent or outbox payload | Published after successful transaction |
| capabilities[] | application service methods | Orchestrate use cases; delegate to aggregates |
| businessRules[] | domain service or aggregate method | Map BR-ID to enforcing class/method |

## Aggregate rules

1. **Consistency boundary** — one transaction modifies one aggregate root unless saga documented.
2. **Reference by ID** — aggregates reference other aggregates by identifier, not object graph.
3. **Factory methods** — named constructors for complex aggregate creation.
4. **Repository per aggregate root** — `XxxRepository` interface in domain; JPA impl in infrastructure.

## Required Domain artifact sections

- `boundedContexts` — id, name, package, description
- `aggregates` — name, rootEntity, entities, valueObjects, invariants, domainEvents
- `domainServices` — cross-aggregate rules not belonging to a single root
- `applicationServices` — map 1:1 to IR workflows/capabilities where practical
- `repositories` — interface name, aggregateRoot, query methods from IR read models

## Traceability

Every aggregate and application service must cite IR ids: `boundedContextRef`, `capabilityIds`, `businessRuleIds`.
