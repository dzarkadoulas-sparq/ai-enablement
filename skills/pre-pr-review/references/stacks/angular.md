# Pre-PR gate: Angular (v17+)

## Baseline branch

`origin/develop` or repo default. `git diff origin/develop...HEAD`.

## Gate order

1. **Lint + format** — `pnpm lint --fix`, `pnpm format`, then `pnpm lint && pnpm format:check`.
2. **Unit tests** — `pnpm test -- --watch=false --browsers=ChromeHeadless --code-coverage` (or project’s exact test script).
3. **E2E (conditional)** — `pnpm test:e2e` when diff touches routes/guards; follow project policy.
4. **Accessibility** — `pnpm test:a11y`. New standalone component without `jasmine-axe` (or equivalent) = fail.
5. **Production build** — `pnpm build` (`ng build` production). **Do not** raise `angular.json` bundle budgets to green the build.
6. **Security** — `pnpm audit --audit-level=high`.
7. **Diff hygiene** — see **Diff hygiene (Angular)** below.
8. **Optional** — Playwright MCP for UI changes. See `commands/angular/pre-pr.md` in ai-enablement.
9. **Summary**. **Do not push or open PR** unless asked.

## Diff hygiene (Angular)

- `console.log` / `debugger`.
- `xit` / `xdescribe` / `.skip` without ticket.
- Commented-out templates or code.
- `DomSanitizer.bypassSecurityTrust*` / `[innerHTML]` without safety policy.
- `OnPush` removed, new `NgModule` for new code, template-driven `FormsModule` when project mandates reactive, new `*ngIf`/`*ngFor` where control flow should be `@if`/`@for`, `.subscribe` without `takeUntilDestroyed` or async pipe.
- Hardcoded URLs — use `environments/`.

## Exemplar

`commands/angular/pre-pr.md`
