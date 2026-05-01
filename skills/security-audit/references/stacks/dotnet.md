# Security audit: .NET + ASP.NET Core

## Read first

- `references/language-specific/dotnet-security.md`
- `commands/dotnet/security-scan.md` (ai-enablement)

## Dependency scan

- `dotnet list package --vulnerable --include-transitive`; `dotnet list package --deprecated`. Prioritize **reachable** / **touched** packages.

## Code scope (typical)

- `*Endpoints*`, `*Controller*`, `*Program.cs*`, `*Authentication*`, `*Authorization*`, EF `FromSql*`, `appsettings*.json`.

## Focus

- **Minimal API** / **controller** `RequireAuthorization` vs `AllowAnonymous`; **policy** **handlers**.
- **EF** raw SQL; **Newtonsoft** `TypeNameHandling` with user JSON.
- **HSTS** / **forwarding** headers when behind **reverse proxy** (`UseForwardedHeaders` + options).

## Build

- `dotnet build` with same **warnings** as **Release**; **SAST** in pipeline if any.
