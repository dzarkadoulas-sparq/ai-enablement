# API test suite: .NET + ASP.NET Core

## Discovery

- **OpenAPI** (Swashbuckle, NSwag): `/swagger/v1/swagger.json` (or a versioned path) in dev—export for static analysis or import to Postman.
- **Code:** **MapGet/MapPost** **(minimal)** **+** **MapGroup**; **or** **Controller** **attributes**—**build** **path** **from** **Route** + **Http** **attributes**.

## Test harness

- `WebApplicationFactory` + `HttpClient` (in-memory host, integration), or an older `TestServer` + `WebApplication` pattern if the repo still uses it.
- **Auth:** **TestAuth** **handler** or **bypass** **(per** **repo** **Test** **web** **application** **builder** **extensions**).

## Schema

- **System.Text.Json** and problem details (RFC 7807) if using `TypedResults.Problem`—assert `type` / `title` / `status` when the API uses them.

## Postman

- **Import** **from** **swagger.json**; **or** **run** **newman** in **CI** **against** **Kestrel** **test** **port**.

## Coverage

- Map minimal API `MapGet` / `MapPost` groups to test code (e.g. grep in `Program.cs` or extension methods that register routes).
