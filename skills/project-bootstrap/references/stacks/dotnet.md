# Stack: .NET / ASP.NET Core

## Detection

- `*.sln` and `*.csproj` with `Microsoft.NET.Sdk` / `Web.Sdk`.
- `Program.cs` with minimal hosting or `Startup` pattern; ASP.NET Core pipeline.

## Read first

`README.md`, `.sln`, `Directory.Packages.props` / `Directory.Build.props` if present, `global.json` for SDK pin, `src/**/Program.cs` or `Host`, `appsettings*.json`, EF Core design-time project (README often documents `dotnet ef`).

## Trace one flow

Minimal API or `Controller` → **MediatR** handler (if used) or service → `DbContext` / external client. Note **Carter** / **FastEndpoints** if used instead of controllers. Respect **clean architecture** folders if visible (`Domain` / `Application` / `Infrastructure` / `Api`).

## `CLAUDE.md` sections to generate

1. **Quick Reference** — `dotnet restore` / `build` / `test` / `format`, `dotnet ef` migrations, `docker compose` if documented.
2. **Read first** — solution, API host `Program.cs`, central package management files, settings.
3. **Architecture** — project dependency direction, MediatR/pipeline, API vs Infrastructure, mapping (Mapster/AutoMapper) if any.
4. **Code style** — nullable, `TreatWarningsAsErrors`, file-scoped namespaces, `async` all the way, records for DTOs.
5. **Security** — authentication handlers, `IAuthorizationService`, data protection, secrets in user-secrets/KeyVault patterns from repo.
6. **EF Core** — migrations location, `AsNoTracking` for read-only, bounded contexts.
7. **Testing** — xUnit/NUnit, `WebApplicationFactory` or integration style from `tests/`.
8. **Pitfalls** — `DbContext` in handlers across threads, sync-over-async, leaking `IQueryable` to API layer.
9. Conflict / stop-and-ask line.

## Stack red flags

- `DbContext` or `IQueryable` returned from application API surface.
- `.Result` / `.GetAwaiter().GetResult()` in ASP.NET request path.
- Missing nullable reference context or inconsistent `nullable` per project (debt).

## Exemplar (ai-enablement)

`claude.md/dotnet/CLAUDE.md`
