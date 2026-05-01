Run an API test pass for the changed endpoints using **Postman MCP**.

1. **Identify changed endpoints** — `git diff origin/develop...HEAD` on
   controllers under `src/main/java/.../api/`. List each as
   `METHOD /path (Controller#method)`.

2. **Locate the collection** — use Postman MCP to find the project's
   collection (`postman_search_collections` with the project name). If none
   exists, create one named `<service>-api-tests` in the team workspace.

3. **Ensure a request exists per changed endpoint** — for any missing one,
   create it with:
   - Correct method + path variables.
   - Auth header from the collection-level `{{authToken}}` variable
     (populated by a pre-request script hitting the auth endpoint with
     test creds from the `Staging` environment).
   - Body matching the request DTO schema.
   - Tests (Postman JS) asserting status, `Content-Type`, key response
     fields, and SLA (`pm.expect(pm.response.responseTime).to.be.below(1000)`).

4. **Run the collection** against the `Staging` environment using
   Postman MCP (`postman_run_collection` or equivalent). Do NOT run against
   production.

5. **Failures** — for each failing request, open the Controller / Service
   and propose a fix. Do not apply without confirmation.

6. **Summary** — total requests, pass/fail count, any SLA violations,
   and the Postman run URL. Do not commit Postman environment files that
   contain real tokens — use variables backed by the Postman vault.

Constraints: never use production credentials, never target production hosts,
never log raw `Authorization` header values.
