Scaffold a new feature slice. Feature name: `$ARGUMENTS`.

Follow the project's Clean Architecture + MediatR vertical-slice convention.
Use the existing `Users` feature as the reference — copy patterns, naming,
and file structure exactly.

1. **Create the Application feature folder** under
   `src/Application/Features/<Feature>/`:
   - `Commands/Create<Feature>/Create<Feature>Command.cs` — MediatR `IRequest<...>`,
     a `record` with `required` members.
   - `Commands/Create<Feature>/Create<Feature>CommandHandler.cs` — returns a
     `Result<T>` or throws domain exceptions. Accepts `CancellationToken`.
   - `Commands/Create<Feature>/Create<Feature>CommandValidator.cs` —
     `AbstractValidator<Create<Feature>Command>` from FluentValidation.
   - `Queries/Get<Feature>ById/` — mirror structure (query + handler + validator).
   - `Dtos/<Feature>Dto.cs` — response record.
   - `Exceptions/<Feature>NotFoundException.cs` — extends `DomainException`.

2. **Create the Domain entity** at
   `src/Domain/<Feature>/<Feature>.cs` — class with private setters and a
   static `Create(...)` factory enforcing invariants. Pure; no EF attributes.

3. **Create the Infrastructure config** at
   `src/Infrastructure/Persistence/Configurations/<Feature>Configuration.cs`
   — `IEntityTypeConfiguration<<Feature>>`.

4. **Register the endpoint** in `src/Api/Endpoints/<Feature>Endpoints.cs`
   as a Minimal API `MapGroup("/api/<feature>s")` with:
   - `[Authorize]` (policy-based; pick the right policy or ask).
   - POST / GET / PUT / DELETE wired to MediatR via `ISender`.
   - `Results.Problem(...)` via the global exception handler.
     Wire this group into `Program.cs` where other endpoint groups are registered.

5. **Create the EF migration**
   - `dotnet ef migrations add Add<Feature>Table -p src/Infrastructure -s src/Api`.
   - Do NOT run `dotnet ef database update` — leave it to the developer.

6. **Create the tests**
   - `tests/Application.UnitTests/Features/<Feature>/Create<Feature>CommandHandlerTests.cs`
     — xUnit + FluentAssertions + NSubstitute.
   - `tests/Api.IntegrationTests/<Feature>EndpointTests.cs`
     — `WebApplicationFactory<Program>` + Testcontainers, happy path +
     one auth failure.
   - Test data builders under `tests/TestUtils/Builders/<Feature>Builder.cs`.

7. **Verify** — `dotnet build -warnaserror`. Report any errors before finishing.

8. **Summary** — list every file created and next manual steps
   (add the endpoint to the OpenAPI doc if generated, add a policy to
   `AuthorizationOptions` if new, run integration tests).

Respect CLAUDE.md rules: no EF types leaking out of Infrastructure, no
business logic in endpoints, inward-only references.
