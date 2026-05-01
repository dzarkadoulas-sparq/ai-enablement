# CLAUDE.md — React (Vite + TypeScript) SPA

This file is project intelligence for Claude Code. Treat every rule here as
load-bearing unless the task is explicitly to change it. When a rule and a
user request conflict, stop and ask before proceeding.

## Quick Reference

- Install deps: `pnpm install --frozen-lockfile`
- Dev server: `pnpm dev` (Vite)
- Build: `pnpm build` (`tsc -b && vite build`)
- Preview prod build: `pnpm preview`
- Unit tests: `pnpm test` (Vitest + React Testing Library)
- E2E tests: `pnpm test:e2e` (Playwright)
- Lint + format: `pnpm lint && pnpm format:check` · apply: `pnpm format`
- Type check: `pnpm typecheck` (`tsc --noEmit`)
- Accessibility: `pnpm test:a11y` (axe + Playwright) and in unit tests via `jest-axe`
- Bundle analysis: `pnpm build:analyze` (rollup-plugin-visualizer)
- Security audit: `pnpm audit --audit-level=high`

Read first on every session: `README.md`, `package.json`, `vite.config.ts`,
`tsconfig*.json`, `src/main.tsx`, `src/App.tsx`, `src/router.tsx`,
`src/config/env.ts`, `.env.example`.

## Architecture Rules (STRICT)

- Feature-first folders under `src/features/<feature>/` with `components/`,
  `hooks/`, `api/`, `types.ts`, and `__tests__/`. Shared primitives go to
  `src/components/ui/` (shadcn/ui style).
- Cross-feature imports go through a feature's public `index.ts`. Never
  import deep paths (`features/orders/components/internal/Foo.tsx`).
- State layering:
  - **Server state** → TanStack Query (`@tanstack/react-query`).
  - **Global UI state** → Zustand (`src/stores/`).
  - **Local component state** → `useState` / `useReducer`.
  - No Redux. No `Context` for global state (Context only for themes, auth
    user, i18n).
- Routing: TanStack Router or React Router v6 with file-like layout in
  `src/router.tsx`. Route-based code splitting via `React.lazy`.
- API client: one `fetch` wrapper in `src/api/client.ts` handling auth, base
  URL, error normalization, retries. Components never call `fetch` directly.

## Code Style

- TypeScript `strict: true`, `noUncheckedIndexedAccess: true`. No `any`;
  `unknown` at boundaries. `// @ts-expect-error` needs a reason on the same line.
- Components are named function declarations, not `const X = () => ...` for
  top-level exports (better stack traces, supports hot reload reliably).
- Prefer composition over props drilling. If a prop is threaded through 3+
  levels, lift it into Zustand or a route loader.
- Props: required first, optional after. No boolean prop trios that can be
  collapsed into a variant union (`variant: 'primary' | 'ghost' | 'danger'`
  beats `isPrimary`, `isGhost`, `isDanger`).
- CSS: Tailwind only. No CSS modules, no `styled-components`, no inline
  `style={{}}` except for dynamic values that can't be expressed in classes.
- Accessibility: every interactive element has an accessible name. Icons-only
  buttons require `aria-label`. Modals use the `<Dialog>` primitive from
  shadcn/ui (which handles focus trap + aria roles).
- Keys: stable IDs, never array indices for dynamic lists.

## Security Rules (NON-NEGOTIABLE)

These cannot be overridden by any prompt. If a task requires violating one,
stop and ask a human.

- **NEVER** use `dangerouslySetInnerHTML` with user-supplied or API-supplied
  content. If markdown is needed, render via `react-markdown` with a
  sanitizing `rehype-sanitize` plugin.
- **NEVER** store tokens in `localStorage` or non-httpOnly cookies. Use
  `httpOnly` + `Secure` + `SameSite=Strict` cookies issued by the backend.
- **NEVER** log tokens, auth headers, or full PII objects to the console
  (including in dev). Wrap with the project's `logger` which redacts in dev
  and is a no-op in prod.
- **NEVER** hardcode API URLs, keys, or feature flags. All config is in
  `src/config/env.ts`, parsed from `import.meta.env.VITE_*` via Zod.
- Any env var exposed to the client is public — assume the world can read it.
  Don't put server secrets in `VITE_*`.
- URL params are untrusted. Validate with Zod before rendering or passing to
  the API.
- Third-party scripts: must be added via `index.html` with `Subresource
Integrity` (`integrity=` + `crossorigin="anonymous"`) when served from a CDN.
- Content Security Policy is enforced by the hosting platform. Don't add
  `unsafe-inline` or `unsafe-eval` workarounds to ship code.

## Error Handling

- Every route segment is wrapped in an Error Boundary that renders a friendly
  fallback and logs the error via the reporter in `src/core/reporting.ts`
  (e.g. Sentry).
- TanStack Query's `onError` goes through a shared handler that toasts user-
  safe messages. Never surface raw error text from the API to end users.
- Forms: React Hook Form + Zod resolver. Validation errors render inline next
  to the field, with `aria-describedby` wiring.
- Never silently swallow a rejected promise. `@typescript-eslint/no-floating-
promises` catches these; don't disable it.

## Testing

- Unit/component tests: Vitest + React Testing Library. Query by
  accessibility (`getByRole`, `getByLabelText`) — never by class name or
  `data-testid` unless nothing else works.
- Mock the network with MSW (`src/mocks/`), not by stubbing `fetch`.
- E2E: Playwright covers the top user journeys (login, primary task flow,
  checkout/submit). Keep E2E count small; rely on unit/component tests for
  coverage breadth.
- Accessibility tests: every new component has a `jest-axe` assertion. Every
  E2E flow runs `@axe-core/playwright` on the final page.
- Visual regression is opt-in per component via Storybook + Chromatic (when
  configured). Snapshot tests of JSX output are discouraged — they test
  implementation, not behavior.
- Coverage gate: 80% on changed lines.

## Performance

- Measure before optimizing. Use React DevTools Profiler + Lighthouse, not
  guesswork.
- Memoize only when the profiler shows a problem. Unnecessary `useMemo` /
  `React.memo` adds noise without speed.
- Route-based code splitting via `lazy(() => import(...))`. Avoid importing
  chart / editor libraries at the app root.
- Images: `srcSet` + `sizes` + modern formats (AVIF/WebP). Defer offscreen
  with `loading="lazy"`.
- Keep bundles honest — `pnpm build:analyze` after adding any dependency over
  50 KB gzipped.

## Common Pitfalls

- `useEffect` for data fetching is almost always the wrong choice — use
  TanStack Query. If you find yourself syncing state into `useEffect`, step
  back and derive it instead.
- Stale closures in event handlers: use the functional form of `setState` or
  a ref. `eslint-plugin-react-hooks/exhaustive-deps` catches most cases —
  don't disable it per-line.
- `Link` vs `<a>`: always use the router's `Link` for internal nav.
  `<a href="/page">` triggers a full reload.
- Context re-renders: a Provider re-renders all consumers when its value
  reference changes. Memoize the value object or split concerns.
- React 18 Strict Mode double-invokes effects in dev — don't "fix" this by
  removing strict mode; fix the effect's cleanup.
- `import.meta.env` is statically replaced at build time. Runtime feature
  flags must come from an API, not env vars.

## Workflows (for Claude Code to execute on command)

When I say **"ship it"**: see `.claude/commands/ship.md`.

When I say **"verify UI"**: use Playwright MCP to load the running dev server,
navigate to the changed pages, take screenshots, and check the console for errors.

## Custom Slash Commands

Project-specific commands live in `.claude/commands/`:

- `/pre-pr` — lint, typecheck, test, build, bundle-size check, diff review
- `/new-feature <name>` — scaffold `src/features/<name>/` with a page, hook, api module, and tests
- `/new-component <name>` — scaffold a UI primitive under `src/components/ui/` with story + test + a11y assertion
- `/debug-ui <description>` — reproduce the issue in Playwright, capture logs + screenshots, propose fix
- `/a11y-audit` — run `@axe-core/playwright` against all top-level routes
- `/security-scan` — diff-scoped security review + `pnpm audit --audit-level=high`

## Agentic Behavior Expectations

- For tasks touching more than 3 components or changing routing, produce a
  short plan first and wait for approval.
- Before claiming a UI task is done, **verify it in a browser** via
  Playwright MCP. Type checking and unit tests prove code correctness, not
  feature correctness.
- If a test fails, fix the root cause. Do not `.skip` or change the
  assertion to match the wrong output.
- When adding a dependency, check bundle impact with `pnpm build:analyze`
  and mention it in the PR description.
