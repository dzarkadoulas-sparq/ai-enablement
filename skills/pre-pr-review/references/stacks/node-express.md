# Pre-PR gate: Node.js + Express (TypeScript API)

## Baseline branch

`origin/develop` or repo default. `git diff origin/develop...HEAD`.

## Gate order

1. **Lint + format** — `pnpm lint --fix`, `pnpm format`, then `pnpm lint` and `pnpm format:check`.
2. **Typecheck** — `pnpm typecheck`. No new `// @ts-expect-error` without a same-line reason.
3. **Unit tests** — `pnpm test --coverage`.
4. **Integration tests** — `pnpm test:integration`. If Docker isn’t running, **stop and report** (do not skip).
5. **Coverage gate** — e.g. 80% on changed lines if that’s the project bar; use project tooling.
6. **Build** — `pnpm build`.
7. **Security** — `pnpm audit --audit-level=high`; flag HIGH/CRIT on **introduced** deps.
8. **Diff hygiene** — see **Diff hygiene (Node API)** below.
9. **Summary**. **Do not push or open PR** unless asked.

## Diff hygiene (Node API)

- `console.log` / `console.debug` / `debugger` in server code.
- `.skip` / `.todo` without ticket.
- Commented-out code blocks.
- Hardcoded URLs, tokens, passwords.
- Routes without auth that aren’t listed as public in the project’s allowlist.
- Request handlers without schema validation (e.g. Zod) when that’s the standard.
- `prisma.$queryRawUnsafe` (or ORM equivalent) with unsafe string interpolation.
- `any` / `as unknown as` without justification.

## Exemplar

`commands/node-express/pre-pr.md`
