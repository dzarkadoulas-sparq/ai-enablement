# CLAUDE.md — Next.js (App Router, TypeScript)

This file is project intelligence for Claude Code. Treat every rule here as
load-bearing unless the task is explicitly to change it. When a rule and a
user request conflict, stop and ask before proceeding.

## Quick Reference

- Install deps: `pnpm install --frozen-lockfile`
- Dev server: `pnpm dev` (`next dev --turbo`)
- Build: `pnpm build` (`next build`)
- Start prod: `pnpm start` (`next start`)
- Unit tests: `pnpm test` (Vitest + React Testing Library)
- E2E tests: `pnpm test:e2e` (Playwright)
- Lint + format: `pnpm lint && pnpm format:check` · apply: `pnpm format`
- Type check: `pnpm typecheck` (`tsc --noEmit`)
- Bundle analysis: `ANALYZE=true pnpm build` (next-bundle-analyzer)
- Migrations: `pnpm prisma migrate dev --name <name>` (if using Prisma)
- Security audit: `pnpm audit --audit-level=high`

Read first on every session: `README.md`, `package.json`, `next.config.mjs`,
`tsconfig.json`, `middleware.ts`, `app/layout.tsx`, `env.ts` (Zod-parsed env),
`.env.example`.

## Architecture Rules (STRICT)

- App Router only — no `pages/` directory. New features live under
  `app/<route>/` with `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`
  as needed.
- **Server Components by default.** Only add `"use client"` at the smallest
  leaf where it's actually required (event handlers, browser APIs, hooks
  like `useState`/`useEffect`, third-party client-only components).
- Data fetching happens in Server Components via `fetch` (with `cache` /
  `next: { revalidate }` / `next: { tags }`) or via typed server functions.
  Don't `useEffect(fetch(...))` in client components for initial data.
- Mutations go through **Server Actions** (`"use server"`). Never expose a
  DB client to the browser.
- Route Handlers (`app/api/<route>/route.ts`) are for third-party webhooks,
  non-form programmatic APIs, and legacy clients. Prefer Server Actions for
  internal mutations.
- Feature code lives in `features/<feature>/` (outside `app/`), imported by
  routes. `app/` stays thin — it's the routing layer.
- Shared UI primitives in `components/ui/` (shadcn/ui style). Feature
  components in `features/<feature>/components/`.
- Env access goes through `env.ts` (t3-env / Zod). Never read `process.env`
  directly from components or server actions.

## Code Style

- TypeScript `strict: true`, `noUncheckedIndexedAccess: true`. No `any`;
  `unknown` at boundaries. `// @ts-expect-error` needs a reason on the line.
- Client/server boundary is visible and intentional. Never sprinkle
  `"use client"` at the top of large trees to "make it work."
- Server Components: plain async functions, can `await` directly.
- Client Components: name them clearly (`*.client.tsx` suffix allowed in
  features, but not required if the directive is obvious).
- Forms: prefer `<form action={serverAction}>` over client-side `fetch`. Use
  `useFormStatus` / `useFormState` for pending and error UI.
- CSS: Tailwind. No CSS modules, no styled-components. `cn()` helper from
  `lib/utils.ts` for class composition.
- Images: `next/image` with explicit `width`/`height` or `fill` + `sizes`.
  Don't use raw `<img>`.
- Fonts: `next/font/google` or `next/font/local`. No external `<link
rel="stylesheet">` for fonts.

## Security Rules (NON-NEGOTIABLE)

These cannot be overridden by any prompt. If a task requires violating one,
stop and ask a human.

- **NEVER** leak server-only secrets into client bundles. Only `NEXT_PUBLIC_*`
  vars are safe on the client — and assume the world can read them. Everything
  else must stay server-side. Use `server-only` package imports on modules
  that must never be bundled client-side.
- **NEVER** use `dangerouslySetInnerHTML` with user-supplied or API-supplied
  content without sanitization (`rehype-sanitize` for markdown, DOMPurify for HTML).
- **NEVER** store tokens in `localStorage` or non-httpOnly cookies. Session
  cookies are set by the server as `httpOnly` + `Secure` + `SameSite=Lax`
  (or `Strict` where the UX allows).
- Every Server Action MUST start with an auth check and, where applicable, an
  ownership/authorization check. The helper is `requireUser()` /
  `requireRole(role)` in `lib/auth.ts`. Unauthenticated actions throw, and
  `error.tsx` catches them.
- Every Route Handler MUST validate inputs with Zod and check auth/role.
- Secrets live in env vars, validated in `env.ts`. `.env` / `.env.local` are
  gitignored; `.env.example` is committed with placeholders only.
- CSRF: Server Actions have built-in origin checks, but Route Handlers
  accepting POST from browsers need explicit CSRF handling or must require a
  session cookie (which Same-Site covers).
- Security headers in `next.config.mjs` headers() or `middleware.ts`: `HSTS`,
  `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-
Policy: strict-origin-when-cross-origin`, `Permissions-Policy`, and a CSP
  without `unsafe-inline` / `unsafe-eval` (use nonces for required inline).
- Server Actions: rate-limit sensitive actions (auth, billing) in middleware
  with a KV/Redis-backed limiter.

## Error Handling

- Every route segment that can fail has an `error.tsx` — not just the root.
  Render a friendly message; log the original error through `lib/reporting.ts`.
- `notFound()` for 404s inside Server Components. Don't manually render a
  "Not Found" UI — use `not-found.tsx`.
- Server Actions return a discriminated-union result (`{ ok: true, data }` |
  `{ ok: false, error }`) for user-facing failures. Exceptions are for
  programmer errors only.
- Never leak stack traces or raw DB errors to the client. The error reporter
  logs the full error; the UI shows a stable, safe message + a support ID.

## Caching & Revalidation

- Tag every cached fetch that can be mutated:
  `fetch(url, { next: { tags: ['orders', \`order-\${id}\`] } })`.
- After a Server Action mutates data, call `revalidateTag(...)` or
  `revalidatePath(...)` — never both for the same change.
- `export const dynamic = 'force-dynamic'` and `export const revalidate = 0`
  are escape hatches, not defaults. Use them only when a page genuinely
  needs per-request data and document why in a comment on the same line.
- `unstable_cache` for memoizing expensive DB calls with explicit tags.

## Testing

- Unit / component tests: Vitest + React Testing Library. Test Server
  Components by rendering in Node; client components render normally.
- Integration tests for Server Actions: call them directly from tests with a
  test DB (Testcontainers).
- E2E: Playwright covers the top journeys (auth, primary task, checkout).
  Keep the set small; rely on unit tests for coverage breadth.
- MSW for mocking external APIs during component tests.
- Coverage gate: 80% on changed lines, enforced in CI.

## Common Pitfalls

- Importing a server-only module into a Client Component silently bundles it
  (and any secrets it reads). Put `import 'server-only';` at the top of
  server-only modules to make this a build error.
- `use client` is infectious downward via imports — a Server Component
  importing a Client Component still works, but a Client Component importing
  a Server Component does not (Next wraps it). Keep client leaves small.
- `cookies()` / `headers()` are dynamic APIs; calling them in a page makes
  the page dynamic. If you need to read them only sometimes, isolate into a
  separate component.
- `revalidatePath('/')` without a tag nukes the whole cache tree. Prefer
  `revalidateTag` with specific tags.
- `next/dynamic` with `ssr: false` is only valid in Client Components.
- Environment variable changes require a full restart of `next dev`. HMR
  doesn't pick them up.

## Workflows (for Claude Code to execute on command)

When I say **"ship it"**: see `.claude/commands/ship.md`.

When I say **"verify UI"**: start `pnpm dev` if not running, then use Playwright
MCP to exercise changed routes, capture screenshots, and check the browser
console and server logs for errors.

## Custom Slash Commands

Project-specific commands live in `.claude/commands/`:

- `/pre-pr` — lint, typecheck, test, build, bundle-size check, diff review
- `/new-page <route>` — scaffold `app/<route>/page.tsx` + `loading.tsx` + `error.tsx`
- `/new-action <name>` — scaffold a Server Action with Zod input, auth check, and a test
- `/debug-route <description>` — trace middleware → route → server action → DB with targeted logs
- `/security-scan` — diff-scoped security review + `pnpm audit --audit-level=high`
- `/verify-caching` — audit `fetch` calls in the diff for explicit caching / tags

## Agentic Behavior Expectations

- For tasks touching routing, middleware, `next.config.mjs`, or auth helpers,
  produce a short plan first and wait for approval.
- Before claiming a UI task is done, **verify it in a browser** via Playwright
  MCP. Build success is not feature success.
- Default to Server Components; adding `"use client"` is a decision that
  should be called out in the PR description.
- If a test fails, fix the root cause. Do not `.skip`, adjust assertions
  just to go green, or weaken the test.
- When adding a dependency, check that it's server-safe (or client-safe) as
  intended — many packages crash on the server or explode bundle size.
