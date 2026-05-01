# Stack: Node.js + Express (TypeScript API)

## Detection

- `express` in dependencies; server entry in `src/index.ts`, `src/server.ts`, or `src/main.ts`.
- **Not** Next: no Next as the primary server (may coexist in monorepos — then treat as separate package).

## Read first

`README.md`, `package.json`, `tsconfig.json`, entry file, `src/config/env.*` (or `config.ts`), any ORM (`prisma/schema.prisma`, `TypeORM`, Drizzle), `.env.example`.

## Trace one flow

HTTP: mount order in `app` → **router** → **controller** → **service** → **repository/DB** (or the layering actually present). Note **where validation** runs (Zod, Joi, etc.).

## `CLAUDE.md` sections to generate

1. **Quick Reference** — dev with watch, build, test, integration test, migration commands from scripts and docs.
2. **Read first** — entry, env, schema/migrations, Docker Compose if any.
3. **Architecture** — router/controller/service/repository; feature folders; no business logic in middleware; central error handler.
4. **Code style** — ESM vs CJS, `strict` TS, validation at boundaries, logging.
5. **Security** — auth (JWT/session), rate limit, CORS, body limits, no env leakage; dependency audit command if configured.
6. **Testing** — unit vs integration; DB test strategy (Docker, testcontainers from README).
7. **Pitfalls** — unvalidated `req.body`, N+1 queries, missing transaction boundaries for multi-step writes.
8. Conflict / stop-and-ask line.

## Stack red flags

- Direct SQL or ORM in route handlers.
- `process.env` read outside a single config module.
- No integration tests for routes that perform writes.

## Exemplar (ai-enablement)

`templates/node-express/CLAUDE.md`
