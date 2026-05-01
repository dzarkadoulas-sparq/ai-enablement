# Next.js (App Router) testing notes

## Split by boundary

- **Client components** (`"use client"`): test with Vitest + RTL like React; same patterns as `react-typescript.md`.
- **Server Components** (no `use client`): do **not** run them through RTL like client trees—use **direct unit tests** on **extracted pure** functions, **server helpers**, or **integration** tests with `next`’s test utilities if the project has them, or e2e (Playwright) for full page.
- **Server Actions** — unit-test **validation** and **auth gating** in pure functions; integration: call the action in a test harness the repo uses (see existing tests), or e2e.

## Mocking

- `next/navigation` — mock `useRouter`, `usePathname`, `useSearchParams` as existing tests do.
- `next/image` — may need jest/vitest mock or pass-through; copy project pattern.

## Env

- `process.env` for tests: use `.env.test` or vitest `env` block; **never** rely on real `NEXT_PUBLIC_*` secrets in tests.

## Caching

- If testing code that uses `fetch` with Next cache tags, control **deduping** in test or use **MSW** to assert request counts.

## Anti-pattern

- Full **RSC** tree in RTL with fake **Request** objects for every test—prefer small units or e2e when the project already does.
