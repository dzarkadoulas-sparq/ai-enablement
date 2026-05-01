# Security audit: Angular (v17+)

## Read first

- `references/language-specific/angular-frontend-security.md`
- `commands/angular/security-scan.md` (ai-enablement)

## Dependency scan

- `pnpm audit --audit-level=high` (or `npm` per project).

## Code scope (typical)

- `src/**`, `angular.json`, `package.json`, lockfile; **HttpClient** interceptors, **guards**, **resolvers** with **auth** side effects.

## Focus

- **DomSanitizer** bypass; **innerHTML**; **environment** URLs; **CORS** with **withCredentials**; **XSRF** alignment with **backend**.
