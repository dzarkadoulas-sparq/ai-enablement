# Impact analysis: .NET + ASP.NET Core

## Tracing

- **Usings** and **project** **references** in **`.csproj`**. **Transitive** **ref** in **Solution** can be **traced** **via** **“Find** **all** **references**” in **Rider/VS** (best for **C#**).
- **MediatR** — `IRequestHandler<YourQuery>` **implementations**; **Wolverine** / **MassTransit** **consumers** **if** used.

## Dynamic

- **Reflection** — `Activator`, **DI** **keyed** **by** **string**; **Minimal** **API** **group** **MapGet** **with** **delegate** **names** in **tests** **only** **sometimes**.

## Config

- **Options** **pattern** — `IOptions<Section>` **if** you **rename** **section**; **`appsettings*.json`** **grep** **hierarchy**.

## EF

- **DbContext** **DbSet** **properties**; **migrations** **+** **compiled** **queries** **by** **entity** **name**.

## Public API

- **NuGet** **consuming** **this** **assembly** (external)—**out** of **tree**; **note** as **version** **+** **changelog** **risk**.

## Tools

- `dotnet build` **+** **IDE** **refs**; **Roslyn** **analyzers** for **obvious** **breaks**.
