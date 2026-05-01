# Refactor guide: Angular (v17+)

## Verify

- `pnpm lint`, `ng build` (or the project’s production build script), `pnpm test` with coverage as configured.

## Refactor-specific notes

- **Standalone** components: update the `imports` array on every component that referenced the old selector or a removed NgModule-based symbol.
- **Signals** — when extracting `computed` or `effect`, keep **injection** context and **OnPush** behavior correct for the new structure.
- **Routes** — update every `loadComponent` / `loadChildren` string when you move or rename feature entry points.

## Pitfalls

- Barrel `index.ts` re-exports: a renamed export can affect **tree-shaking** and **lazy** chunks—compare bundle output for large refactors.
