Scaffold a new standalone component. Name: `$ARGUMENTS`.

1. **Decide placement**
   - Feature-specific → `src/app/features/<feature>/components/<name>/`.
   - Shared primitive → `src/app/shared/components/<name>/`.
     Ask if ambiguous.

2. **Create the component** — use the project's schematic defaults:

   ```
   ng g component <path>/<name> \
     --standalone --change-detection=OnPush \
     --style=scss --skip-tests=false
   ```

   If the project uses Jest instead of Karma, adjust accordingly.

3. **Required structure**
   - `<name>.component.ts` — `standalone: true`, `changeDetection: OnPush`,
     explicit `imports` array listing only what the template uses.
   - Signal inputs (`input()`, `input.required()`) and outputs (`output()`)
     — not legacy `@Input()` / `@Output()`.
   - Template uses new control flow (`@if`, `@for` with `track`, `@switch`).
   - Styles: SCSS with component-scoped selectors. No global overrides.

4. **Accessibility baseline (non-negotiable)**
   - Correct semantic element (`<button>`, `<a>`, `<nav>`, etc.).
   - Icon-only buttons have `[attr.aria-label]`.
   - Focus visible; do not remove `outline` without replacement.
   - Interactive state (`disabled`, `aria-expanded`, `aria-pressed`)
     is driven by inputs or internal state, not duplicated.

5. **Spec**
   - TestBed with the component imported standalone.
   - Query by role / accessible name (use Angular Material harnesses if
     wrapping a Material component).
   - `jasmine-axe` assertion: `expect(await axe(fixture.nativeElement))
.toHaveNoViolations()`.

6. **Visual verification** — with `pnpm start` running, navigate via
   Playwright MCP to a route / Storybook page that renders the component.
   Screenshot; confirm no console errors.

7. **Verify**
   - `pnpm lint --fix`.
   - `pnpm test -- --watch=false --browsers=ChromeHeadless
--include='<component-path>/**'`.

8. **Summary** — files created, inputs / outputs, a11y check result,
   screenshot, and any remaining polish (reduced-motion, RTL handling,
   high-contrast) for the PR checklist.

Do not add a new external dependency without approval.
