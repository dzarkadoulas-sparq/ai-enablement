# Stack: React (Vite + TypeScript) SPA

## Detection

- `react` + `vite` in `package.json`; often `index.html` at repo root; `vite.config.*`.
- Not Next: absence of `next` dependency; no `next.config.*` as the primary app.

## Read first

`README.md`, `package.json`, `vite.config.*`, `tsconfig*.json`, `index.html`, `src/main.tsx`, top-level `src/App.tsx` or `src/routes`, env sample (`.env.example`, `src/config/env.ts`).

## Trace one flow

User navigation: router definition → page/screen component → data hook (e.g. TanStack Query) → `fetch` or API client module. Note **where errors and auth** are handled.

## `CLAUDE.md` sections to generate

1. Title line: **CLAUDE.md — React (Vite + TypeScript) SPA** (or match repo’s actual stack).
2. **Quick Reference** — real scripts from `package.json` (install, dev, build, test, lint, typecheck, e2e if present).
3. **Read first on every session** — concrete paths (adjust to repo).
4. **Architecture Rules (STRICT)** — feature folders, state layering (server vs client), routing, API client boundary, asset/public rules.
5. **Code Style** — TypeScript strictness, component patterns, a11y baseline, styling approach found in repo.
6. **Security Rules** — client-exposed env, `dangerouslySetInnerHTML`, tokens, CORS, links to sanitizer if markdown/HTML is used.
7. **Testing** — unit vs e2e commands and co-location of tests.
8. **Workflow / Pitfalls** — if detectable: bundle budgets, Conventional Commits, branch naming from CI.
9. Conflict line: _When a rule and a user request conflict, stop and ask before proceeding._

## Stack red flags

- `fetch` scattered in components with no shared client.
- Auth tokens in `localStorage` without a documented exception.
- Missing error boundary or global error UI when router exists.
- No a11y lint or tests when app has many interactive components.

## Exemplar (ai-enablement)

`claude.md/react/CLAUDE.md` — use as full structural template when working inside this training repo.
