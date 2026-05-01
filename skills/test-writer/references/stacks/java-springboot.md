# Test writer: Java + Spring Boot

## Language references

- `references/java-junit5.md` — JUnit 5, Mockito, **@WebMvcTest** / **@SpringBootTest**, Testcontainers.
- `references/edge-case-catalog.md`, `references/anti-patterns.md`

## Detect / locate tests

- `src/test/java` mirroring `main` packages; JUnit 5, AssertJ, Mockito as in `build.gradle` / `pom.xml`.
- **Integration** tests under `src/integrationTest` or `*IT.java` if the project uses that layout.

## Commands

- Unit + compile: e.g. `./gradlew test` (exclude integration if only unit requested).
- Integration: `./gradlew integrationTest` when Docker/DB is required.

## Focus

- **Controller:** `@WebMvcTest` + **MockBean** for services; JSON content assertions.
- **Service:** unit with mocks; **repository** with **@DataJpaTest** or Testcontainers for SQL.

## Exemplar context

`templates/java-springboot/CLAUDE.md`
