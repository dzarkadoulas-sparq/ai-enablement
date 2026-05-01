Scaffold a new feature. Feature name: `$ARGUMENTS`.

Follow the project's standalone + signals + OnPush convention. Use the
existing `orders` feature as the reference — copy patterns exactly.

1. **Create** `src/app/features/<feature>/`:
   - `<feature>.routes.ts` — feature-scoped routes, lazy-loaded from the
     root router via `loadChildren`.
   - `pages/<feature>-list/<feature>-list.component.ts` — standalone
     page component with `ChangeDetectionStrategy.OnPush`.
     Template and styles inline or in sibling `.html` / `.scss`.
   - `pages/<feature>-detail/<feature>-detail.component.ts` — same pattern.
   - `components/<feature>-form/<feature>-form.component.ts` —
     reusable standalone component. Reactive Forms only.
   - `services/<feature>.api.ts` — typed `HttpClient` wrapper.
     `providedIn: 'root'` only if app-wide; otherwise scope to the
     feature's route `providers:`.
   - `services/<feature>.store.ts` — signal-based store (signals +
     computed + update methods). No NgRx unless the app already uses it.
   - `models/<feature>.model.ts` — types + Zod schemas (or hand-written
     interfaces if Zod isn't used).

2. **Wire the lazy route** in `src/app/app.routes.ts`:

   ```
   { path: '<feature>s',
     loadChildren: () => import('./features/<feature>/<feature>.routes')
       .then(m => m.<FEATURE>_ROUTES) }
   ```

3. **Create specs**
   - `<feature>-list.component.spec.ts` — TestBed, renders the component,
     queries by accessible name / role harness, includes a
     `jasmine-axe` assertion.
   - `<feature>-detail.component.spec.ts` — same.
   - `<feature>.api.spec.ts` — `HttpTestingController` for all endpoints.

4. **Auth / guards** — if the feature requires auth, wire the existing
   functional `authGuard` in the feature's route definitions.

5. **Verify**
   - `pnpm lint --fix`.
   - `pnpm test -- --watch=false --browsers=ChromeHeadless
--include='src/app/features/<feature>/**'`.
   - `pnpm build` (must stay within budgets).

6. **Summary** — files created, route path, bundle-size impact (from
   `ng build` output), a11y result, next manual steps (add to main nav,
   add i18n strings, wire analytics event).

Respect CLAUDE.md rules: standalone only, OnPush, signals for state,
reactive forms, `takeUntilDestroyed()` for subscriptions, no `NgModule`.
