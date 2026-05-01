# Module 6 — Codebase Understanding & Refactoring

## Detailed Lesson Plans with Examples

> **Why this module matters:** Most developers spend far more time reading and
> modifying existing code than writing new code from scratch. Claude Code's ability
> to rapidly comprehend large codebases, trace data flows across layers, explain
> undocumented design decisions, and execute large-scale refactors safely is one
> of its highest-value capabilities.

---

## Lesson 6.1 — Onboarding onto an Unfamiliar Codebase

Starting on a new team or inheriting a project means weeks of reading code, asking questions, and building a mental model. Claude Code compresses this dramatically — it can read the entire project structure, identify patterns, map dependencies, and give you a guided tour in minutes.

### Example 1: The First 10 Minutes on a New Project

You've just cloned a repo you've never seen before:

```
I just joined this project and have never seen this codebase.
Give me a comprehensive orientation:

1. Project structure: What's the high-level directory layout
   and what does each top-level directory contain?
2. Tech stack: What language, framework, database, and key
   libraries are used? Check package.json / pom.xml / .csproj
   and config files.
3. Architecture: Is this a monolith, microservices, monorepo?
   How is the code organized — by feature, by layer, or hybrid?
4. Entry points: Where does the application start? What's the
   main file, and how does a request flow from entry to response?
5. Data layer: What database is used? Where are models/entities
   defined? Is there an ORM?
6. Key patterns: What recurring patterns do I need to understand
   (repository pattern, middleware chain, event bus, CQRS, etc.)?
7. Build & run: How do I build, test, and run this locally?
   Check scripts in package.json, Makefile, or CI config.
8. Red flags: Anything unusual, concerning, or that I should
   be careful about?
```

Claude Code reads the project and produces a structured briefing:

> **Project: OrderFlow (E-commerce Backend)**
>
> **Stack:** Java 17, Spring Boot 3.2, PostgreSQL 15, Redis for caching,
> RabbitMQ for async messaging. Built with Gradle.
>
> **Architecture:** Modular monolith. Code is organized by domain module
> (orders, inventory, users, payments) under `src/main/java/com/orderflow/`.
> Each module has its own controller, service, repository, and DTO layers.
>
> **Request flow:** HTTP → Spring Security filter chain → Controller →
> Service (business logic) → Repository (JPA) → PostgreSQL. Async
> operations (email, inventory sync) go through RabbitMQ listeners
> in each module's `events/` package.
>
> **Key patterns to know:**
>
> - All repositories extend `BaseRepository<T>` which adds soft-delete
>   filtering automatically
> - Services use `@Transactional` at the method level — be careful about
>   nested transactions
> - DTOs are mapped using MapStruct (see `*/mapper/` packages)
> - Error handling uses a global `@ControllerAdvice` in `common/exceptions/`
>
> **Red flags:**
>
> - The `payments/` module has a `LegacyPaymentAdapter` that talks to an
>   old SOAP service. Comments say "temporary" but the git log shows it's
>   been there since 2021.
> - `inventory/service/StockService.java` is 847 lines with 23 public
>   methods. This is the god class of the project.
> - No integration tests for the RabbitMQ consumers.

### Example 2: Understanding a Specific Module in Depth

After the orientation, dive deeper into the area you'll be working on:

```
I'll be working on the payments module. Give me a deep dive:

1. Read every file in src/main/java/com/orderflow/payments/
   and explain the purpose of each class.
2. Draw the dependency graph: what does the payments module
   depend on, and what depends on it?
3. Trace the payment flow step by step: what happens when a
   user clicks "Pay" — which classes are involved, in what
   order, and what data flows between them?
4. Where are the configuration values (API keys, endpoints,
   timeouts)? Are they in application.yml, environment
   variables, or hardcoded?
5. What tests exist? What's the coverage like?
6. What are the riskiest parts of this module — where would
   a bug cause the most damage?
```

### Example 3: Understanding Code You Didn't Write

You found a function that does something complex and undocumented:

```
Explain what the function `reconcileInventorySnapshot` in
src/main/java/com/orderflow/inventory/service/StockService.java
does. I need to understand:

- What is its purpose in plain English?
- What are the inputs and outputs?
- What side effects does it have (database writes, events, etc.)?
- Under what conditions is it called? Search for all call sites.
- Are there any edge cases or race conditions I should worry about?
- Why does it exist? Check the git blame and PR history for
  context on when and why it was added.
```

Claude Code reads the function, traces its callers, checks git history, and explains:

> **`reconcileInventorySnapshot`** compares the in-memory inventory counts
> (cached in Redis) against the actual database values and corrects any drift.
>
> It was added in PR #187 (August 2023) after an incident where Redis cache
> became stale after a failed deployment, causing overselling. The PR comment
> says: "Temporary safeguard until we fix the cache invalidation properly."
>
> It runs on a `@Scheduled` cron job every 15 minutes (see `InventoryScheduler.java`).
> It also gets called manually from the admin endpoint `POST /api/admin/inventory/reconcile`.
>
> **Race condition risk:** If a purchase happens during reconciliation, the
> reconciliation might overwrite the decremented count with the stale snapshot.
> There's a comment on line 234 that says `// TODO: add optimistic locking here`
> but it was never implemented.

### Good Practices

**Start broad, then narrow:**

```
Give me the 10,000-foot view first. Don't go into any file
in detail until I understand the overall architecture.
```

Then:

```
Now zoom into the payments module specifically.
```

Then:

```
Now explain this specific function in the payments module.
```

Each zoom level builds on the mental model from the previous one.

**Ask Claude Code to find the "tribal knowledge":**

```
What are the things about this codebase that aren't obvious
from reading the code? Check for:
- Comments that say "TODO", "HACK", "FIXME", "WORKAROUND"
- Configuration that seems unusual or non-default
- Code that contradicts the apparent architecture
  (e.g., direct database queries in controllers)
- Dependencies that seem outdated or deprecated
```

This surfaces the hidden context that would normally take months to discover through painful experience.

**Generate a CLAUDE.md from your exploration:**

```
Based on everything you've learned about this codebase,
draft a CLAUDE.md file that would help you (or another
Claude Code session) work effectively in this project.
Include architecture patterns, naming conventions, common
pitfalls, and build/test commands.
```

This turns your onboarding session into a reusable asset for the entire team.

### Things to Avoid

**Trying to understand everything at once:**

```
Read every file in the entire codebase and explain what
each one does.
```

A codebase with 500 files can't be understood in one prompt. The explanation will be either superficial or overwhelming. Focus on the area you need to work in first.

**Assuming the code matches the documentation:**

```
Read the README and explain the architecture.
```

READMEs go stale. Always verify against the actual code:

```
Read the README for the architecture overview, then check
if the actual code matches. Flag any discrepancies between
what the docs say and what the code actually does.
```

---

## Lesson 6.2 — Dependency and Impact Analysis

Before changing anything in a large codebase, you need to understand the blast radius. What depends on the code you're about to change? What will break? Claude Code can trace dependency chains across files, modules, and services — something that's tedious and error-prone for humans.

### Example 1: "What Will Break if I Change This?"

```
I need to change the signature of the `UserRepository.findByEmail`
method in src/repositories/UserRepository.ts. Currently it returns
`User | null` and I want to change it to return
`Result<User, NotFoundError>`.

Before I make this change, find every file that calls
`findByEmail` — directly or through a service that wraps it.
For each call site, tell me:
- File and line number
- How the current return value is used (null check, optional
  chaining, direct access, etc.)
- What would need to change to work with the new Result type
- Risk level: trivial (just unwrap the Result), moderate
  (needs new error handling), or complex (fundamental logic change)

Give me the full impact report before I change anything.
```

Claude Code traces the dependency chain and reports:

> **Impact analysis: `UserRepository.findByEmail` signature change**
>
> **Direct callers (5 files):**
>
> 1. `src/services/AuthService.ts:34` — Uses `if (!user)` null check
>    → Trivial: change to `if (result.isErr())`
> 2. `src/services/UserService.ts:78` — Chains with optional: `user?.email`
>    → Moderate: need to unwrap Result first, handle error case
> 3. `src/controllers/AuthController.ts:22` — Passes directly to response
>    → Complex: currently returns 404 when null, needs restructuring to
>    use Result's error type for the 404 response
> 4. `src/middleware/auth.ts:15` — Used in authentication middleware
>    → Moderate: critical path, needs careful error propagation
> 5. `src/jobs/WelcomeEmailJob.ts:41` — Silently skips if null
>    → Trivial: change null check to Result check
>
> **Indirect callers (through AuthService):**
>
> 6. `src/controllers/LoginController.ts:18` — Calls `authService.login()`
>    which calls `findByEmail` internally
>    → No change needed (AuthService handles the Result internally)
>
> **Estimated total changes: 5 files, ~40 lines**
> **Risk: Medium — the auth middleware (item 4) is on the critical path**

### Example 2: Module Dependency Mapping

```
Map the dependency graph for the entire src/modules/ directory.
For each module, show:
- What other modules it imports from
- What other modules import from it
- Any circular dependencies

Present this as a dependency matrix. Flag any module that is
imported by more than 5 other modules — these are the most
dangerous to change because the blast radius is large.
```

### Example 3: Database Schema Impact Analysis

```
I want to rename the `users.email_address` column to
`users.email`. Before I write the migration, find every
reference to this column across the entire codebase:
- ORM model definitions
- Raw SQL queries (grep for "email_address")
- Database seeders and fixtures
- Test factories
- API response serializers that expose this field
- Frontend code that references the field name from API responses
- Documentation and API specs

I need to change ALL of them atomically or the deploy will break.
```

### Good Practices

**Always do impact analysis before changing shared interfaces:**

```
/plan I want to refactor the ErrorResponse type. Before
planning the implementation, map every file that imports
or uses ErrorResponse and assess the blast radius.
```

The `/plan` + impact analysis combo prevents you from starting a refactor that turns out to touch 200 files.

**Check for indirect dependencies, not just direct imports:**

```
Don't just search for direct imports of UserRepository.
Also check for:
- Services that wrap UserRepository and are called elsewhere
- Event handlers that react to user-related events
- Scheduled jobs that query users
- Test helpers and factories that create test users
```

### Things to Avoid

**Changing a shared interface without checking consumers:**

```
Change the UserDTO to add a required `phoneNumber` field.
```

If 30 places construct a UserDTO, they all need updating. Check first:

```
Before adding the required field, find every place that
constructs a UserDTO. Should this be optional instead of
required to avoid a massive change?
```

**Trusting your IDE's "Find Usages" for dynamic languages:**

In TypeScript/JavaScript/Python, IDEs miss dynamic references, string-based lookups, and reflection. Claude Code can search more comprehensively:

```
Search for usages of `findByEmail` using both AST-aware
tools and plain text grep. Include:
- Direct method calls
- Dynamic calls via bracket notation (obj["findByEmail"])
- String references ("findByEmail" in configs or tests)
- References in comments and documentation
```

---

## Lesson 6.3 — Large-Scale Refactoring

Large refactors — the kind that touch dozens or hundreds of files — are where Claude Code provides a step change in productivity. A human doing a codebase-wide rename, pattern migration, or architecture change needs hours of careful, repetitive work. Claude Code can do it systematically and consistently.

### Example 1: Migrating from Callbacks to Async/Await (Node.js/TypeScript)

```
Our codebase has a mix of callback-style and async/await code.
Migrate everything to async/await following this plan:

Phase 1: Identify all callback-style functions.
Search for patterns like:
- Functions that take a `callback` or `cb` parameter
- Functions that call `callback(err, result)` or `cb(null, data)`
- Promise chains using `.then().catch()`

List them all with file paths and line numbers.

Phase 2: Migrate in dependency order.
Start with the lowest-level utilities (no dependencies on
other callback functions), convert them to async, then move
up to their callers. This way we never have a half-migrated
chain.

Phase 3: Update all callers.
After converting a function, find all its call sites and
update them from callback-style to await.

Phase 4: Verify.
After each file, run `npx tsc --noEmit` to check for type
errors. After each module, run the tests.

Work through Phase 1 now. Show me the full list before
we start Phase 2.
```

### Example 2: Extracting a God Class (Java)

The 847-line `StockService.java` from our codebase orientation needs to be broken up:

```
Read src/main/java/com/orderflow/inventory/service/StockService.java.
This is an 847-line god class that does too much.

Step 1: Analyze the class and identify distinct responsibilities.
Group the public methods by the domain concern they handle.
I expect you'll find at least 3-4 separate concerns mixed together.

Step 2: Propose a refactoring plan that extracts each responsibility
into its own service class. For each proposed class:
- Name and location
- Which methods move there
- What dependencies it needs
- What interface it should implement

Step 3: Identify the tricky parts:
- Methods that span multiple responsibilities
- Shared private helper methods
- Shared state (instance variables used by multiple groups)
- Transaction boundaries that currently span responsibilities

Show me the plan before making any changes.
```

Claude Code analyzes and proposes:

> **Identified responsibilities in StockService (847 lines):**
>
> 1. **Stock Query** (lines 45-180): `getStock`, `getStockBulk`,
>    `getStockHistory`, `getLowStockAlerts`
>    → Extract to `StockQueryService`
> 2. **Stock Mutation** (lines 181-390): `decrementStock`, `incrementStock`,
>    `adjustStock`, `transferStock`, `bulkAdjust`
>    → Extract to `StockMutationService`
> 3. **Reconciliation** (lines 391-580): `reconcileInventorySnapshot`,
>    `detectDrift`, `autoCorrect`, `generateReconciliationReport`
>    → Extract to `StockReconciliationService`
> 4. **Forecasting** (lines 581-847): `forecastDemand`, `calculateReorderPoint`,
>    `suggestReorderQuantity`, `generateForecastReport`
>    → Extract to `StockForecastingService`
>
> **Tricky parts:**
>
> - `validateStockLevel()` (private, line 160) is used by both Query and
>   Mutation. → Move to a shared `StockValidator` utility.
> - `decrementStock` and `reconcileInventorySnapshot` both use the
>   `stockCache` instance variable. → Inject the cache as a dependency
>   into both new services.
> - `decrementStock` emits a `StockChangedEvent` that `reconcileInventorySnapshot`
>   listens to. This creates a hidden coupling. → Make the event contract
>   explicit via an interface.

After approval:

```
Good plan. Execute Phase 1: Extract StockQueryService.
Move the 4 query methods and their tests. Update all callers
to use the new service instead of StockService. Run the full
test suite after.

Don't touch the other responsibilities yet — we'll extract
them one at a time.
```

### Example 3: Migrating from REST to GraphQL (Incremental)

```
We're adding GraphQL alongside our existing REST API. We're NOT
replacing REST — both will coexist during the migration period.

Phase 1 (this session): Set up GraphQL infrastructure.
- Add Apollo Server to the existing Express app
- Create the schema for the User type (mirror the REST response shape)
- Create a resolver that calls the EXISTING UserService
  (don't duplicate business logic)
- Add a single query: `user(id: ID!): User`

The key constraint: the resolver must call UserService.findById(),
not query the database directly. GraphQL is a new API layer,
not a new data layer.

After this works, we'll add more types and queries in future phases.
```

### Example 4: Framework Upgrade (.NET)

```
We need to upgrade this project from .NET 6 to .NET 8.
This involves:

Step 1: Audit.
- Check the .NET 6 → 7 → 8 migration guides for breaking changes
- Read our codebase and identify which breaking changes affect us
- List every NuGet package and check which ones need version bumps
  for .NET 8 compatibility

Step 2: Upgrade project files.
- Update TargetFramework in all .csproj files
- Update NuGet packages to .NET 8 compatible versions
- Update the Dockerfile base image

Step 3: Fix breaking changes.
- Apply each required code change from Step 1
- Pay special attention to:
  - Minimal API changes
  - Authentication/authorization middleware changes
  - EF Core migration compatibility

Step 4: Verify.
- Build the solution: `dotnet build`
- Run all tests: `dotnet test`
- Fix any issues

Show me the Step 1 audit before proceeding. I want to know
what we're getting into.
```

### Example 5: Replacing a Library Across the Codebase (React)

```
We're replacing Moment.js with date-fns across the entire
React frontend.

Step 1: Find every file that imports from 'moment'. List them.

Step 2: For each Moment function used, identify the date-fns
equivalent. Create a mapping:
- moment().format('YYYY-MM-DD') → format(new Date(), 'yyyy-MM-dd')
- moment(date).fromNow() → formatDistanceToNow(date)
- moment(date).isBefore(other) → isBefore(date, other)
- etc.

Show me the complete mapping before you start replacing.

Step 3: Replace file by file, starting with utility files
(used by many components), then components. After each file:
- Run TypeScript compiler to check types
- Run that file's tests if they exist

Step 4: After all files are migrated, remove moment from
package.json and verify the build succeeds.
```

### Example 6: Database Schema Migration (Java/Hibernate)

```
We need to split the `addresses` table (which stores both
billing and shipping addresses in the same table with a `type`
column) into two separate tables: `billing_addresses` and
`shipping_addresses`.

This is a multi-step migration:

Step 1: Create the new tables and entities.
- New Hibernate entities for BillingAddress and ShippingAddress
- Flyway migration to create the tables

Step 2: Write a data migration.
- Flyway migration that copies data from `addresses` into the
  correct new table based on the `type` column
- Verify row counts match after migration

Step 3: Update the codebase.
- Replace all references to the old Address entity/repository
- Update services, controllers, DTOs, and mappers
- Update tests

Step 4: Create the cleanup migration.
- Drop the old `addresses` table (only after verifying everything
  works)

Do NOT execute Step 4 yet — we'll do that in a future release
after confirming the migration is stable.

Start with Step 1 and show me the entities and migration
before proceeding.
```

### Good Practices

**Refactor in phases with verification between each phase:**

```
After each phase, run the full test suite and build.
Don't start Phase 2 until Phase 1 is green. If Phase 1
breaks something, fix it before adding more changes.
```

**Keep behavior changes and structural changes separate:**

```
This refactor should NOT change any behavior. It's purely
structural — moving code, renaming, splitting classes.
All existing tests should pass without modification.
If a test needs to change, that's a signal that we're
accidentally changing behavior.
```

This is the golden rule of refactoring. If you change structure and behavior at the same time, you can't tell which caused a test failure.

**Use the strangler fig pattern for large migrations:**

```
Don't replace the old system all at once. Build the new
approach alongside it, migrate one piece at a time, and
delete the old code only when nothing depends on it.
Keep both running during the transition.
```

### Things to Avoid

**Refactoring without tests as a safety net:**

```
Refactor the PaymentService into smaller classes.
```

If PaymentService has no tests, you have no way to verify the refactor preserved behavior. Write characterization tests first (see Module 5, Lesson 5.3), then refactor:

```
Before refactoring PaymentService, write characterization
tests that capture its current behavior for all code paths.
We need these as a safety net. THEN we'll refactor.
```

**Making the "perfect" refactor in one giant PR:**

```
Refactor the entire data layer from raw SQL to use the
repository pattern across all 45 database-accessing files.
```

A 45-file PR is unreviewable and risky. Break it up:

```
Let's migrate to the repository pattern one module at a time.
Start with the Users module (5 files). When that's merged
and stable, we'll do Orders, then Payments, etc.
Each module is its own PR.
```

**Refactoring code you don't understand:**

```
This utility function is messy. Clean it up.
```

Before refactoring, understand _why_ it looks the way it does:

```
Before cleaning up this function, explain what it does and
check git blame for context. Is the complexity intentional
(handling edge cases discovered in production) or accidental
(accumulated tech debt from quick fixes)?
```

Sometimes messy code is messy for a reason — it handles edge cases that a "clean" rewrite would miss.

---

## Lesson 6.4 — Architecture Analysis and Documentation

Claude Code can read an entire codebase and produce architectural documentation that would take a human architect days to write. This is valuable for onboarding, design reviews, and making informed decisions about where to invest refactoring effort.

### Example 1: Generate an Architecture Decision Record

```
I need to document our architecture for new team members.
Read the entire codebase and generate an Architecture
Decision Record (ADR) that covers:

1. System overview: What does this system do, who uses it?
2. Architecture style: What pattern (monolith, microservices,
   event-driven, etc.) and why can you infer that from the code?
3. Module map: What are the main modules and their responsibilities?
4. Data flow: How does data flow through the system for the
   3 most common operations?
5. External dependencies: What external services, APIs, and
   databases does the system integrate with?
6. Cross-cutting concerns: How are auth, logging, error handling,
   caching, and configuration managed?
7. Technical debt: What are the top 5 areas of technical debt
   you can identify from the code?
8. Missing pieces: What would you expect to find in a production
   system that's missing here (monitoring, rate limiting,
   circuit breakers, etc.)?
```

### Example 2: Detect Architectural Violations

```
Our intended architecture has these rules:
- Controllers should only call Services (never Repositories directly)
- Services should only call Repositories and other Services
- No circular dependencies between modules
- All database access goes through Repository classes
- Events are the only way modules communicate asynchronously

Scan the codebase and find every violation of these rules.
For each violation, show:
- File and line number
- Which rule is violated
- What the code does instead
- How hard it would be to fix (trivial, moderate, complex)
```

Claude Code scans and reports:

> **Architectural violations found: 12**
>
> **Rule: Controllers should only call Services**
>
> 1. `src/controllers/ReportController.ts:45` — Calls `OrderRepository.findAll()`
>    directly, bypassing OrderService. Severity: moderate. The controller is
>    building a custom query that OrderService doesn't support.
> 2. `src/controllers/AdminController.ts:89` — Calls `db.query()` with raw SQL.
>    Severity: complex. The admin dashboard has 5 raw queries for analytics that
>    don't map to any existing repository method.
>
> **Rule: No circular dependencies** 3. `orders/OrderService` imports from `payments/PaymentService`, and
> `payments/PaymentService` imports from `orders/OrderService`.
> Severity: complex. They call each other for order-payment reconciliation.
> Needs an event-based decoupling or a new shared service.
>
> [...]

### Example 3: Map the Data Flow for a Critical Operation

```
Trace the complete data flow for "user places an order" from
the moment the HTTP request arrives to the moment the response
is sent. Include:

- Every function call in the chain
- Every database read and write
- Every external service call (payment gateway, email, etc.)
- Every event emitted or consumed
- Every cache read/write
- Every validation step

I want to see the full picture, including async operations
that happen after the response is sent (webhook handlers,
background jobs, etc.).
```

### Example 4: Technical Debt Assessment

```
Analyze the codebase for technical debt. Quantify what you find:

1. Code complexity: Which files have the highest cyclomatic
   complexity? List the top 10.
2. Duplication: Find significant code duplication (not just
   one-liners — patterns of 10+ lines that repeat).
3. Outdated dependencies: Check all dependencies against their
   latest versions. Flag any with known security vulnerabilities
   or that are end-of-life.
4. Dead code: Find functions, classes, and files that are
   never imported or called.
5. TODO/FIXME/HACK comments: List them all with context. How
   old are they (git blame)?
6. Test gaps: Which modules have the lowest test coverage
   relative to their complexity?

Rank the debt items by a combined score of risk × effort-to-fix.
What should we tackle first?
```

### Good Practices

**Document as you explore — don't separate understanding from documentation:**

```
As you analyze each module, write the documentation in real
time. Create a docs/architecture/ directory with:
- overview.md (system-level)
- One file per module (module-name.md)
- data-flows.md (key operation traces)
- tech-debt.md (prioritized debt inventory)

This way the exploration produces a lasting artifact, not just
a conversation I'll forget.
```

**Validate your architectural assumptions against the code:**

```
Our tech lead says we follow clean architecture with strict
dependency inversion. Verify this: do the inner layers
(domain/entities) truly have no imports from outer layers
(frameworks, infrastructure)? Show me any violations.
```

**Use architecture analysis to inform refactoring priorities:**

```
Based on the technical debt assessment, which refactoring
would give us the highest return on investment? Consider:
- Risk reduction (fixing things that could cause incidents)
- Developer velocity (fixing things that slow us down daily)
- Effort required (quick wins vs. multi-sprint projects)

Give me a prioritized roadmap: what to do this sprint,
next sprint, and next quarter.
```

### Things to Avoid

**Generating documentation and never updating it:**

Documentation that's generated once and never maintained becomes misleading. Include a date and a "how to regenerate" command:

```
At the top of each generated doc, add:
"Generated by Claude Code on [date]. To regenerate, run the
following prompt: [include the prompt that created this doc]."
```

**Treating Claude Code's architecture analysis as infallible:**

Claude Code reads code statically. It can miss:

- Runtime behavior that depends on configuration or feature flags
- Dependency injection that's configured at startup (not in the code it reads)
- Infrastructure-level concerns (load balancers, service meshes, DNS routing)
- Deployment topology (which services run where)

Always validate the analysis against your team's actual knowledge:

```
Here's Claude Code's architecture analysis. Review it with
the team and annotate anything that's wrong or missing.
```

---

## Lesson 6.5 — Generating and Maintaining Documentation

Documentation is one of Claude Code's highest-ROI use cases: it reads the code, understands the intent, and produces documentation that's grounded in what the code _actually does_ — not what someone remembers it doing.

### Example 1: API Documentation from Code

```
Read all route handlers in src/routes/ and generate OpenAPI 3.0
documentation for every endpoint. For each endpoint, include:
- HTTP method and path
- Request parameters (path, query, body) with types
- Response schemas for all status codes (200, 400, 401, 404, 500)
- Authentication requirements
- Example request and response

Infer the schemas from the actual TypeScript types used in the
handlers and the validation schemas (zod). Don't guess — base
everything on what the code actually does.

Write the output as openapi.yaml in the docs/ directory.
```

### Example 2: Code-Level Documentation for Complex Logic

```
The following files contain complex business logic that no one
on the team fully understands anymore:
- src/services/pricing/DiscountEngine.java
- src/services/pricing/TaxCalculator.java
- src/services/pricing/ShippingCostResolver.java

For each file:
1. Read the code carefully, including all branches and edge cases
2. Add JSDoc/Javadoc comments to every public method explaining:
   - What it does (in plain English)
   - Why it exists (infer from git history and surrounding code)
   - Edge cases it handles (and why)
   - Example inputs and outputs
3. Add inline comments for any non-obvious logic (magic numbers,
   workarounds, business rules)

Don't change any code — only add documentation.
```

### Example 3: Generating a Migration Guide

```
We're upgrading from v2 to v3 of our API. Read both versions:
- v2 routes: src/routes/v2/
- v3 routes: src/routes/v3/

Generate a migration guide for API consumers that covers:
- Endpoints that changed (old path → new path)
- Request/response schema changes (field renames, type changes,
  removed fields, new required fields)
- Behavioral changes (pagination style, error format, auth flow)
- Endpoints that were removed
- New endpoints added

Format this as a markdown document that we can publish for
external API users.
```

### Example 4: Onboarding Guide from Codebase Analysis

```
Generate a developer onboarding guide based on what you've
learned about this codebase. The audience is a mid-level
developer joining the team. Include:

1. Getting Started (setup instructions verified against
   actual config files)
2. Architecture Overview (based on actual code structure)
3. Development Workflow (based on scripts in package.json,
   CI config, and git hooks)
4. Key Concepts (patterns, conventions, and domain terms
   they need to know)
5. Common Tasks (how to add a new endpoint, create a migration,
   write a test — with file templates based on existing examples)
6. Troubleshooting (common issues based on TODOs, comments,
   and error handling code)
```

### Good Practices

**Generate documentation from code, not from memory:**

```
Base every statement in the documentation on something you
can point to in the code. If you're not sure about something,
say "inferred from [file:line]" or flag it as needing
human verification.
```

**Keep generated docs alongside the code they describe:**

```
Put the API docs in docs/api/ next to the routes.
Put the architecture docs in docs/architecture/.
Put inline docs directly in the source files.

Don't create a separate documentation repository that will
drift from the code.
```

### Things to Avoid

**Generating generic documentation that doesn't reflect the actual code:**

```
Write documentation for a typical Spring Boot application.
```

This will produce boilerplate that might not match your project. Always base documentation on the actual code:

```
Write documentation for THIS Spring Boot application,
based on reading the actual source code.
```

**Over-documenting trivial code:**

```
Add comments to every line of every file.
```

Comments should explain _why_, not _what_. `i++; // increment i` is noise. Focus on complex logic, business rules, and non-obvious decisions.

---

## Module 6 Assessment

### Project: Inherit and Improve an Unknown Codebase

Students receive a medium-sized application (~200 files) they've never seen, with intentional architectural issues, technical debt, and missing documentation.

**Part A: Codebase Exploration (graded)**

Produce a comprehensive orientation document within a fixed time limit:

- Architecture overview with dependency map
- Tech stack identification
- Critical path tracing for 2 core operations
- Risk assessment (what could go wrong in production?)

Graded on accuracy, depth, and how quickly a new developer could get productive using the document.

**Part B: Impact Analysis (graded)**

The instructor provides 3 proposed changes. Students must:

- Analyze the blast radius of each change
- Identify all affected files and the nature of each change needed
- Rate risk and effort for each
- Recommend which changes are safe to make and which need redesign

**Part C: Large-Scale Refactoring (graded)**

Execute one major refactoring:

- Extract a god class into focused, single-responsibility classes
- Maintain all existing behavior (tests must pass without changes)
- Work in phases with verification between each phase
- Final code should pass linting and type-checking with zero warnings

**Part D: Architecture Documentation (graded)**

Generate production-quality documentation:

- Architecture Decision Record
- API documentation (OpenAPI spec)
- Developer onboarding guide
- Technical debt inventory with prioritized roadmap

**Grading criteria:**

- Exploration accuracy (did they correctly identify the architecture, or did they miss key patterns?)
- Impact analysis completeness (did they find all affected files, including indirect dependencies?)
- Refactoring safety (did they break anything? did they verify between phases?)
- Documentation quality (would a new developer find it useful? is it grounded in the actual code?)
- Use of Claude Code techniques from earlier modules (CLAUDE.md, `/plan`, prompt patterns, etc.)
