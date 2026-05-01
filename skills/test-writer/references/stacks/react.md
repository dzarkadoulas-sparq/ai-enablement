# Test writer: React (Vite + TypeScript)

## Language references

- `references/react-typescript.md` — Vitest, RTL, userEvent, MSW.
- `references/edge-case-catalog.md`, `references/anti-patterns.md`

## Detect / locate tests

- Vitest + `@testing-library/react` from `package.json`; config in `vitest.config.*`.
- Colocate `*.test.tsx` with components or under `__tests__/` per existing tests.

## Commands (from manifests — do not invent)

- Unit + coverage: typically `pnpm test --coverage` (see `templates/react/CLAUDE.md` / `package.json`).
- A11y: `pnpm test:a11y` when adding UI.

## Focus for this stack

- **User-visible** behavior; mock network with **MSW** if app uses fetch/client API layer.
- **Router + QueryClient** wrappers from existing test setup.

## Exemplar context

`templates/react/CLAUDE.md` — test commands and accessibility expectations.
