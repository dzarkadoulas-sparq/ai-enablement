Debug a Next.js route using **Playwright MCP**. Problem: `$ARGUMENTS`.

1. **Classify the route**
   - App Router page (`app/<route>/page.tsx`) — Server Component by default.
   - Route Handler (`app/api/<route>/route.ts`).
   - Server Action (`"use server"` function called from a form / client).
   - Middleware (`middleware.ts`).

2. **Read the relevant files**
   - The page / handler / action / middleware.
   - Data fetchers (`features/<feature>/data.ts`) and the cache flags used.
   - Auth helpers in `lib/auth.ts`.
   - Any `revalidateTag` / `revalidatePath` calls touching this route.

3. **Reproduce**
   - Start `pnpm dev` if not running.
   - Use Playwright MCP to navigate to the route and perform the exact
     user steps.
   - Capture: screenshot at failure, console messages, network requests
     (include server response bodies for 4xx/5xx, redact auth headers).
   - Also capture the dev server's stdout for this request — Next.js logs
     RSC payload errors, cache misses, and Server Action traces there.

4. **Add targeted debug logging**
   - In Server Components / Server Actions: `console.log('[DEBUG-ROUTE]', ...)`
     — this appears in the dev-server logs, not the browser.
   - In Client Components: `console.debug('[DEBUG-ROUTE]', ...)`.
   - Tag every temporary line so we can strip them.

5. **Common failure modes to consider**
   - `"use client"` missing where a hook / event handler runs.
   - `"use client"` imported into a Server Component — works, but the
     reverse fails; check which is which.
   - Cache returning stale data (`fetch` without `cache: 'no-store'` or a
     `revalidateTag` that never fires).
   - `cookies()` / `headers()` called in a component that forces the whole
     tree to be dynamic when you didn't intend it.
   - Middleware matcher too broad or too narrow.
   - Server Action not receiving the expected form fields (check the
     `<form action={...}>` wiring).
   - Secret leaking into the client bundle (grep output of `pnpm build`).

6. **Correlate** — user action → middleware → page/action → server log →
   network tab. Find where expected ≠ actual.

7. **Propose the fix** — file + line + reasoning. Do not apply without
   confirmation.

8. **Verify** — apply the fix, repeat the Playwright repro, confirm the
   screenshot + console + dev-server log are clean.

9. **Clean up** — remove every `[DEBUG-ROUTE]` log and run
   `pnpm lint && pnpm typecheck && pnpm test`.

Constraints: never log tokens or PII; never disable CSP / auth to "make
it work"; never move server code to the client to bypass a boundary error.
