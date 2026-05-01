Debug an ASP.NET Core endpoint. Problem: `$ARGUMENTS`.

1. **Identify the route** — grep for the path + method in `src/Api/Endpoints/`
   and related MediatR handlers in `src/Application/Features/`. Read:
   - The endpoint registration.
   - The MediatR command/query + handler.
   - The validator (FluentValidation).
   - The repository or EF query used.
   - Middleware order in `Program.cs` (auth, exception handler, CORS).

2. **Add targeted debug logging** via `ILogger<T>` at each boundary.
   Use structured templates and tag with `"[DEBUG-TRACE]"` so we can grep
   them out. NEVER `Console.WriteLine`.
   - Endpoint hit (method, route, authenticated user id).
   - Validator pass/fail + which rule failed.
   - Handler entry / exit with domain args (PII-redacted).
   - Repository call + returned count.
   - Exception handler hit, if any.

3. **Reproduce via Postman MCP**
   - Locate the project collection (`postman_search_collections`). If none
     exists for this service, create a scratch request.
   - Call the failing endpoint against the `Staging` environment with a
     test token from Postman's vault. NEVER production credentials.
   - Capture: request headers (REDACT `Authorization`, `Cookie`), body,
     response status, headers, body, and duration.
   - Also capture a known-good variant for comparison.

4. **Correlate** the HTTP exchange with the server log output. Find where
   expected ≠ actual (wrong policy, missing claim, DTO binding failure,
   EF tracking issue, transaction rollback, etc.).

5. **Propose a fix** — specific file + line + reasoning. Do not apply
   without my confirmation.

6. **Clean up** — after the fix is approved, strip every `[DEBUG-TRACE]`
   log line and re-run `dotnet test` to confirm no regression.

Constraints: never use production credentials; never disable auth, CORS,
or antiforgery to "make it work"; never set `ValidateLifetime = false` on
JWT options.
