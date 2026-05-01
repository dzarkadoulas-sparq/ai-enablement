# API test suite: React (Vite) SPA

## Role

- A typical React app has **no in-process HTTP server**. “API tests” here usually mean one of:
  - **Contract / integration** tests against a **running backend** (E2E or a separate CI job), or
  - **MSW** handlers that **mock** the API for UI tests (not the same as verifying a real server—**call this out** in the report).

## When this stack is the target

- If the user wants “API tests” in a **React-only** repo, recommend testing the **separate** `node-express` or `python-fastapi` (etc.) service with this skill’s backend-oriented stacks, or **E2E** (Playwright) for user journeys.

## MSW (optional)

- Align MSW handlers to **OpenAPI** when a spec exists. For components that `fetch` on user action, state coverage as **“client mocked routes”** rather than **“server endpoints executed”**.

## Discovery

- List path prefixes and HTTP methods from `src/api/` (or similar) by reading **fetch/axios** wrappers—use as a **client contract checklist** for what the UI expects from the backend.
