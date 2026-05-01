# Refactor guide: .NET + ASP.NET Core

## Verify

- `dotnet build` (match CI configuration for Release and warnings), `dotnet test` (unit; integration with the project’s filter), `dotnet format` if the repo enforces it.

## Refactor-specific notes

- **DI** in `Program.cs` or an Autofac module: understand **scoped** vs **singleton** lifetimes and **circular** dependencies (MediatR handlers, `DbContext`) before reordering registrations.
- **EF Core** — renames: **migrations** plus `FromSqlRaw` / `ExecuteSql*`, raw SQL, and `IQueryable` in reporting or ad-hoc code.
- **Minimal API** `MapGroup` — moving endpoints between files can change **OpenAPI** order or **versioning** edge cases.

## Pitfalls

- **Source generators** and **AOT** (when enabled): refactors that affect attributes or generated code may need a **clean** build to validate.
