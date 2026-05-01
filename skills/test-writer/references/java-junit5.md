# JUnit 5 + Mockito + Spring Boot Test

## Unit tests (pure Java)

- **JUnit 5** — `@Test`, `@ParameterizedTest` + `ArgumentsSource` / Csv; `@DisplayName` for clarity.
- **AssertJ** (if in classpath) or JUnit `assertions` / AssertJ style—**match the project**.
- **Mockito** — `@ExtendWith(MockitoExtension.class)` + `@Mock` or `mock()`. **Strictness** as team configures.

## Slices (Spring)

- **`@WebMvcTest(controllers = X.class)`** — test controller + JSON mapping + validation; **mock** `@MockBean` services.
- **`@DataJpaTest`**, **`@JdbcTest`**, etc.—only for persistence slice; not for full app unless that’s the goal.

## Spring Boot test

- `@SpringBootTest` + `webEnvironment = RANDOM_PORT` or `MOCK` for **integration**; use **`TestRestTemplate`**, **`MockMvc`**, or **`@AutoConfigureWebMvc` + `WebTestClient`** (WebFlux) as existing tests do.
- **Transactions** — `@Transactional` on test class to roll back DB when appropriate (watch lazy-loading behavior).

## Testcontainers

- If the project uses **@Testcontainers** and Docker for integration tests, do **not** add parallel suites that **starve** Docker; follow `integrationTest` task from Gradle.

## Async / time

- **Awaitility** (if in deps) for async assertions; or **virtual threads** / executor patterns per Spring Boot 3+.

## Naming

- `MethodName_Should_Expected_When_Condition` or **Given_When_Then**—**mirror existing** test class style.

## Anti-patterns

- **Mocking** DTOs or data classes with no behavior.
- **Asserting** on `Optional.get()` without `isPresent` — use `assertThat(optional).isPresent()` + `get()`.
