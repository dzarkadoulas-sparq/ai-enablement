# Angular (TestBed) testing

## Conventions

- **File naming:** `*.spec.ts` next to component/service or in `__tests__` if the repo does that; follow **Angular style guide** + team `angular.json` test config.
- **TestBed** — `TestBed.configureTestingModule({ imports: [...] })` with **standalone** `imports` matching production; `provideHttpClient` / `HttpClientTestingModule` for HTTP; **override** components only when necessary.
- **Signals** — set inputs with `componentRef.setInput` (Angular 17+) or fixture’s API as per version; use **fake async** or **AsyncTestCompleter** for tick-based effects when needed.

## Component tests

- **DOM:** `fixture.debugElement.query(By.css('...'))` or native element; use **Accessibility**-friendly queries where possible (`ByRole` via Testing Library if project adds `@testing-library/angular`).
- **Change detection** — `fixture.detectChanges()` after state changes; **OnPush** components may need explicit `detectChanges` or `markForCheck` patterns from existing specs.

## HTTP

- `HttpClientTestingModule` + `HttpTestingController` to **expect** one request, flush response, assert **unmatched** is empty in `afterEach`.

## Router

- `RouterTestingModule` with routes; navigate in test; **spy** on `router.navigate` when asserting side effects.

## Services

- Pure services: test without TestBed; DI-heavy: `TestBed.runInjectionContext` or inject in `beforeEach` per project.

## E2E

- Playwright/Cypress is separate from this skill’s unit focus; e2e lives in project’s `e2e/` scripts.
