Debug a Spring Boot API endpoint. Problem: `$ARGUMENTS`.

1. **Identify the route** — grep for the path/verb in `src/main/java/.../api/`.
   Read the controller, service interface + impl, and repository it touches.
   Note the filter chain order in `SecurityConfig` and any `@ControllerAdvice`
   that might rewrite the response.

2. **Add targeted debug logging** at each boundary using `log.debug(...)`
   with structured key/values. DO NOT use `System.out.println`. Tag each
   line with `[DEBUG-TRACE]` so we can strip them later.
   - Filter chain entry / exit.
   - Controller method entry with request DTO (PII-redacted).
   - Service method entry / exit with domain args.
   - Repository call + returned row count.
   - Exception handler hit, if any.

3. **Reproduce via Postman MCP**:
   - Locate or create a Postman collection for this service.
     Use `postman_search_collections` or fall back to an ad-hoc request.
   - Send the failing request with realistic auth (test token from
     `.env.test`, never production credentials).
   - Capture request headers (REDACTED: `Authorization`, `Cookie`), body,
     response status, headers, body, and timing.
   - Repeat the known-good equivalent and diff the two.

4. **Correlate** — line up the captured HTTP exchange against the debug
   log output. Identify where expected ≠ actual (status mapping, missing
   filter, wrong principal, bad DTO binding, transaction rollback, etc.).

5. **Propose a fix** — specific file + line + reasoning. Do not apply yet;
   show the diff first.

6. **Clean up** — after I approve the fix, remove every `[DEBUG-TRACE]` log
   line and re-run `./gradlew test` to verify nothing regressed.

Constraints: never use production credentials. Never modify production
config. Never disable security (CSRF, CORS, auth) to "make it work".
