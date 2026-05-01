# Refactor guide: Next.js (App Router)

## Verify

- `pnpm lint`, `pnpm typecheck`, `pnpm build` (build catches RSC and env boundary issues), `pnpm test` as available.

## Refactor-specific notes

- **Client vs server** — moving code across `"use client"` / Server Components **changes** the **bundle** and what can be imported; do it in a **dedicated** phase.
- **Server Actions** — renaming or splitting actions may break **form** `action` attributes; grep **callers** and E2E.
- **Caching** — `fetch` cache and **tags** must stay **consistent** when you move data-loading code.

## Pitfalls

- **env** access only through the project’s `env` module; refactors that touch config must not leak client secrets.
