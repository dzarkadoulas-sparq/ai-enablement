# Stack: Next.js (App Router, TypeScript)

## Detection

- `next` in `package.json`.
- `next.config.*`; `app/` (App Router) and/or `pages/`.

## Read first

`README.md`, `package.json`, `next.config.*`, `tsconfig.json`, `middleware.ts` if present, `app/layout.tsx`, env handling (`env.mjs` / `env.ts` / t3-env), `.env.example`.

## Trace one flow

**Server-first:** a representative `app/**/page.tsx` — data fetch (RSC) or props — into UI. **Mutations:** Server Action or `app/api/.../route.ts` if that’s what the app uses. Note **cache** / **revalidate** if used.

## `CLAUDE.md` sections to generate

1. **Quick Reference** — `next dev`, `next build`, `next start`, test/lint from manifests.
2. **Read first** — layout, middleware, env, Prisma or DB config if present.
3. **Architecture** — App Router vs Pages, Server vs client components, where `use client` is allowed, Server Actions vs Route Handlers, colocation of `features/` or `lib/`.
4. **Data** — `fetch` caching, revalidation, tags, no secrets in client bundles.
5. **Code style** — TypeScript, RSC-friendly patterns, forms (Server Actions + `useFormStatus` / validation lib if present).
6. **Security** — env separation, auth callbacks, headers in `next.config` / middleware, CSP if configured.
7. **Testing** — unit (Vitest/Jest) and E2E (Playwright) from `package.json`.
8. **Pitfalls** — serializing non-POJOs to client, `useEffect` for initial data that should be RSC, large client bundles.
9. Conflict / stop-and-ask line for rule vs. request.

## Stack red flags

- Database or secrets imported into client components.
- `getServerSideProps` mixed with App Router without an architectural note (migration debt).
- `process.env` used in client code instead of validated env module.

## Exemplar (ai-enablement)

`templates/next/CLAUDE.md`
