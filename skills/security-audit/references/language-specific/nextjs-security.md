# Next.js (App Router) — security focus

## Server vs client boundary

- **Server-only** secrets must **never** appear in client bundles — no `NEXT_PUBLIC_*` for DB, admin keys, or signing keys.
- **Server Actions** — **authz** on every action (role/tenant); **Zod** (or equivalent) for input; no **unchecked** `formData` to DB.
- **Route Handlers** (`route.ts`) — same as API: **auth**, **validation**, **rate limit** if project standard.

## Data fetching

- **`fetch`** in RSC: default **caching** can leak data between users if URL is user-influenced — confirm **cache** / **tags** / **no-store** per sensitivity.
- **Revalidation** — avoid **broad** `revalidatePath` that invalidates sensitive data for everyone.

## Headers

- **`next.config`**: `headers()`, `Content-Security-Policy`, **HSTS** if TLS termination is at app.
- **Middleware** — auth redirect, **CSP** injection, **CORS** for API routes.

## Client

- Same **XSS** / **env** rules as **React** for **client components** (`react-frontend-security.md`).

## Tools

- `pnpm audit --audit-level=high`.
- **Next** build may fail on **env** violations — run `pnpm build` in audit if config changed.

## See also

`commands/next/security-scan.md` in ai-enablement.
