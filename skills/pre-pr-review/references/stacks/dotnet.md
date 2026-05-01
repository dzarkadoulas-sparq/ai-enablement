# Pre-PR gate: .NET / ASP.NET Core

## Baseline branch

`origin/develop` or repo default. `git diff origin/develop...HEAD`.

## Gate order

1. **Format** — `dotnet format` then `dotnet format --verify-no-changes` (or project standard).
2. **Build** — `dotnet build -c Release -warnaserror --no-restore` (or stricter; match CI).
3. **Unit tests** — `dotnet test` with `Category!=Integration` and collection for coverage.
4. **Integration tests** — `Category=Integration`. If Docker isn’t running, **stop and report**.
5. **Coverage** — e.g. `reportgenerator` or team tool; threshold on **changed** lines (e.g. 80% if that’s the bar).
6. **Dependencies** — `dotnet list package --vulnerable --include-transitive` and `dotnet list package --deprecated`; flag HIGH/CRIT on **touched** packages.
7. **Diff hygiene** — see **Diff hygiene (.NET)** below.
8. **Summary**. **Do not push or open PR** unless asked.

## Diff hygiene (.NET)

- `Console.WriteLine` / `Debug.WriteLine` in production code.
- `[Fact(Skip = ...)]` / `[Theory(Skip = ...)]` without ticket.
- Commented-out code; hardcoded URLs, IPs, connection strings, or secret-like values.
- `#pragma warning disable` without `// Reason:` on the line above.
- New endpoints: `[Authorize]` (unless explicitly `[AllowAnonymous]`) and validation for DTOs if that’s the standard.
- `FromSqlRaw` with string interpolation; unsafe dynamic SQL.

## Exemplar

`commands/dotnet/pre-pr.md`
