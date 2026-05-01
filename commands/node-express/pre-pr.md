Run the full pre-PR gate for this Node.js / Express (TypeScript) service.

Stop at the first failing step and report; do not continue past a failure.

1. **Lint + format**
   - `pnpm lint --fix` (ESLint).
   - `pnpm format` (Prettier).
   - Re-run `pnpm lint` and `pnpm format:check` — any remaining = fail.

2. **Type check**
   - `pnpm typecheck` (`tsc --noEmit`).
   - Zero errors on changed files. No new `// @ts-expect-error` without a
     reason comment on the same line.

3. **Unit tests**
   - `pnpm test --coverage`.

4. **Integration tests**
   - `pnpm test:integration`.
   - If Docker isn't running, say so and stop — don't skip.

5. **Coverage gate**
   - 80% on changed lines. Fail if below.

6. **Build**
   - `pnpm build` — must succeed on its own (catches declaration-only
     errors that `typecheck` misses when files are emit-only).

7. **Security**
   - `pnpm audit --audit-level=high`.
   - Flag HIGH/CRITICAL from any dependency introduced in this branch.

8. **Diff hygiene** — `git diff origin/develop...HEAD` and check for:
   - `console.log` / `console.debug` / `debugger` in production code.
   - `.skip` / `.todo` without linked ticket.
   - Commented-out code blocks.
   - Hardcoded URLs, IPs, tokens, passwords.
   - Routes missing auth middleware that aren't in `src/core/public-routes.ts`.
   - Request handlers without a Zod `validate(schema)` middleware.
   - `prisma.$queryRawUnsafe` with string interpolation.
   - `any` or `as unknown as X` added without justification.

9. **Summary**
   - Commits on the branch + one-line description.
   - Changed-file count grouped by feature.
   - Gate status (PASS / FAIL + reason).
   - 2–3 bullet draft of the PR description.

Do not push, do not open a PR. This command only verifies the branch.
