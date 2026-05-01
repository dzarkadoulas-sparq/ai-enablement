# Refactor guide: React (Vite + TypeScript)

## Verify after each phase

- `pnpm lint`, `pnpm typecheck`, `pnpm test` (or scripts from `package.json` / `templates/react/CLAUDE.md`).

## Refactor-specific notes

- **Extract component** — move props interface with the component; keep **public** exports through feature `index.ts` if that’s the team pattern.
- **State** — splitting **Zustand** / **TanStack Query** usage: avoid double subscriptions; prefer **selectors** when extracting hooks.
- **Hooks** — custom hooks that touch context or router: update **all** call sites in one phase when the hook contract changes.

## Pitfalls

- **Hot reload** can hide missing dependency array issues—run **full test** + **build** before declaring done.
