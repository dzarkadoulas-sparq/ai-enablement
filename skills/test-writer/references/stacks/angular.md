# Test writer: Angular (v17+)

## Language references

- `references/angular-testing.md` — TestBed, HttpClientTestingModule, router.
- `references/edge-case-catalog.md`, `references/anti-patterns.md`

## Detect / locate tests

- `*.spec.ts` with **Karma/Jasmine** or **Jest** as in `angular.json` / `package.json`.
- **Signals** / `input()` / `output()` — use Angular 17+ testing APIs from existing specs.

## Commands

- `pnpm test` with `--code-coverage` if that’s the standard (see `angular` pre-pr / `package.json`).
- E2E `pnpm test:e2e` for route changes—**separate** from unit test addition.

## Focus

- **OnPush** and `detectChanges` patterns; **HttpTestingController** request matching.
- Avoid **NgModule**-heavy tests if the app is **standalone**-only for new code.

## Exemplar context

`claude.md/angular/CLAUDE.md`
