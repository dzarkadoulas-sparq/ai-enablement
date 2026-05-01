# Test writer: Next.js (App Router)

## Language references

- `references/react-typescript.md` — client components and RTL.
- `references/nextjs-testing.md` — RSC, Server Actions, `next/navigation` mocks.
- `references/edge-case-catalog.md`, `references/anti-patterns.md`

## Detect / locate tests

- Vitest + RTL; may have **separate** patterns for **server** code (pure function tests in `lib/` or `features/`).
- Follow existing `*.test.ts` / `*.test.tsx` next to `app/` or under `features/`.

## Commands

- `pnpm test --coverage`, `pnpm test:e2e` for route-level (see `package.json`).

## Focus

- **Do not** force full RSC trees into RTL if the repo tests **helpers** and **actions** separately (see `nextjs-testing.md`).

## Exemplar context

`claude.md/next/CLAUDE.md`
