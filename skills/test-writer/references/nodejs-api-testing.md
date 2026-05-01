# Node.js + Express (API) testing

## Stack (detect from `package.json`)

- **Unit:** Vitest or Jest — test **pure** functions, **Zod** schemas, mappers, **use cases** with fake repos.
- **HTTP integration:** **`supertest`** (or `light-my-request` for some stacks) against **non-listening** `app` export, or as the repo does with a test server and dynamic port.

## Layout

- Colocate `*.test.ts` with feature or under `src/features/**/__tests__/`; follow existing tests.
- **E2E** vs **integration:** integration hits **in-memory** or **test** DB; match Docker compose from `README` if `test:integration` is documented.

## Supertest pattern

- `import request from 'supertest'`
- `import { app } from '../app'`
- `const res = await request(app).get('/api/...').set('Authorization', 'Bearer test-token')`
- Assert **status**, **body shape** (Zod-parsed in route), **not** every header unless contract requires.

## Mocks

- **DB/Prisma** — use **test DB**, **transaction rollback** per test, or **repository fakes** as the project does; do not mock `PrismaClient` in 20 places if `testcontainers` is standard.

## Auth

- Reuse test **helpers** for JWT/session headers; do not copy full tokens—use project’s `auth as user X` util.

## Env

- Load `NODE_ENV=test` and `.env.test` before importing `app`; **never** point tests at **prod** URLs.
