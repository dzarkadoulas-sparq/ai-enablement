Scaffold a new feature. Feature name: `$ARGUMENTS`.

Follow the project's package-by-feature convention. Use the existing
`users` feature as the reference — copy patterns and file names exactly.

1. **Create** `src/features/<feature>/`:
   - `index.ts` — re-exports the router only. This is the public surface.
   - `<feature>.router.ts` — `Router()` with each route wrapped in
     `asyncHandler(...)` and gated by auth middleware + `validate(schema)`.
   - `<feature>.controller.ts` — pulls validated data off `req`, calls the
     service, returns `res.json(...)`. No business logic.
   - `<feature>.service.ts` — business logic. Returns typed results or
     throws a `DomainError`.
   - `<feature>.repository.ts` — Prisma calls only. Returns domain objects,
     not Prisma models.
   - `<feature>.schema.ts` — Zod schemas for request/response. Export
     `z.infer<>` types.
   - `types.ts` — feature-internal types.
   - `__tests__/<feature>.service.test.ts` — Vitest unit tests, repo mocked.
   - `__tests__/<feature>.router.integration.test.ts` — supertest + real
     Postgres via Testcontainers.

2. **Wire the router** in `src/app.ts` under the versioned prefix
   (`/api/v1/<feature>s`).

3. **Create the Prisma migration**

   ```
   pnpm prisma migrate dev --name add_<feature>_table --create-only
   ```

   Review the generated SQL. Do NOT `migrate deploy` — leave it to the dev.

4. **Create test factories** in `tests/factories/<feature>.ts` using
   `@faker-js/faker`. No hardcoded UUIDs or emails.

5. **Verify**
   - `pnpm lint --fix && pnpm typecheck`.
   - `pnpm test src/features/<feature>` — green.

6. **Summary** — list every file created, migration filename, next manual
   steps (add to OpenAPI spec if generated, rate-limit if public, add to
   public-routes test if public).

Respect CLAUDE.md rules: no direct imports of a sibling feature's
service, no `process.env` outside `src/config/env.ts`, no default exports.
