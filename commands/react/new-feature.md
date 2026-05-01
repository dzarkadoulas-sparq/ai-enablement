Scaffold a new feature. Feature name: `$ARGUMENTS`.

Follow the project's feature-first convention. Use the existing `orders`
feature as the reference — copy structure and naming exactly.

1. **Create** `src/features/<feature>/`:
   - `index.ts` — exports only the public surface (pages, public hooks).
   - `api/<feature>.api.ts` — typed functions wrapping `src/api/client.ts`.
     Request/response types from Zod schemas co-located here or in `types.ts`.
   - `hooks/use<Feature>.ts` — TanStack Query hook (`useQuery` / `useMutation`)
     wrapping the api module. Exports `{ data, isPending, error, mutate }`
     with stable, documented query keys.
   - `components/<Feature>List.tsx` + `<Feature>Form.tsx` — named-function
     components. No default exports.
   - `pages/<Feature>Page.tsx` — route-level component. Uses the hook,
     renders the components.
   - `types.ts` — Zod schemas + `z.infer<>` types.
   - `__tests__/use<Feature>.test.tsx` — Vitest + RTL, MSW mocks the API.
   - `__tests__/<Feature>Page.test.tsx` — Vitest + RTL, queries by role /
     label, includes a `jest-axe` assertion.

2. **Wire the route** in `src/router.tsx`:
   - Lazy-load the page: `const <Feature>Page = lazy(() => import(...))`.
   - Add the route with `element={<Suspense ...><<Feature>Page /></Suspense>}`.
   - If the feature is auth-gated, wrap with the `RequireAuth` element.

3. **Add MSW handlers** in `src/mocks/handlers/<feature>.ts` for all
   endpoints the hook hits; register them in `src/mocks/handlers/index.ts`.

4. **Verify**
   - `pnpm lint --fix && pnpm typecheck`.
   - `pnpm test src/features/<feature>` — green.
   - `pnpm dev` + Playwright MCP navigation to the new route — no
     console errors.

5. **Summary** — every file created, the route path, and the next manual
   steps (add to navigation menu, add feature flag if behind a toggle,
   wire telemetry events).

Respect CLAUDE.md rules: no Context API for global state, no Redux, no
deep-path imports into another feature's internals, Tailwind only.
