# Refactor guide: Java + Spring Boot

## Verify

- Gradle/Maven: compile, unit test, static analysis (SpotBugs, PMD) if enabled; `integrationTest` (or equivalent) when persistence or full Spring context changes.

## Refactor-specific notes

- **Bean** renames: search `@Bean` names, `@Qualifier`, and `@ComponentScan` packages when moving classes.
- **JPA** — entity renames need **migrations** and updates to all **JPQL** and **native** queries; watch second-level **cache** and `serialVersionUID` if splitting DTO vs entity (rare).
- **Public REST** — DTOs, `produces` / `consumes`, and **OpenAPI** if you publish a spec.

## Pitfalls

- **@Transactional** boundaries: moving a method between transactional and non-transactional services changes **rollback** behavior. Tests that run every method in a test-only transaction can **mask** real transaction edges.
