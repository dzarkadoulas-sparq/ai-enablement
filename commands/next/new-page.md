Scaffold a new App Router page. Route: `$ARGUMENTS` (e.g. `orders/[id]`).

Default to Server Components. Only add `"use client"` at the smallest
required leaf.

1. **Create** `app/<route>/`:
   - `page.tsx` — default-exported async Server Component. Receives
     `params` and `searchParams` with the correct shape. Calls a typed
     data-fetcher from `features/<feature>/data.ts` with explicit caching
     (e.g. `{ next: { tags: ['feature', \`feature-\${id}\`] } }`).
   - `loading.tsx` — skeleton that matches the page layout.
   - `error.tsx` — `"use client"`. Logs the error via `lib/reporting.ts`
     and renders a friendly fallback with a reset button.
   - `not-found.tsx` — for `notFound()` throws from `page.tsx`.

2. **If the page takes form input** — create a Server Action file
   `features/<feature>/actions.ts` with:
   - `"use server"` at the top.
   - Zod input validation.
   - `requireUser()` / `requireRole(...)` call as the first statement.
   - Returns `{ ok: true, data } | { ok: false, error }`.
   - Calls `revalidateTag(...)` / `revalidatePath(...)` with a specific
     target (never the bare root).

3. **Client components** (if any) — go in
   `features/<feature>/components/<Name>.client.tsx` with `"use client"`.
   Keep them small and leafy.

4. **Metadata** — export `generateMetadata` from `page.tsx` with a title
   and description. For dynamic pages, use the param to fetch the entity
   name; reuse the cached fetch so it doesn't double-fetch.

5. **Tests**
   - `app/<route>/__tests__/page.test.tsx` — Vitest + RTL, renders the
     Server Component with mocked data.
   - If a Server Action exists: `features/<feature>/__tests__/actions.test.ts`
     invoking the action directly against a test DB (Testcontainers).

6. **Verify**
   - `pnpm lint --fix && pnpm typecheck`.
   - `pnpm build` — any build-time failure means the page isn't ready.
   - Playwright MCP: navigate to the route, screenshot, check console.

7. **Summary** — files created, cache strategy, auth requirements, any
   new env vars, and Playwright verification result.

Respect CLAUDE.md rules: Server Components by default; no `process.env`
outside `env.ts`; every Server Action starts with an auth check.
