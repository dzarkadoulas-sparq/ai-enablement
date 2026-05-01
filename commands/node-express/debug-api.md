Debug an Express endpoint. Problem: `$ARGUMENTS`.

1. **Identify the route** — grep `router.(get|post|put|delete|patch)` under
   `src/features/**`. Read:
   - The router + middleware chain (auth, validate, rate-limit).
   - The controller + service + repository.
   - Middleware order in `src/app.ts` (helmet, cors, json, error handler).

2. **Add targeted debug logging** via `pino` at each boundary.
   Bind `trace: "debug"` so we can filter. NEVER `console.log`.
   - Middleware entry (auth, validate).
   - Controller entry with validated payload (PII-redacted via pino redact paths).
   - Service call with domain args.
   - Repository call + returned row count.
   - Error middleware hit, if any.

3. **Reproduce via Postman MCP**
   - Find the project's collection (`postman_search_collections`). If absent,
     create a scratch request. Use a test token from the Postman vault on the
     `Staging` environment. NEVER production creds.
   - Capture: request headers (REDACT `Authorization`, `Cookie`), body,
     response status/headers/body, duration.
   - Capture a known-good variant for diff.

4. **Correlate** the HTTP exchange with the JSON logs. Find where
   expected ≠ actual (auth middleware short-circuit, Zod validation failure,
   Prisma tracking issue, forgotten `await`, etc.).

5. **Propose a fix** — file + line + reasoning. Do not apply without
   my confirmation.

6. **Clean up** — after the fix is approved, remove every `trace: "debug"`
   log call and re-run `pnpm test` to confirm no regression.

Constraints: never use production credentials; never disable auth, CORS,
or helmet to "make it work"; never add `any` to silence a type error —
narrow it properly.
