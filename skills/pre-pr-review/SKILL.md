---
name: pre-pr-review
description: >-
  Runs the full pre-merge quality gate: lint, format, typecheck, tests, build where
  applicable, security/dependency checks, and diff review. Produces a prioritized
  P0–P2 findings report. Optionally drafts or opens a PR with GitHub MCP. Use when
  the user asks to review my changes, pre-PR check, is this ready for review,
  check my branch before PR, /pre-pr, or to validate work before a pull request.
---

# Pre-PR review

## Goal

Verify the current branch is ready for review without guessing scripts: run **what the repo and stack reference say**, then **review the diff** and produce a **prioritized report** (P0 blockers → P2 nits). Default: **do not push** and **do not open a PR** unless the user explicitly opts in (step 8).

## When stack is unknown

1. **Detect** the stack (same as `project-bootstrap`): `package.json`, `*.sln`, Gradle/Maven, `pyproject.toml`, etc.
2. **Read exactly one** stack file: `references/stacks/<id>.md`.
3. **Read** the shared references:
   - `references/security-checklist.md`
   - `references/code-quality-checklist.md`
   - `references/commit-message-conventions.md`

| `<id>`            | Detection signals (examples)              |
| ----------------- | ----------------------------------------- |
| `react`           | Vite + React; `vite.config.*`             |
| `next`            | `next` in deps; `next.config.*`           |
| `angular`         | `@angular/core`; `angular.json`           |
| `node-express`    | Express API; not Next as primary app      |
| `python-fastapi`  | `fastapi` / `uvicorn` in Python manifests |
| `java-springboot` | Spring Boot in Gradle/Maven               |
| `dotnet`          | Solution + ASP.NET Core projects          |

Monorepos: run the gate **per package** the branch touches, or as documented in the root README/CI; say which package failed first.

## Workflow (execute in order)

Align with the stack file’s **step order** (it matches `commands/<id>/pre-pr.md` in ai-enablement). Unless the user says otherwise, **stop at the first failing step** and report; do not run later steps to “collect more failures” without asking.

1. **Automated quality** — lint/format, typecheck (if stack uses it), unit tests, integration tests when required by stack, build, coverage thresholds, audits (`npm audit`, `pip-audit`, `dotnet list package --vulnerable`, Gradle dependency check, etc.). **Fix auto-fixable** issues only when safe (formatters, some lint fixes); re-run the step after applying fixes.
2. **Diff quality** — apply `code-quality-checklist.md` to `git diff <base>...HEAD` (use the repo’s default base: `origin/main`, `origin/develop`, or what `CLAUDE.md` / CI says).
3. **Security pass** — apply `security-checklist.md` to changed code and new dependencies; cross-check stack-specific diff-hygiene bullets in `references/stacks/<id>.md`.
4. **Coverage of changed code** — flag new/changed files with no tests when the area is non-trivial (auth, money, migrations, public API).
5. **Commits** — check messages against `commit-message-conventions.md` and any `CONTRIBUTING.md` / `commitlint` config in the repo.
6. **Hygiene** — `console.log` / debug prints, `debugger`, skipped tests without ticket, secrets, commented-out code blocks, unrelated files in the diff (per stack file).
7. **Report** — **P0** (merge blockers: failing checks, security, broken tests), **P1** (should fix: coverage gaps, risky patterns), **P2** (nits: naming, small DRY). Include a **2–3 bullet** draft PR description and suggested reviewers **only** if you can infer from `CODEOWNERS` or git history (optional).
8. **Optional: open PR** — only if the user asks. Use **GitHub MCP** to create the PR with title/body from the report; if MCP is missing, list exact steps and stop.

### Optional: UI / route changes

If the stack is a frontend and the diff changes routes or visible UI, suggest **Playwright MCP** (or the project’s e2e command) to smoke-test changed routes. If MCP is unavailable, say so and rely on the project’s a11y/e2e scripts from the stack file.

## Exemplar commands (ai-enablement)

Full gate recipes live at **`commands/<id>/pre-pr.md`**. When working in this repo, treat those as the **canonical** step list for each `<id>`.

## Anti-patterns

- **Inventing** `pnpm`/`pytest`/`dotnet` scripts that are not in the repo’s manifests or CI.
- **Continuing** the gate after a step fails (unless the user wants a full list of issues).
- **Empty** security review on branches that add routes, auth, or dependency upgrades.
- **False PASS** when tests were skipped because Docker/DB was down without stating **BLOCKED** in the report.
- **Creating a PR** without explicit user request.

## Reference map

- `references/stacks/<id>.md` — order of steps, key commands, stack-specific diff hygiene, optional Playwright.
- `references/security-checklist.md` — injection, auth, secrets, data exposure, dependencies.
- `references/code-quality-checklist.md` — diff review, naming, errors, DRY.
- `references/commit-message-conventions.md` — Conventional Commits and repo overrides.

Valid `<id>` values: `react`, `next`, `angular`, `node-express`, `python-fastapi`, `java-springboot`, `dotnet`.
