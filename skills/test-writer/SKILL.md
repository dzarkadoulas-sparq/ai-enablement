---
name: test-writer
description: >-
  Produces idiomatic unit and integration tests: detects framework and conventions,
  infers behaviors from target code, and outputs tests in new-code, TDD, characterization,
  or coverage-gap mode. Uses project factories and fixtures, runs the suite, and reports
  coverage changes. Use when the user asks to write tests, add coverage, TDD, characterization
  tests, or to increase test coverage.
---

# Test writer

## Goal

Add **correct, maintainable** tests that match the repo’s **framework, folder layout, and patterns**. Prefer **behavior** and **public contracts** over testing implementation details. Always **run the test command** after adding or changing tests and report pass/fail (TDD: expect **red** before implementation).

## When stack is unknown

1. **Detect** language and test stack (manifests, existing `**/*.test.*`, `**/*Test.java`, `**/*Tests.cs`, `conftest.py`, `vitest.config.*`, `pytest.ini`, etc.).
2. **Read exactly one** stack file: `references/stacks/<id>.md` — it points to the right **language** references below.
3. **Always read** (unless user scope is only one area):
   - `references/edge-case-catalog.md` — boundaries and domain edge cases
   - `references/anti-patterns.md` — what not to test, brittle patterns

| `<id>` | Typical test stack | Also read (language / UI) |
| ------ | ------------------ | ------------------------- |
| `react`           | Vitest + RTL + (msw)                                      | `references/react-typescript.md`                                  |
| `next`            | Vitest + RTL; RSC/Server Actions                          | `references/react-typescript.md` + `references/nextjs-testing.md` |
| `angular`         | Karma/Jest + TestBed (+ Playwright e2e elsewhere)         | `references/angular-testing.md`                                   |
| `node-express`    | Vitest/Jest + `supertest` (or project choice)             | `references/nodejs-api-testing.md`                                |
| `python-fastapi`  | `pytest` + `httpx` / Starlette `TestClient`               | `references/python-pytest.md`                                     |
| `java-springboot` | JUnit 5 + Mockito + Spring Boot Test                      | `references/java-junit5.md`                                       |
| `dotnet`          | xUnit + Fluent Assertions + Moq + `WebApplicationFactory` | `references/csharp-xunit.md`                                      |

## Modes (pick from user request or ask once)

1. **New code** — Happy path, error paths, edge/boundary conditions; assert observable outcomes.
2. **TDD** — **One failing test** first, minimal **green** implementation, then refactor. Do not add implementation before the red test is written and run.
3. **Legacy / characterization** — Document **current** behavior in tests without judging correctness; name tests `it('documents that ...')` or equivalent when the behavior is surprising.
4. **Coverage gap** — If coverage data exists, target **highest-risk** untested code (auth, money, public API, migrations). If not, use diff-based: test what changed and its direct collaborators.

## Workflow (execute in order)

1. **Scope** — File(s), class, or behavior under test. Resolve **entry points** (exported function, public method, route handler, component).
2. **Inventorize conventions** — Existing test file naming, `describe` structure, `beforeEach`/`fixtures`/`@BeforeEach`/`IClassFixture`, test doubles style (strict mocks vs fakes), MSW or wiremock usage.
3. **Behaviors** — List **inputs → outputs**, errors, idempotence, side effects, auth, time, I/O. Use `edge-case-catalog.md` for the domain.
4. **Write tests** — Reuse project **factories/builders**; if none, add small helpers in test scope only. Match **file location** in the stack file.
5. **Run** — Execute the project’s unit test command from manifests (`package.json` scripts, `./gradlew test`, `dotnet test`, `pytest`, etc.). Fix flakiness (time, random, order dependence).
6. **Report** — List files added/updated, mode used, test run result, and **coverage** delta if a coverage command exists.

## Exemplar repo (ai-enablement)

Stack-specific commands and colocation often mirror **`claude.md/<id>/CLAUDE.md`** and **`commands/<id>/pre-pr.md`**.

## Anti-patterns

- **Snapshot abuse** for logic that should be assertive.
- **Real network/DB** in unit tests when the project uses fakes; **mocking the system under test**.
- **Giant** setup blocks — extract builders or `test-data` module per project style.
- **Flaky** tests without `fake timers` / deterministic clock or isolation.

## Reference map

- `references/stacks/<id>.md` — where tests live, commands to run, stack quirks.
- `references/edge-case-catalog.md`, `references/anti-patterns.md` — always-on.
- Language files: `react-typescript.md`, `nextjs-testing.md`, `angular-testing.md`, `nodejs-api-testing.md`, `python-pytest.md`, `java-junit5.md`, `csharp-xunit.md`.

Valid `<id>` values: `react`, `next`, `angular`, `node-express`, `python-fastapi`, `java-springboot`, `dotnet`.
