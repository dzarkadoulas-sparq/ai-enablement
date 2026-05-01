# React (Vite) — client security focus

## XSS and HTML

- `dangerouslySetInnerHTML` without **rehype-sanitize** (or project sanitizer); **Markdown** without sanitization.
- **`target="_blank"`** without `rel="noopener noreferrer"`.
- **User** or **API** strings in **`href` / `src`** without allowlist or `javascript:` block.
- **postMessage** handlers without **origin** check.

## Secrets in the browser

- Anything under **`VITE_*`** is **public** — no server keys, DB passwords, private API keys for server-only services.
- **Auth tokens** in `localStorage` / **non-HttpOnly** cookies (flag if `CLAUDE.md` forbids and code violates).

## Input

- **URL** / **search** params — validate with **Zod** (or project schema) before use in API calls or render.

## Third-party

- **Script** tags: prefer **integrity** + **crossorigin** for CDN.
- **CSP** in Vite / `index.html`: flag **unsafe-inline** / **unsafe-eval** workarounds.

## Tools

- `pnpm audit --audit-level=high` on **lockfile** and **package.json** changes.
- **Semgrep** / **ESLint** security plugins if in repo.

## See also

`commands/react/security-scan.md` in ai-enablement for a diff-scoped checklist.
