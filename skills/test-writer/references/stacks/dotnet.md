# Test writer: .NET + ASP.NET Core

## Language references

- `references/csharp-xunit.md` — xUnit, Moq, Fluent Assertions, `WebApplicationFactory`.
- `references/edge-case-catalog.md`, `references/anti-patterns.md`

## Detect / locate tests

- `*Tests.csproj` or `tests/**`; **xUnit** by convention; `WebApplicationFactory` in integration tests.
- **Verify** or snapshot libs if present—follow project naming.

## Commands

- `dotnet test --no-build` or with filter `Category!=Integration` / `Category=Integration` per pre-PR.
- Collect coverage with team’s `--collect` (e.g. XPlat) if the user asked for coverage numbers.

## Focus

- **Minimal API** / `Results.*` **integration** through `CreateClient()`.
- **MediatR handlers** — test with mocked `I*Repository` or in-memory fakes.
- **No** `.Result` in async tests; **use** `await` and `FluentAssertions` async.

## Exemplar context

`claude.md/dotnet/CLAUDE.md`
