# Pre-PR gate: Next.js (App Router)

## Baseline branch

`origin/develop` or repo default. `git diff origin/develop...HEAD`.

## Gate order

1. **Lint + format** — `pnpm lint --fix`, `pnpm format`, then `pnpm lint && pnpm format:check`.
2. **Typecheck** — `pnpm typecheck`. No new `// @ts-expect-error` without a same-line reason.
3. **Unit + component tests** — `pnpm test --coverage`.
4. **Build** — `pnpm build` (catches RSC boundary and env issues).
5. **Bundle budget** — `ANALYZE=true pnpm build` (or project analyzer); compare to baseline, >10% without justification = fail.
6. **Security** — `pnpm audit --audit-level=high`.
7. **Diff hygiene** — see **Diff hygiene (Next)** below.
8. **Optional** — Playwright MCP for changed UI routes. See `commands/next/pre-pr.md` in ai-enablement.
9. **Summary** — same as other stacks. **Do not push or open PR** unless asked.

## Diff hygiene (Next)

- `console.log` / `debugger`.
- `.skip` / `.todo` without ticket.
- Unnecessary `"use client"` at large tree roots.
- `dangerouslySetInnerHTML` without sanitizer.
- `fetch` in Server Components without explicit `cache` / `revalidate` / `tags` where the project pattern requires it.
- `process.env` outside the project’s validated `env.ts` (or equivalent).
- Server Actions / Route Handlers missing auth, validation, or public justification.
- `NEXT_PUBLIC_*` for server-only secrets; overly broad `revalidatePath`.

## Exemplar

`commands/next/pre-pr.md`
