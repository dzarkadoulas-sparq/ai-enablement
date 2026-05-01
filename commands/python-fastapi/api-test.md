Run an API test pass for the changed endpoints using **Postman MCP**.

1. **Identify changed endpoints** — `git diff origin/develop...HEAD` on
   `app/features/**/router.py`. List each as
   `METHOD /api/v1/<path> (<module>.<handler>)`.

2. **Locate the Postman collection** via `postman_search_collections`.
   If none exists, create one named `<service>-api-tests`.

3. **Ensure a request exists per changed endpoint** — for any missing:
   - Correct method + path params.
   - `Authorization: Bearer {{bearerToken}}` where the variable is
     populated by a pre-request script that hits `/auth/token` with
     test creds from the `Staging` env + Postman vault.
   - Body matching the Pydantic request model exactly.
   - Tests (Postman JS): status, `Content-Type: application/json`,
     presence of key fields, SLA (`pm.expect(responseTime).to.be.below(1000)`).

4. **Run the collection** via Postman MCP against the `Staging`
   environment. NEVER production.

5. **Failures** — open the router / service / repository and propose a
   fix. Do not apply without confirmation.

6. **Summary** — request count, pass/fail, SLA violations, Postman run URL.
   Do not commit Postman environment files with real tokens; use the vault.

Constraints: never use production credentials; never target production;
never log raw `Authorization` values.
