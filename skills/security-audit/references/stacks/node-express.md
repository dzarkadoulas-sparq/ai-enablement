# Security audit: Node.js + Express (TypeScript)

## Read first

- `references/language-specific/node-express-security.md`
- `commands/node-express/security-scan.md` (ai-enablement)

## Dependency scan

- `pnpm audit --audit-level=high`; include **transitive** **HIGH** on direct deps you add or upgrade.

## Code scope (typical)

- `src/**/router*`, `**/controller*`, `**/middleware*`, `**/schema*`, `prisma/**`, `package.json`, lockfile.

## Tools (if in repo)

- **Semgrep**, **eslint-plugin-security** — run the same way CI does.

## Focus

- **Zod** (or project validator) on all inputs; **raw** SQL/Prisma; **CORS** + **Helmet**; **public** route list.
