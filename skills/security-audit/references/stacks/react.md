# Security audit: React (Vite + TypeScript)

## Read first

- `references/language-specific/react-frontend-security.md`
- `commands/react/security-scan.md` (ai-enablement — diff-scoped checklist + output format)

## Dependency scan

- `pnpm audit --audit-level=high` (or `npm` / `yarn` per lockfile). Flag **HIGH/CRIT** on **touched** dependencies.
- **GitHub MCP** (if available): Dependabot / advisory info for changed packages.

## Code scope (typical)

- `git diff origin/develop...HEAD -- '*.ts' '*.tsx' 'package.json' 'pnpm-lock.yaml' 'vite.config*' 'index.html'`

## Security sections in CLAUDE

- Cross-check new code against **Security** rules in `claude.md/react/CLAUDE.md` (XSS, tokens, env, CSP).
