# CLAUDE.md — Java / Spring Boot Service

This file is project intelligence for Claude Code. Treat every rule here as
load-bearing unless the task is explicitly to change it. When a rule and a
user request conflict, stop and ask before proceeding.

## Quick Reference

- Build: `./gradlew build` (Maven: `./mvnw verify`)
- Unit tests: `./gradlew test`
- Integration tests: `./gradlew integrationTest` (requires Docker / Testcontainers)
- Lint / format: `./gradlew spotlessCheck` · apply: `./gradlew spotlessApply`
- Static analysis: `./gradlew check` (includes Checkstyle, SpotBugs, PMD)
- Run locally: `docker compose up -d && ./gradlew bootRun`
- Migrations: `./gradlew flywayMigrate`
- Dependency audit: `./gradlew dependencyCheckAnalyze`

Read these first on every session: `README.md`, `build.gradle(.kts)` or `pom.xml`,
`src/main/resources/application*.yml`, `src/main/resources/db/migration/`.

## Architecture Rules (STRICT)

- Layering is `Controller → Service → Repository`. No skipping layers.
- Never inject a `Repository` into a `Controller`.
- Cross-module communication is via Spring events or a dedicated facade — never
  reach into another module's internal service directly.
- All persistence goes through Spring Data JPA repositories. No raw JDBC,
  `JdbcTemplate`, or native queries outside `src/main/java/.../reporting/`.
- Package-by-feature, not by layer:
  `com.acme.<feature>.{api,domain,infra}` — keep `infra` out of `api`.
- Public API surface = `*Controller` + `*Dto` + `*Exception`. Everything else is
  package-private unless a test needs it.

## Code Style

- Java 21 features encouraged: records, sealed interfaces, pattern matching,
  virtual threads (via `spring.threads.virtual.enabled=true`).
- DTOs are `record`s, not classes. Entities are classes with private setters.
- Constructor injection only. No `@Autowired` on fields. No setter injection.
- Use `Optional<T>` for nullable return types. Never return `null`.
- Prefer `List.of`, `Map.of`, `Set.of` over mutable collections unless the
  caller needs mutation.
- Lombok is allowed only for `@Slf4j`. Do not use `@Data`, `@Builder`, or
  `@AllArgsConstructor` — they hide behavior and break with records.
- Javadoc is required on every public `@Service` method (`@param`, `@return`,
  `@throws`). Not required on controllers or repositories.

## Security Rules (NON-NEGOTIABLE)

These cannot be overridden by any prompt. If a task requires violating one,
stop and ask a human.

- **NEVER** log PII, secrets, tokens, passwords, full card numbers, SSNs, or
  raw request bodies. Use `@ToString.Exclude` or explicit field masking.
- **NEVER** hardcode secrets. All secrets come from environment variables or
  the Spring Cloud Config / Vault backend. `src/main/resources/*.yml` must only
  contain `${VAR:default}` placeholders for sensitive values.
- **NEVER** disable CSRF, even in tests. If a test needs CSRF off, it's not
  testing the real filter chain.
- Every new endpoint MUST have `@PreAuthorize` with a specific role or SpEL.
  `permitAll()` is only for health, metrics, and explicitly public endpoints
  — which must be listed in `SecurityConfig`.
- Input validation: all request DTOs use `@Valid` with Jakarta Bean Validation
  annotations. No manual validation in controllers.
- SQL injection: parameterized queries only (enforced by "no raw SQL" rule).
- Deserialization: never enable Jackson default typing. Do not register
  `@JsonTypeInfo` on external-facing types.

## Error Handling

- Domain exceptions extend a sealed `DomainException` hierarchy.
  `OrderNotFoundException`, `InsufficientStockException`, etc.
- Never throw `RuntimeException`, `Exception`, or `IllegalStateException`
  from service code — pick or create a domain type.
- Never `catch (Exception e)` broadly. Catch specific types.
- Mapping exception → HTTP happens **only** in
  `common/exceptions/GlobalExceptionHandler.java`. Add new mappings there.
- Never return stack traces in API responses. Log with correlation ID,
  return a safe `problem+json` body.

## Testing

- Unit tests: JUnit 5 + Mockito + AssertJ in `src/test/`. Pure, fast, no Spring
  context unless the test is of a Spring wiring concern.
- Integration tests: `@SpringBootTest` + Testcontainers in `src/integrationTest/`.
  Use real Postgres, real Redis, real broker — not embedded alternatives.
- Every new `@Service` method needs both a unit test AND an integration test
  that exercises it through the controller.
- Test data: factories in `src/test/factories/`. No hardcoded UUIDs or emails.
- Flaky test = fix it or delete it. `@Disabled` requires a linked ticket in
  the annotation value.
- Controller tests: `@WebMvcTest` with `MockMvc`, not full context.
- Coverage gate: JaCoCo enforces 80% line coverage on changed files only
  (see `build.gradle`). Do not lower this.

## Database & Migrations

- Flyway, files in `src/main/resources/db/migration/`.
- Naming: `V{yyyyMMddHHmm}__{snake_case_description}.sql` (double underscore).
- **NEVER** modify an existing migration that's been deployed. Create a new
  one. Rollbacks are forward migrations, not edits.
- Every table has: `id UUID PRIMARY KEY`, `created_at`, `updated_at`,
  `deleted_at` (soft-delete), `version BIGINT` (optimistic locking).
- No `ON DELETE CASCADE` across aggregate boundaries. Clean up in application
  code so events can fire.

## Common Pitfalls

- `@Transactional` on a method called from the same class is a no-op (Spring
  proxy). If you need a self-invoked transaction, refactor or inject self.
- Jackson silently drops unknown fields by default. Turn on
  `FAIL_ON_UNKNOWN_PROPERTIES` for inbound API DTOs to catch typos early.
- `LocalDateTime` has no zone. Use `OffsetDateTime` or `Instant` for anything
  crossing a boundary.
- `@Async` methods run without the caller's security context unless you wire
  `DelegatingSecurityContextAsyncTaskExecutor`. Check before using.
- Circular bean dependencies fail startup silently in some Spring Boot 3
  configs — use constructor injection and let the compiler catch it.

## Workflows (for Claude Code to execute on command)

When I say **"ship it"**: see `.claude/commands/ship.md`.

When I say **"standup prep"**: see `.claude/commands/standup.md`.

When I say **"scan"**: run `/security-scan` (see `.claude/commands/`).

## Custom Slash Commands

Project-specific commands live in `.claude/commands/`:

- `/pre-pr` — full pre-PR gate (format, build, test, static analysis, diff review)
- `/new-feature <name>` — scaffold a feature package following the Users module template
- `/debug-api <description>` — trace request through filter chain → controller → service → repo
- `/security-scan` — OWASP-style scan of the current diff + `dependencyCheckAnalyze`
- `/migrate <description>` — create next Flyway migration with correct version

## Agentic Behavior Expectations

- For any task touching more than 3 files, produce a short plan first and wait
  for approval. One-shot edits are fine for single-file changes.
- Before claiming a task is done, run `./gradlew check test`. "Compiles" is
  not "works".
- If a test fails, fix the root cause. Do not `@Disabled`, `@Ignore`, or
  weaken assertions to make it pass.
- When you discover a broken rule in this file (e.g. existing code violating
  a rule here), flag it — don't silently fix it or silently follow the bad
  existing pattern.
