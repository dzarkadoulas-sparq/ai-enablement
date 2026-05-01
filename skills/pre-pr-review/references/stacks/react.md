# Pre-PR gate: React (Vite + TypeScript) SPA

## Baseline branch

Use `origin/develop` or the repo’s default (see `CLAUDE.md` / CI). Example diff: `git diff origin/develop...HEAD`.

## Gate order (stop on first failure unless user asks otherwise)

1. **Lint + format** — `pnpm lint --fix`, `pnpm format`, then `pnpm lint && pnpm format:check`.
2. **Typecheck** — `pnpm typecheck` (`tsc -b --noEmit`). No new `// @ts-expect-error` without a same-line reason.
3. **Unit + component tests** — `pnpm test --coverage`.
4. **Accessibility** — `pnpm test:a11y`. New component without a `jest-axe` (or project equivalent) assertion = fail.
5. **Build + bundle budget** — `pnpm build`; `pnpm build:analyze` vs `bundle-baseline.json` if present; >10% chunk growth without justification = fail.
6. **Security** — `pnpm audit --audit-level=high`.
7. **Diff hygiene** — see **Diff hygiene (React)** below.
8. **Optional** — Playwright MCP on changed routes (light/dark, console). See `commands/react/pre-pr.md` in ai-enablement.
9. **Summary** — commits, files by feature, PASS/FAIL, 2–3 bullet PR blurb. **Do not push or open PR** unless asked.

## Diff hygiene (React)

- `console.log` / `debugger` in committed code.
- `.skip` / `.todo` tests without linked ticket.
- Commented-out JSX.
- `dangerouslySetInnerHTML` without sanitizer.
- `any` / `as unknown as X` without justification.
- Hardcoded URLs — must go through `src/config/env.ts` (or project env module).
- Tokens in `localStorage` / non-httpOnly cookies.
- Icon-only buttons without `aria-label`.

## Exemplar

`commands/react/pre-pr.md` in the ai-enablement repo.
