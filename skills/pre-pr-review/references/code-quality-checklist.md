# Code quality (diff review)

Apply to `git diff <base>...HEAD` (or the PR diff). Prefer **concrete** comments with file paths.

## Structure and clarity

- **New modules** — single responsibility; avoid “god” additions to already huge files when the project splits by feature.
- **DRY** — obvious copy-paste with the same bug risk: suggest one abstraction or shared helper.
- **Dead code** — new unused exports, imports, or unreachable branches; **commented-out** blocks: remove or ticket.

## Error handling

- **Swallowed errors** — empty `catch`, `except: pass`, or log-only without rethrow/translation where callers need signal.
- **User-facing errors** — stable, safe messages; map internal errors to client-safe text (per stack patterns).

## Types and contracts

- **TypeScript** — new `any` or `as unknown as` without reason; new `@ts-expect-error` / `@ts-ignore` without a comment.
- **Python** — new bare `# type: ignore` without reason; new `Any` in public API without justification.
- **API contracts** — request/response DTOs updated together with OpenAPI/consumer when applicable.

## Performance (only if the diff is hot-path)

- **N+1** queries, unbounded `list`/`in` for large ID sets, blocking I/O in `async` handlers.
- **Large** new dependencies: worth calling out in report (P2 unless security-heavy).

## Tests

- **New logic** in critical areas (auth, money, permissions, migrations) without tests — **P0/P1** depending on team bar.
- **`.skip` / `xit` / `@Ignore`** on tests touched by the branch — need ticket or removal.

## Naming and comments

- Accurate **names** for new symbols; **misleading** names on changed behavior = P2.
- **TODO/FIXME** in new code — need ticket or owner if they imply unfinished work pre-merge (team-dependent).

## Unrelated changes

- IDE noise, reformat of unrelated files, or mixed concerns in one PR — P1/P2: suggest split unless trivial.
