# Repository pattern (Java / Spring Boot)

Map IR `data.entities` + `dataEntities[]` → JPA repository design.

## Structure

```java
public interface {Entity}Repository extends JpaRepository<{Entity}, UUID> {
  // query methods from IR access patterns only
}
```

## Rules

- Entity class maps IR attributes to columns (conceptual types → JPA types in design)
- Mapper interface between Entity and domain/DTO when IR has separate DTOs
- No SQL strings unless IR specifies native query
- `@Transactional` on service layer, not repository

## IR mapping

| IR field | Artifact |
|----------|----------|
| dataEntities[].name | Entity + Repository name |
| data.relationships | JPA relationship annotations in design |
| data.persistence.indexing | @Index or query method notes |
