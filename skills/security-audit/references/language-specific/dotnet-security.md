# .NET + ASP.NET Core — security focus

## Endpoints

- **Minimal APIs / Controllers** — `[Authorize]` by default; **`[AllowAnonymous]`** only where intended; **policy** / **role** names match **data** protection.
- **Anti-forgery** for **cookie** auth + form posts when applicable.

## Data access

- **EF** — `FromSqlRaw` with **string** interpolation; use **FormattableString** / parameters. **SQL injection** in `ExecuteSql*`.
- **IQueryable** returned to client = **unbounded** / **unfiltered** query risk in rare patterns — assert **server-side** filters for **tenant** / **user**.

## Secrets

- **User-secrets** / **KeyVault** in prod; no keys in `appsettings.Production.json` in repo.

## Headers

- `UseHsts`, `UseXContentTypeOptions` or **manual** in middleware; **CSP** in **NWebSec** or custom (see `header-recommendations.md`).

## Deserialization

- **TypeNameHandling** in **Newtonsoft** with untrusted input — **block**; **System.Text.Json** is safer by default; **polymorphic** `JsonSerializer` options need **allowlist**.

## Auth

- **JWT** **ValidateIssuerSignatureKey**, **ValidateAudience**, **ValidateLifetime**; no **symmetric** key in **client**; **mTLS** for service-to-service if used.

## Tools

- `dotnet list package --vulnerable --include-transitive`; `dotnet list package --deprecated`.
- **Security** analyzers in **.NET** and **Rosa** / **CodeQL** in pipeline if any.

## See also

`commands/dotnet/security-scan.md` in ai-enablement.
