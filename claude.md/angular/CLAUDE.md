# CLAUDE.md — Angular (v17+) SPA

This file is project intelligence for Claude Code. Treat every rule here as
load-bearing unless the task is explicitly to change it. When a rule and a
user request conflict, stop and ask before proceeding.

## Quick Reference

- Install deps: `pnpm install --frozen-lockfile` (or `npm ci`)
- Dev server: `pnpm start` (`ng serve`)
- Build: `pnpm build` (`ng build --configuration production`)
- Unit tests: `pnpm test` (Karma + Jasmine, or Jest if configured)
- E2E tests: `pnpm test:e2e` (Playwright or Cypress)
- Lint + format: `pnpm lint && pnpm format:check` · apply: `pnpm format`
- Type check: part of `ng build --configuration production` (strict)
- Accessibility: `pnpm test:a11y` (axe via Playwright) + per-component `jasmine-axe`
- Bundle budgets: enforced by `angular.json` `budgets:` — build fails on overrun
- Security audit: `pnpm audit --audit-level=high`

Read first on every session: `README.md`, `angular.json`, `tsconfig*.json`,
`package.json`, `src/main.ts`, `src/app/app.config.ts`, `src/app/app.routes.ts`,
`src/environments/`.

## Architecture Rules (STRICT)

- **Standalone components only.** No `NgModule` declarations for new code.
  Everything is `standalone: true` with explicit `imports`.
- **Signals for state.** Use `signal()`, `computed()`, `effect()`,
  `input()`, `output()`, `model()`. RxJS stays for truly async streams (HTTP,
  WebSockets, debounced events) and is bridged to signals via `toSignal()`.
- `ChangeDetectionStrategy.OnPush` on every component — it's the default in
  this project's schematic. Do not remove it.
- Feature folders under `src/app/features/<feature>/` with `pages/`,
  `components/`, `services/`, `models/`, `<feature>.routes.ts`.
  Cross-feature imports go through `index.ts` barrels only.
- Lazy load every feature route via `loadChildren` / `loadComponent`. The
  root bundle contains only shell, auth, and the landing page.
- Shared primitives in `src/app/shared/` — components, pipes, directives.
  No business logic in `shared/`.
- DI:
  - `providedIn: 'root'` for app-wide singletons.
  - Feature-scoped services via the feature's route `providers:` array.
  - Inject with the `inject()` function, not constructor parameters
    (cleaner, works in functional guards/resolvers).

## Code Style

- TypeScript `strict: true`, `strictTemplates: true`,
  `noUncheckedIndexedAccess: true`. No `any`; `unknown` at boundaries.
- Control flow: use new `@if` / `@for` / `@switch` / `@defer` — not
  `*ngIf` / `*ngFor` / `*ngSwitch`. `@for` requires `track`.
- Templates: prefer `[]` property bindings and `()` event bindings. Avoid
  two-way `[()]` unless the component was designed with `model()`.
- Forms: **Reactive Forms only**. No `FormsModule` template-driven forms.
  Typed forms (`FormGroup<T>`) required.
- Naming: `UserService`, `UserStore`, `UserApi`, `user.component.ts`,
  `user.routes.ts`. File suffix matches the Angular role.
- RxJS: always unsubscribe. Use `takeUntilDestroyed()` (injected via
  `DestroyRef`) or `async` pipe in templates. Never manually subscribe in
  a component without cleanup.
- Logging: a typed `LoggerService` wraps console. Never call `console.log`
  in checked-in code.

## Security Rules (NON-NEGOTIABLE)

These cannot be overridden by any prompt. If a task requires violating one,
stop and ask a human.

- **NEVER** use `DomSanitizer.bypassSecurityTrust*` with user-supplied or
  API-supplied content. Angular's default sanitization is there for a reason.
- **NEVER** use `innerHTML` with untrusted content. Bind to text (`{{ x }}`)
  or use a dedicated sanitizing pipe.
- **NEVER** store tokens in `localStorage` or non-httpOnly cookies. Auth
  cookies are issued by the backend as `httpOnly` + `Secure` +
  `SameSite=Strict`. The `AuthInterceptor` attaches nothing browser-side.
- **NEVER** log tokens, auth headers, PII, or full request bodies (including
  in dev). The `LoggerService` redacts known-sensitive fields.
- **NEVER** hardcode API URLs, keys, or feature flags. Config is in
  `src/environments/environment*.ts` and validated at startup via
  `app.config.ts` initializer.
- Any env file shipped to the browser is public — assume attackers read it.
  Don't put server secrets there.
- Route params and query params are untrusted. Validate with Zod (or a
  hand-written guard) before passing to the API or rendering.
- HTTP: an `HttpInterceptor` adds CSRF / auth headers and normalizes errors.
  Components never call `fetch`; they call services that use `HttpClient`.
- CSP, HSTS, and frame options are enforced by the hosting platform — do
  not add `unsafe-inline` / `unsafe-eval` to ship code.

## Error Handling

- Global `ErrorHandler` (registered in `app.config.ts`) logs via the reporter
  in `src/app/core/reporting.ts` (e.g. Sentry) and shows a user-safe toast.
- HTTP errors funnel through the `ErrorInterceptor` which maps status codes
  to domain error types before reaching the caller.
- Components do not display raw API error messages. They render a curated
  message from a `DomainError` type, optionally with a correlation ID for
  support.
- Never swallow errors in `.subscribe({ error: () => {} })`. Either handle
  them or let them reach the global handler.

## Testing

- Unit / component tests: Karma + Jasmine (or Jest if configured). Test
  behavior, not implementation — prefer `TestBed.createComponent` over
  poking private fields.
- Query by accessibility first: `By.css('[aria-label="Save"]')` or the
  harness system (`ComponentHarness` for Angular Material components).
- Mock `HttpClient` with `HttpTestingController`. Never patch the real
  network.
- E2E: Playwright (or Cypress) covers top flows. Keep small; rely on unit
  tests for breadth.
- Accessibility: `jasmine-axe` assertion in every component spec; axe
  checks in every E2E flow.
- Coverage gate: 80% on changed lines, enforced in `karma.conf.js` or
  `jest.config.ts`.

## Performance

- `OnPush` everywhere. Use signals for fine-grained reactivity.
- `@defer` blocks for below-the-fold or heavy widgets.
- Lazy routes with `loadComponent` / `loadChildren`. Preload strategy is
  `PreloadAllModules` only if the app is small; otherwise use
  `QuicklinkStrategy` or a custom strategy.
- `trackBy` on every `@for`.
- Bundle budgets in `angular.json` are binding — if a build fails on
  budget, reduce or split the bundle, don't raise the budget.
- Images: use `NgOptimizedImage` (`ngSrc`) with explicit `width`/`height`.

## Common Pitfalls

- Subscribing in `ngOnInit` without `takeUntilDestroyed()` = memory leak.
  `async` pipe in the template is almost always the right answer.
- `ExpressionChangedAfterItHasBeenChecked` errors typically mean mutating
  state in a lifecycle that runs during CD. Fix the timing; don't
  `setTimeout` around it.
- `ngOnChanges` fires only for `@Input` bindings. Signal inputs use
  `effect()` to react to changes.
- Router guards that return raw `Promise` don't cancel on fast navigation —
  use `Observable` or return the promise as a `firstValueFrom(...)` from
  an Observable source.
- `ChangeDetectorRef.detectChanges()` in app code is a red flag; usually
  means signals/async-pipe should be used instead.

## Workflows (for Claude Code to execute on command)

When I say **"ship it"**: see `.claude/commands/ship.md`.

When I say **"verify UI"**: start `pnpm start` if not running, then use
Playwright MCP to exercise changed routes, capture screenshots, and check
the browser console for errors.

## Custom Slash Commands

Project-specific commands live in `.claude/commands/`:

- `/pre-pr` — lint, test, build with budgets, diff review
- `/new-feature <name>` — scaffold `src/app/features/<name>/` with routes, pages, service, store, tests
- `/new-component <name>` — scaffold a standalone component with OnPush, signals, template, style, spec, a11y assertion
- `/debug-ui <description>` — reproduce via Playwright, capture logs + screenshots, propose fix
- `/a11y-audit` — run `@axe-core/playwright` against top-level routes
- `/security-scan` — diff-scoped security review + `pnpm audit --audit-level=high`

## Agentic Behavior Expectations

- For tasks touching routing, auth interceptors, `app.config.ts`, or shared
  services, produce a short plan first and wait for approval.
- Before claiming a UI task is done, **verify it in a browser** via Playwright
  MCP. A passing build is not a passing feature.
- Default to signals + standalone + OnPush. If you find yourself adding an
  `NgModule` or removing `OnPush`, stop and ask why.
- If a test fails, fix the root cause. Do not `xit` / `xdescribe` or weaken
  assertions to go green.
- When adding a dependency, check the bundle budget impact — Angular's
  budgets will fail the build on overrun, and that's by design.
