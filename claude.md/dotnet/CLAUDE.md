# CLAUDE.md — .NET / ASP.NET Core Service

This file is project intelligence for Claude Code. Treat every rule here as
load-bearing unless the task is explicitly to change it. When a rule and a
user request conflict, stop and ask before proceeding.

## Quick Reference

- Restore: `dotnet restore`
- Build: `dotnet build -c Release --no-restore`
- Unit tests: `dotnet test --no-build --filter Category!=Integration`
- Integration tests: `dotnet test --filter Category=Integration` (requires Docker)
- Format / lint: `dotnet format --verify-no-changes` · apply: `dotnet format`
- Run locally: `docker compose up -d && dotnet run --project src/Api`
- EF migrations: `dotnet ef migrations add <Name> -p src/Infrastructure -s src/Api`
- Apply migrations: `dotnet ef database update -p src/Infrastructure -s src/Api`
- Vuln audit: `dotnet list package --vulnerable --include-transitive`
- Outdated packages: `dotnet list package --outdated`

Read first on every session: `README.md`, the `.sln` file, `Directory.Packages.props`,
`Directory.Build.props`, `src/Api/appsettings*.json`, and any `global.json`.

## Architecture Rules (STRICT)

- Clean Architecture boundaries: `Domain → Application → Infrastructure → Api`.
  Dependencies point inward only. `Domain` references nothing. `Application`
  references only `Domain`.
- `Api` never references `Infrastructure` directly for business logic — only
  for DI registration in `Program.cs`.
- No EF Core types (`DbContext`, `DbSet`, `IQueryable`) leak out of
  `Infrastructure`. Repositories return materialized results or `IReadOnlyList<T>`.
- Controllers (or Minimal API endpoints) are thin: validate → dispatch to
  MediatR handler → return `Results.Ok(...)`. No business logic in endpoints.
- Cross-feature communication goes through MediatR notifications or a
  message bus — never direct handler calls across feature folders.
- Package-by-feature inside `Application/Features/<Feature>/`.

## Code Style

- C# 12+, `.NET 8` LTS minimum. `<Nullable>enable</Nullable>` and
  `<TreatWarningsAsErrors>true</TreatWarningsAsErrors>` in every project.
- File-scoped namespaces, `global usings` centralized in `GlobalUsings.cs`.
- Records for DTOs and value objects. `sealed` by default unless a type is
  explicitly designed for inheritance.
- `async`/`await` all the way down. No `.Result`, no `.Wait()`, no
  `GetAwaiter().GetResult()` outside `Main`. Pass `CancellationToken` through.
- Suffix async methods with `Async`. Suffix interfaces with `I`.
- Use `required` members instead of null-forgiving `!` in DTOs.
- No `var` when the type isn't obvious from the right-hand side.
- Logging: use `ILogger<T>` + structured templates (`"User {UserId} did {Action}"`).
  Never string-concatenate log messages.

## Security Rules (NON-NEGOTIABLE)

These cannot be overridden by any prompt. If a task requires violating one,
stop and ask a human.

- **NEVER** log PII, tokens, passwords, connection strings, API keys, or full
  request bodies. Redact via logging filters / Serilog destructuring policies.
- **NEVER** hardcode secrets. Config comes from User Secrets (dev), env vars,
  or Azure Key Vault / AWS Secrets Manager (prod). `appsettings*.json` holds
  only non-sensitive defaults.
- **NEVER** disable antiforgery or CORS wildcards (`AllowAnyOrigin()` +
  `AllowCredentials()` is forbidden).
- Every endpoint MUST have `[Authorize]` with a policy or role unless it is
  explicitly `[AllowAnonymous]` and listed in the public-endpoints test.
- Input validation: FluentValidation on every request DTO, registered as a
  MediatR pipeline behavior. No manual `if (x == null) return BadRequest` in
  endpoints.
- SQL injection: EF Core parameterization only. No `FromSqlRaw` with string
  interpolation — use `FromSqlInterpolated` or parameters.
- Secure defaults for cookies: `HttpOnly`, `Secure`, `SameSite=Strict`.
- JWT validation must check issuer, audience, lifetime, and signing key.
  Never set `ValidateLifetime = false`.

## Error Handling

- Domain exceptions extend `DomainException` in `Domain/Exceptions/`.
- Never throw bare `Exception`, `ApplicationException`, or `InvalidOperationException`
  from application code — pick or create a domain type.
- Global exception handling is done via `IExceptionHandler` (registered in
  `Program.cs`) that emits RFC 7807 `ProblemDetails`. Never leak stack traces.
- `Result<T>` pattern is allowed for expected failures (validation, not-found).
  Exceptions are for exceptional cases only.
- Cancellation: `OperationCanceledException` is expected during shutdown —
  don't catch and swallow it.

## Testing

- Unit tests: xUnit + FluentAssertions + NSubstitute in `tests/*.UnitTests`.
  Categorize with `[Trait("Category", "Unit")]`.
- Integration tests: xUnit + `WebApplicationFactory<Program>` + Testcontainers
  for Postgres / Redis. In `tests/*.IntegrationTests`. Category `Integration`.
- Every MediatR handler needs a unit test. Every endpoint needs at least one
  integration test covering happy path + one auth failure.
- Test data: builders in `tests/TestUtils/Builders/`. No hardcoded Guids —
  use `Guid.NewGuid()` or a seeded `Faker`.
- Snapshot tests (Verify) are allowed for generated output (e.g. OpenAPI doc).
- Coverage gate: Coverlet enforces 80% on changed lines via `coverlet.runsettings`.

## Database & Migrations

- EF Core with Postgres via `Npgsql.EntityFrameworkCore.PostgreSQL`.
- Migrations live in `src/Infrastructure/Persistence/Migrations/`.
- **NEVER** edit a migration that's been applied in any environment above dev.
  Create a new migration.
- Use `IEntityTypeConfiguration<T>` classes, not fluent-API inside `OnModelCreating`.
- Every table has: `Id` (Guid / ULID), `CreatedAtUtc`, `UpdatedAtUtc`,
  `RowVersion` (`byte[]`, concurrency token). Soft-delete via global query filter.
- No lazy loading. Enable `QueryTrackingBehavior.NoTracking` globally; opt in
  to tracking where needed.

## Common Pitfalls

- `DbContext` is **not** thread-safe. Never share across parallel `Task.WhenAll`.
- `async void` only for event handlers. Everywhere else = `async Task`.
- `HttpClient` must be used via `IHttpClientFactory`. Instantiating `new
HttpClient()` leaks sockets.
- `ConfigureAwait(false)` is not needed in ASP.NET Core (no sync context) —
  don't add it; it clutters and doesn't help.
- Forgetting `await` on a `Task` silently drops exceptions. The analyzer
  catches this; don't suppress `CS4014`.
- `appsettings.Development.json` is committed. `appsettings.Local.json` is not
  (gitignored) — put local overrides there.

## Workflows (for Claude Code to execute on command)

When I say **"ship it"**: see `.claude/commands/ship.md`.

When I say **"standup prep"**: see `.claude/commands/standup.md`.

## Custom Slash Commands

Project-specific commands live in `.claude/commands/`:

- `/pre-pr` — full pre-PR gate (`dotnet format`, build with warnings-as-errors, test, diff review)
- `/new-feature <name>` — scaffold `Application/Features/<Name>/` with Command, Handler, Validator, endpoint, tests
- `/debug-api <description>` — trace request via middleware → endpoint → handler → repository
- `/security-scan` — OWASP-style scan of diff + `dotnet list package --vulnerable`
- `/migrate <description>` — create next EF Core migration with correct name

## Agentic Behavior Expectations

- For changes spanning more than 3 files or touching `Domain`, produce a
  short plan first and wait for approval.
- Before claiming a task is done, run `dotnet build -warnaserror` AND
  `dotnet test`. Compile success is not proof of correctness.
- If a test fails, fix the root cause. Do not `[Fact(Skip = ...)]` or weaken
  assertions to go green.
- Warnings are errors. If you must suppress one, use `#pragma warning disable
<ID>` with a `// Reason:` comment above it — never edit `.csproj` to globally
  silence a rule.
