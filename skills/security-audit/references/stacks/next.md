# Security audit: Next.js (App Router)

## Read first

- `references/language-specific/nextjs-security.md` (+ `react-frontend-security.md` for client components)
- `commands/next/security-scan.md` (ai-enablement)

## Dependency scan

- `pnpm audit --audit-level=high`. Run `pnpm build` if **env** / **RSC** boundary changed — catches mis-split secrets.

## Code scope (typical)

- Include `app/`, `middleware.ts`, `next.config.*`, `package.json`, lockfile, `env` modules.

## Focus

- **Server Actions** auth + validation; **Route Handlers**; **NEXT*PUBLIC*\*** misuse; **cache** behavior for user-specific data.
