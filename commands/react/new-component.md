Scaffold a new UI primitive under `src/components/ui/`. Name: `$ARGUMENTS`.

These are shared, feature-agnostic components. Follow the shadcn/ui style
(composable, headless-ish, Tailwind-first, `asChild` where sensible).

1. **Create** `src/components/ui/<name>/`:
   - `<Name>.tsx` — named-function export. Forward `ref`. Accept a
     `className` and compose via `cn(...)` (from `src/lib/utils.ts`).
     Use a `variant` / `size` props union and implement with
     `class-variance-authority` (`cva`) if variants exist.
   - `<Name>.stories.tsx` — Storybook stories covering default +
     each variant + any interaction states (hover, disabled, focus, error).
   - `<Name>.test.tsx` — Vitest + React Testing Library. Query by role.
     Include a `jest-axe` assertion: `expect(await axe(container)).toHaveNoViolations()`.
   - `index.ts` — re-exports the component and its types.

2. **Accessibility baseline (non-negotiable)**
   - Correct semantic element (`<button>` for actions, `<a>` for nav, etc.).
   - All icon-only buttons: `aria-label`.
   - Interactive elements reachable by keyboard.
   - `aria-disabled` instead of `disabled` when the element must remain
     focusable for announcement.
   - Focus ring preserved — do not use `outline: none` without replacement.

3. **Visual-only verification** — with `pnpm dev` running, use Playwright
   MCP to:
   - Navigate to the Storybook URL for the new component.
   - Screenshot each story.
   - Run axe via `@axe-core/playwright` — zero violations.
   - Report findings.

4. **Verify**
   - `pnpm lint --fix && pnpm typecheck`.
   - `pnpm test src/components/ui/<name>`.

5. **Summary** — files created, variants implemented, a11y checklist
   results, Storybook URL, and any remaining polish (focus-visible,
   reduced-motion support, RTL handling) for the PR checklist.

Do not add a new external dependency without approval. Prefer composing
existing primitives (`Radix`, `cva`, `clsx`) already in the project.
