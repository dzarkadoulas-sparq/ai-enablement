# Refactor guide: Node.js + Express (TypeScript)

## Verify

- `pnpm lint`, `pnpm typecheck`, `pnpm test`, `pnpm build` (or `tsc` emit); `pnpm test:integration` when the phase touches the DB or HTTP stack.

## Refactor-specific notes

- **Routers** — mount paths and `Router()` merge order; **re-run** **supertest** **for** the **affected** **base** path.
- **Zod** / DTOs — a **stricter** schema is a **behavior** change: treat as **separate** commit or **phase** with API notes.
- **Prisma** — renames: **schema** + **migrations** + all **raw** **queries** in the same **phase** when possible.

## Pitfalls

- **Middleware** order: auth **before** body parser where required; moving middleware changes **error** **paths** **in** **tests**.
