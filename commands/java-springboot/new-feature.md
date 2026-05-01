Scaffold a new feature module. Feature name: `$ARGUMENTS`.

Follow the project's layered, package-by-feature convention. Use the existing
`users` module as the reference template — copy patterns, naming, and
annotation usage exactly.

1. **Pick the base package** from `build.gradle` `group` + project structure
   (e.g. `com.acme.<feature>`). Ask if ambiguous.

2. **Create the production source tree** under
   `src/main/java/<base>/<feature>/`:
   - `api/<Feature>Controller.java` — REST controller with CRUD endpoints,
     each annotated with `@PreAuthorize(...)` and `@Valid` on the DTO.
   - `api/dto/<Feature>Request.java`, `<Feature>Response.java` — as Java `record`s
     with Jakarta validation annotations.
   - `domain/<Feature>Service.java` (interface) + `domain/<Feature>ServiceImpl.java`.
   - `domain/<Feature>.java` — entity class with private setters.
   - `domain/<Feature>NotFoundException.java` — extends the project's
     `DomainException` sealed base.
   - `infra/<Feature>Repository.java` — `JpaRepository<..., UUID>` interface.
   - `infra/<Feature>Mapper.java` — MapStruct `@Mapper(componentModel = "spring")`.

3. **Create the test sources**:
   - `src/test/java/<base>/<feature>/domain/<Feature>ServiceImplTest.java`
     — JUnit 5 + Mockito + AssertJ.
   - `src/test/java/<base>/<feature>/api/<Feature>ControllerTest.java`
     — `@WebMvcTest` + `MockMvc`.
   - `src/integrationTest/java/<base>/<feature>/<Feature>IntegrationTest.java`
     — `@SpringBootTest` + Testcontainers, covers happy path + one auth failure.
   - Test factory at `src/test/java/<base>/<feature>/<Feature>TestFactory.java`.

4. **Create the Flyway migration** at
   `src/main/resources/db/migration/V{yyyyMMddHHmm}__create_<feature>.sql`
   with columns `id UUID PRIMARY KEY, created_at, updated_at, deleted_at, version`.

5. **Verify** — `./gradlew compileJava compileTestJava spotlessApply`.
   Report any compile errors before finishing. Do NOT run migrations yet.

6. **Summary** — list every file created and the next manual steps
   (wire security rules, add to OpenAPI, run integration tests).

Respect the architecture rules in CLAUDE.md. Do not inject repositories into
controllers; do not skip layers.
