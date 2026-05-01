# xUnit + Moq + Fluent Assertions (+ WebApplicationFactory)

## Unit tests

- **xUnit** — `Fact`, `Theory` + `InlineData` / `MemberData`. **Async** tests return `Task`.
- **Moq** — `Mock<T>` for interfaces; `Setup`, `Returns`, `ReturnsAsync` for `Task` methods; `Verify` sparingly.
- **Fluent Assertions** — `result.Should().NotBeNull().And...` (match `using` and style in repo).

## Web API integration

- **`WebApplicationFactory<TProgram>`** (minimal hosting) with **`CreateClient()`**; `withWebHostBuilder` to replace **DbContext** with **in-memory** or test DB per team pattern.
- **Test auth** — `WebApplicationFactory` + test **Handler** / fake JWT if project has `TestAuth` helpers—**do not** disable auth globally without explicit test intent.

## MediatR / vertical slices

- Test **handlers** with **mocks** of `IRepository` or **in-memory** fakes; assert **outcome** type and side effects.

## Data

- **In-memory** EF, **SQL Server test container**, or **shared** local DB with transaction rollback—**find existing** integration test and copy the pattern.

## Snapshots in .NET

- If the project uses **Verify** or custom snapshot libs, follow **naming** and `*.received` / `*.verified` flow.

## Anti-patterns

- **Static** `DateTime.Now` in SUT without abstraction—suggest `TimeProvider` / `IDateTimeProvider` if the code needs deterministic tests.
- **`.Result` / `.Wait()`** in test thread—**await** the async call.
