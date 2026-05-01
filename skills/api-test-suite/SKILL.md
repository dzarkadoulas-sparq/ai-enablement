---
name: api-test-suite
description: >-
  Discovers HTTP endpoints from code or OpenAPI, structures tests by resource,
  and generates or drafts coverage for happy paths, validation, auth, pagination,
  and multi-step flows with schema or contract checks. Optional Postman MCP to run
  or sync collections. Reports which endpoints are covered. Use for API testing,
  Postman collections, contract tests, or validating API behavior.
---

# API test suite

## Goal

Build **repeatable** **API** **verification**: either **in-repo** tests (Vitest+supertest, pytest+httpx, xUnit+WAF, etc.) and/or a **Postman** **collection** **aligned** to **OpenAPI** or **route** **definitions**, with a **matrix** of **test** **categories** and **workflows** from the references. End with a **coverage** **report** of **methods+paths** **covered** vs **discovered**.

## Preconditions

- **Source of truth** — `openapi.json` / `openapi.yaml` **in** **repo**, or **routable** code (**Express** **routers**, **FastAPI** **routers**, **Spring** **controllers**, **ASP.NET** **MapGet/MapPost**, **Next** `route.ts`). If **missing** **OpenAPI**, **infer** from **code** and **mark** **inference** in the **report**.

## When stack is unknown

1. **Detect** API style using `package.json` / `pyproject` / Gradle / `.sln` (as in `project-bootstrap`).
2. **Read** `references/test-categories.md` and `references/workflow-templates.md` (always).
3. If **Postman** is in scope, read `references/postman-mcp-patterns.md`.
4. **Read** `references/stacks/<id>.md` for **discovery** **patterns** and **test** **stack**.

| `<id>`            | API surface in typical repo |
| ----------------- | ----------------- |
| `react`           | **Client** to **BFF/REST** — **MSW** or **e2e**; **rare** **in-repo** **HTTP** **server** |
| `next`            | `app/**/route.ts` **Route** **Handlers** + **Server** **Actions** (document **as** **HTTP** or **form** **flows**) |
| `angular`         | **HttpClient** — **in-memory** or **E2E**; **public** **API** often **separate** **service** **repo** |
| `node-express`    | Express (or **Fastify**) **routers** + **Zod** **schemas** |
| `python-fastapi`  | **FastAPI** `APIRouter` — **OpenAPI** **auto** from **Pydantic** |
| `java-springboot` | `@RestController` + **OpenAPI** **(springdoc, etc.)** if **present** |
| `dotnet`          | **Minimal** **API** or **Controller**; **NSwag** / **Swashbuckle** for **OpenAPI** |

## Workflow (execute in order)

1. **Discover** — List **(method, path, tag/resource)**. Prefer **OpenAPI** **merge** of **all** **servers** if **monorepo** has **more** than **one** **app**. If **code-only**, use **stack** file **heuristics** and **rg** for **path** **strings** (note **caveats** for **dynamic** **segments**).

2. **Structure** — Folders or **collection** **folder**s by **resource** (e.g. `orders`, `auth`) and **naming** **convention** in `references/test-categories.md`.

3. **Per-endpoint** **tests** — For **each** **or** **prioritized** **P0** **routes**: **2xx** **happy** **path**, **4xx** **validation** (body/query), **401/403** **if** **protected** (and **one** **allow** for **public** if **applicable**), **edge** (pagination, `If-Match`, **id** **format**) per **test-categories.md**.

4. **Workflows** — From `workflow-templates.md`, add **chained** **tests** (token **→** **create** **→** **get** **→** **state** **change** **→** **idempotent** **retry** where **relevant**). **Store** **vars** in **Postman** **/** **environment** or **test** **fixtures**.

5. **Schema** — Assert **status** + **body** **shape**: **Zod**/**JSON** **Schema** in **code**; **pm.response.json** + **ajv** in **Postman**; **Pydantic**-exported **JSON** **Schema** from **FastAPI** when **available**.

6. **Postman** **MCP** (optional) — If **MCP** **is** **available** and the **user** **wants** **it**, use `postman-mcp-patterns.md` to **list** **workspace/collections**, **update** or **create** **requests**, **run** (if the **MCP** **exposes** **it**), else **export** **collection** **JSON** and **document** **manual** **import**.

7. **Report** — **Table**: **Path**, **Method**, **Categories** **covered** (H/V/A/E/… as **defined** in test-categories), **Gap** (none / missing **auth** / no **4xx**). **Optional** **%** of **P0** **endpoints** **covered** if you **define** **P0** (public + **auth**-critical) **from** the **user**.

## Anti-patterns

- **Only** **200** **assertions** for **endpoints** that must **return** **422/404/409** in **cases**.
- **Hardcoding** **production** **URLs** or **tokens** in **committed** **files**; use **env** / **secrets** **MCP** **or** **Postman** **environments** **(masked)**.
- **Ignoring** **idempotency** and **concurrency** **(ETags,** **Idempotency-Key)** for **write** **APIs** that **claim** to **support** them.

## Reference map

- `references/test-categories.md` — **matrix** of **kinds** of **tests** **per** **request**.
- `references/workflow-templates.md` — **multi-step** **templates**.
- `references/postman-mcp-patterns.md` — **MCP** **usage** **+** **links** **to** **Postman** **docs**.
- `references/stacks/<id>.md` — **discovery** + **native** **test** **harness** **for** **that** **stack**.

Valid `<id>` values: `react`, `next`, `angular`, `node-express`, `python-fastapi`, `java-springboot`, `dotnet`.
