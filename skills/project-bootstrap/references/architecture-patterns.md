# Architecture patterns (cross-stack)

Use this file **after** loading the stack-specific reference (`references/stacks/<id>.md`). It complements stack rules with patterns that recur across many codebases.

## Monorepos

- Look for **workspace** config: `pnpm-workspace.yaml`, `lerna.json`, Nx / Turborepo (`turbo.json`), Gradle composite builds, **`.sln` with multiple projects**.
- Output: a **top section** in `CLAUDE.md` listing each package/app by name, path, tech, and its own "Quick reference" commands if they differ.
- Trace **one flow per major deployable** when time allows.

## Layering (generic)

- Prefer **feature-first** or **package-by-feature** when present; call out **forbidden** shortcuts (e.g. UI → DB skipping application layer).
- Note **where DI / composition root** lives (Spring `@Configuration`, `Program.cs`, FastAPI `include_router`, Express `app.use`).

## Request / entry tracing

- **Web APIs:** find the HTTP entry (router, controller, minimal API), then follow to service/use-case, then persistence.
- **SPAs:** entry (`main.tsx` / `main.ts` / `bootstrapApplication`), top route config, first screen’s data source (loader, query client, service).
- **Next App Router:** `app/layout.tsx`, `middleware.ts`, one representative `app/.../page.tsx` and any Server Action or `route.ts` used for mutations.

## Tribal knowledge (search)

- Ripgrep: `TODO|FIXME|HACK|XXX|@deprecated` in `src/`, `app/`, `lib/` (scope to source, not `node_modules` / `dist` / `build`).
- Highlight items that imply **security**, **data loss**, or **wrong behavior** if ignored.

## Red flags (generic)

- **God files** — single file with huge line count and many unrelated exports (threshold depends on team; call out if central to domain).
- **No tests** next to critical auth, payment, or migration paths.
- **Copy-paste** patterns across features (duplicated validation, three similar API clients) — suggest consolidation in `CLAUDE.md` "Pitfalls" or "Refactor notes".
- **Outdated** major frameworks (compare manifest versions to current LTS; note as risk, not as automatic upgrade task).
- **Secrets** — API keys, connection strings, `password=` in non-vault files. **Never** copy secrets into `CLAUDE.md`; say "remove from repo" and point to file pattern.

## Onboarding doc

Keep it short: **prereqs** (Node/Java/.NET/Python version), **3 commands** (install, run, test), **where config lives**, **where to add a feature** (one paragraph), link to `CLAUDE.md`.
