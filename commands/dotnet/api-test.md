Run an API test pass for the changed endpoints using **Postman MCP**.

1. **Identify changed endpoints** — `git diff origin/develop...HEAD` on
   `src/Api/Endpoints/**` and `src/Api/Controllers/**`. List each as
   `METHOD /path (Handler or Controller#method)`.

2. **Locate the collection** — via Postman MCP (`postman_search_collections`)
   find the project's collection. If none exists, create one named
   `<service>-api-tests` in the team workspace.

3. **Ensure a request exists per changed endpoint** — for any missing:
   - Correct method + path variables.
   - Auth header from the collection-level `{{bearerToken}}` variable,
     populated by a pre-request script that hits the auth endpoint with
     test creds pulled from the `Staging` environment + Postman vault.
   - Body matching the request DTO (`Create<Feature>Command`, etc.).
   - Tests asserting status, `Content-Type`, key response fields, and
     SLA (`pm.expect(pm.response.responseTime).to.be.below(1000)`).

4. **Run the collection** via Postman MCP against the `Staging` environment.
   NEVER production.

5. **Failures** — for each failing request, open the handler / validator /
   repository and propose a fix. Do not apply without confirmation.

6. **Summary** — total requests, pass/fail count, SLA violations, and the
   Postman run URL. Do not commit Postman environment files containing
   real tokens; use variables backed by the Postman vault.

Constraints: never use production credentials; never target production
hosts; never log raw `Authorization` header values.
