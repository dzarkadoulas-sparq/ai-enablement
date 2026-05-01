# Test writer: Node.js + Express (TypeScript)

## Language references

- `references/nodejs-api-testing.md` — supertest, app export, env.
- `references/edge-case-catalog.md`, `references/anti-patterns.md`

## Detect / locate tests

- Vitest or Jest; **supertest** (or project HTTP helper) in `__tests__` or `*.test.ts` next to `router`/`controller`.
- **Zod** / validation: **table-driven** unit tests for schema edge cases.

## Commands

- `pnpm test --coverage`; `pnpm test:integration` for DB-backed tests (Docker as documented).

## Focus

- **Layering:** test **service** and **schema** in unit; **supertest** for **route** contract (status + error shape).
- **Auth middleware** with test tokens from project helpers.

## Exemplar context

`templates/node-express/CLAUDE.md`
