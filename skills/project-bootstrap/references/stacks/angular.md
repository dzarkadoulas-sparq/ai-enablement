# Stack: Angular (modern, standalone)

## Detection

- `@angular/core` in `package.json`.
- `angular.json` present.

## Read first

`README.md`, `angular.json`, `tsconfig*.json`, `package.json`, `src/main.ts`, `src/app/app.config.ts`, `src/app/app.routes.ts`, `src/environments/` (or `environment.*.ts`).

## Trace one flow

`routes` → lazy `loadComponent` or feature route → page component → service (HTTP) → API. Note **signals** vs **RxJS** usage as seen in code.

## `CLAUDE.md` sections to generate

1. **Quick Reference** — `ng serve` / pnpm scripts, `ng build`, test and e2e from `angular.json` and `package.json`.
2. **Read first** — `main.ts`, `app.config`, route table, environment files.
3. **Architecture** — **standalone** components, `ChangeDetectionStrategy`, feature folders, lazy loading, `inject()` vs constructors, where HTTP and interceptors live.
4. **State** — signals vs store (if any); RxJS for async as found in code.
5. **Style** — file naming, `inline` vs template files, Tailwind/SCSS as per repo.
6. **Security** — environment-specific API URLs, `DomSanitizer` usage, CSRF or cookie strategy if visible.
7. **Testing** — Jasmine/Karma vs Jest; component test location.
8. **Pitfalls** — bundle budgets from `angular.json`, deep subscription chains without `takeUntilDestroyed` / `async` pipe.
9. Conflict / stop-and-ask line.

## Stack red flags

- New `NgModule`-heavy code in a project otherwise standalone (debt).
- `any` in templates or public APIs without policy.
- Missing lazy routes for large apps (single bundle bloat) — call out if `angular.json` budgets are tight.

## Exemplar (ai-enablement)

`templates/angular/CLAUDE.md`
