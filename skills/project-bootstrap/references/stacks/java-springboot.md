# Stack: Java + Spring Boot

## Detection

- `spring-boot` starters in `pom.xml` or `build.gradle(.kts)`.
- `@SpringBootApplication` in `src/main/java`.

## Read first

`README.md`, Gradle/Maven file, `src/main/resources/application*.yml` or `.properties`, Flyway/Liquibase location, `Dockerfile` / `docker-compose*`.

## Trace one flow

`@RestController` (or webflux handler) → `@Service` → **repository** or client. Note **DTO vs entity** mapping, validation (`@Valid` / Bean Validation), exception handlers (`@ControllerAdvice`).

## `CLAUDE.md` sections to generate

1. **Quick Reference** — `./mvnw` or `./gradlew` (`build`, `test`, `integrationTest`, `bootRun`), static analysis and dependency check tasks if present.
2. **Read first** — main application, profiles, migration dir.
3. **Architecture** — **layering**; package-by-feature layout; no repository in controller; events vs direct calls between modules; MapStruct or manual mapping as found.
4. **Code style** — Java version, records vs DTOs, Lombok policy if any, Javadoc for public service methods if team does it.
5. **Security** — Spring Security if present, OAuth2/JWT, method security, CORS, actuator exposure.
6. **Persistence** — JPA rules, transaction boundaries, lazy loading pitfalls.
7. **Testing** — JUnit 5, Mockito, `@SpringBootTest`, Testcontainers if used.
8. **Pitfalls** — N+1, `FetchType.EAGER` abuse, `@Transactional` on wrong layer.
9. Conflict / stop-and-ask line.

## Stack red flags

- JPA entities returned directly from REST controllers.
- `Optional.get()` without isPresent, or `null` returns where API promises Optional.
- Heavy logic in `@Scheduled` or listeners without idempotency notes (call out if business-critical).

## Exemplar (ai-enablement)

`templates/java-springboot/CLAUDE.md`
