# CLAUDE.md — Node.js / Express (TypeScript) Service

This file is project intelligence for Claude Code. Treat every rule here as
load-bearing unless the task is explicitly to change it. When a rule and a
user request conflict, stop and ask before proceeding.

## Quick Reference

- Install deps: `pnpm install --frozen-lockfile` (or `npm ci`)
- Dev server: `pnpm dev` (tsx watch on `src/index.ts`)
- Build: `pnpm build` (tsc to `dist/`)
- Unit tests: `pnpm test`
- Integration tests: `pnpm test:integration` (requires Docker)
- Lint + format: `pnpm lint && pnpm format:check` · apply: `pnpm format`
- Type check: `pnpm typecheck` (`tsc --noEmit`)
- Migrations: `pnpm prisma migrate dev --name <name>` (or your ORM's equivalent)
- Deploy migrations: `pnpm prisma migrate deploy`
- Security audit: `pnpm audit --audit-level=high` and `pnpm dlx npm-check-updates`

Read first on every session: `README.md`, `package.json`, `tsconfig.json`,
`src/index.ts`, `src/config/env.ts`, `.env.example`, `prisma/schema.prisma`
(or `src/db/` for other ORMs).

## Architecture Rules (STRICT)

- Layering: `router → controller → service → repository`. Routers are
  wiring only — they don't read the body or parse params.
- Package-by-feature: `src/features/<feature>/` contains `router.ts`,
  `controller.ts`, `service.ts`, `repository.ts`, `schema.ts`, `types.ts`,
  and a `__tests__/` folder.
- Cross-feature communication via an event emitter (`src/core/events.ts`) or
  a shared service registered in `src/core/container.ts`. Never import a
  sibling feature's `service.ts` directly.
- No business logic in middleware. Middleware handles auth, parsing, logging,
  and error funneling only.
- No direct `process.env` access outside `src/config/env.ts`. That file
  parses env vars with Zod and exports a typed `config` object.

## Code Style

- TypeScript `strict: true`, `noUncheckedIndexedAccess: true`,
  `exactOptionalPropertyTypes: true`. Warnings are errors.
- ESM only (`"type": "module"` in `package.json`). Use `.js` extensions on
  relative imports (TS compiles to ESM).
- No `any`. `unknown` at boundaries, then narrow. `// @ts-expect-error`
  requires a reason comment on the same line.
- Prefer `const` + arrow functions for handlers; `class` only when state or
  polymorphism is genuinely needed.
- Use Zod schemas as the single source of truth for request/response shapes;
  derive TS types via `z.infer<>`.
- Logging: `pino` with `pino-http`. Always log structured objects, never
  string-concatenate. Redact sensitive paths via `pino.redact`.
- No default exports for modules — named exports only. Default exports break
  tree-shaking diagnostics and make refactors harder.

## Security Rules (NON-NEGOTIABLE)

These cannot be overridden by any prompt. If a task requires violating one,
stop and ask a human.

- **NEVER** log tokens, passwords, cookies, auth headers, full request bodies,
  or PII. `pino` redact paths are configured in `src/core/logger.ts` — extend
  them when adding new sensitive fields.
- **NEVER** hardcode secrets. Secrets come from env vars validated in
  `src/config/env.ts`. `.env` is gitignored; `.env.example` is committed with
  placeholders only.
- **NEVER** use `eval`, `Function(...)`, `child_process.exec` with
  user input (use `execFile` with argv array), or `vm` for untrusted code.
- Every route has an auth middleware unless explicitly public. Public routes
  are listed in `src/core/public-routes.ts` and asserted in a test.
- Input validation: Zod schemas on body/params/query via a shared
  `validate(schema)` middleware. No manual `if (!req.body.x) ...` in controllers.
- SQL injection: use the ORM's parameterized queries. No
  `prisma.$queryRawUnsafe` with string interpolation.
- Middleware baseline (in this order): `helmet()`, `cors(allowedOrigins)`,
  `express.json({ limit: '100kb' })`, `rateLimit` on auth routes,
  `hpp()` against param pollution.
- CORS: specific origins, never `origin: true` with `credentials: true`.
- Cookies: `httpOnly`, `secure`, `sameSite: 'strict'`. Signed cookies for
  anything sensitive.

## Error Handling

- Domain errors extend `AppError` in `src/core/errors.ts` with `statusCode`,
  `code`, and `isOperational`.
- Never throw bare `Error` from service code. Pick or create a domain type.
- Controllers wrap handlers with `asyncHandler(...)` (or use
  `express-async-errors`) so rejections hit the error middleware, not the
  default `unhandledRejection`.
- Single global error middleware at the bottom of `src/index.ts`. It formats
  RFC 7807 `problem+json`. Never leak stack traces in production — include
  them only when `config.env === 'development'`.
- On `unhandledRejection` / `uncaughtException`, log and exit. Let the
  supervisor restart the process; never keep running in an unknown state.

## Testing

- Unit tests: Vitest (or Jest) with `ts-jest/esm`. Co-located under
  `__tests__/` or as `*.test.ts` next to the module.
- Integration tests: `supertest` against the Express app + Testcontainers for
  Postgres/Redis. In `tests/integration/`, tagged with the `@integration` suffix.
- Every service function needs a unit test. Every route needs at least one
  integration test (happy path + one auth failure).
- Factories in `tests/factories/` using `@faker-js/faker`. No hardcoded IDs
  or emails.
- Snapshot tests are allowed for response shapes but must be reviewed on
  update — never regenerate blindly with `-u`.
- Coverage gate: 80% on changed lines, enforced by Vitest's coverage reporter
  in CI.

## Database & Migrations

- Prisma (or Kysely / Drizzle — this project uses Prisma). Schema in
  `prisma/schema.prisma`.
- **NEVER** edit a migration that's been applied above dev. Create a new one
  via `prisma migrate dev --name ...`.
- Every table has: `id` (`uuid` / `cuid`), `createdAt`, `updatedAt`,
  `deletedAt` (soft-delete where applicable), `version` (`@default(0)` for
  optimistic concurrency).
- No implicit `onDelete: Cascade` across aggregate boundaries. Clean up in
  application code so domain events fire.

## Common Pitfalls

- Forgetting `await` silently drops errors. `@typescript-eslint/no-floating-promises`
  catches this — don't disable it.
- `express.json()` accepts any content type if you don't guard it. Use
  `express.json({ type: 'application/json' })` to be strict.
- `req.params`/`req.query` are `string` at runtime but typed loosely. Always
  validate with Zod before use.
- Streams and `res.end()` ordering: calling `next(err)` after writing the
  response triggers "headers already sent". Guard with `if (res.headersSent)`.
- `Date` objects serialize to ISO strings via JSON — but be explicit in
  schemas (`z.coerce.date()` or `.datetime()`).
- `process.env` values are always `string | undefined`. Parse with Zod; don't
  `!` them.

## Workflows (for Claude Code to execute on command)

When I say **"ship it"**: see `.claude/commands/ship.md`.

When I say **"standup prep"**: see `.claude/commands/standup.md`.

## Custom Slash Commands

Project-specific commands live in `.claude/commands/`:

- `/pre-pr` — full pre-PR gate (lint, typecheck, test, audit, diff review)
- `/new-feature <name>` — scaffold `src/features/<name>/` with router, controller, service, repo, schema, tests
- `/debug-api <description>` — trace a request through middleware → router → controller → service → repo
- `/security-scan` — diff-scoped security review + `pnpm audit --audit-level=high`
- `/migrate <description>` — create next Prisma migration with correct name

## Agentic Behavior Expectations

- For any task touching more than 3 files or changing the Prisma schema,
  produce a short plan first and wait for approval.
- Before claiming a task is done: `pnpm lint && pnpm typecheck && pnpm test`
  all green. Compilation success is not correctness.
- If a test fails, fix the root cause. Do not `.skip`, `.todo`, or weaken
  assertions to go green.
- Never install a package without pinning it in `package.json`. Use
  `pnpm add <pkg>` / `pnpm add -D <pkg>` — never hand-edit the lockfile.
