# Impact analysis: React (Vite + TypeScript)

## Tracing

- **Imports** — `from '...'`, `from "@/"` (resolve **paths** from `tsconfig`, `vite.config`).
- **Barrel** files — `src/features/x/index.ts` re-exports: **grep** **symbol** and **path** `features/x`.
- **Props** — if **target** is a **component**, find **parent** **JSX** `<Target` and **wrappers**; **react** **lazy** `import()` **strings**.
- **Context** / **store** — `createContext`, **Zustand** **selectors** **string** **keys** if any.

## Dynamic

- **Route** **params** in **router** **config** (TanStack Router, React Router `path`).

## Tests

- **RTL** **render** **wrappers**; **MSW** **handlers** **by** **URL** if the **component** **triggers** **fetch**.

## Tools

- `rg` / **IDE** **Find refs**; `pnpm exec tsc --noEmit` may **surface** **broken** **refs** after **rename** (dry run planning).
