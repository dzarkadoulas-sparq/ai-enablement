# React + TypeScript (Vitest + Testing Library)

## Conventions (prefer project’s existing tests)

- **File placement:** colocate `*.test.ts` / `*.test.tsx` or `__tests__/` as in the repo; mirror feature folders.
- **Query priority** — [Testing Library](https://testing-library.com/docs/queries/about#priority): accessible roles, labels, then test ids (last).
- **User events** — `@testing-library/user-event` over `fireEvent` when the project has it; `await user.click(...)`.

## Patterns

- **Wrapper:** custom `render()` that includes **Router**, **QueryClientProvider**, **theme**, **i18n** as the project’s test utils do.
- **Data:** **MSW** (`msw` handlers) for HTTP when components fetch; avoid mocking `fetch` with ad-hoc `jest.fn()` unless the repo already does.
- **Router:** `MemoryRouter` or the project’s test router helper; provide route params the component reads.
- **Swr/tanstack:** pre-seed **cache** or wrap with fresh **QueryClient** per test.

## Async

- Use `await findByRole` / `findByText` for async UI; `waitFor` with explicit assertion, avoid arbitrary `setTimeout` sleeps.

## Accessibility (if project uses `jest-axe` / `vitest-axe`)

- Run on meaningful DOM state after load; not on empty shell.

## What to assert

- **Visible** outcome: text, disabled state, ARIA, redirect (mock navigate if needed), **not** internal `useState` value unless no other signal.

## Next.js

For App Router, see `references/nextjs-testing.md` for RSC and Server Action boundaries.
