---
name: debug-with-playwright
description: >-
  Diagnoses frontend issues using a real browser via Playwright MCP: navigate,
  set viewport, run the user’s repro steps, capture screenshots, console output,
  and network activity, then explain likely causes. Can suggest or apply code
  fixes and re-run to verify. Use for debug in browser, visual bug, screenshot
  the page, what happens when I click, UI broken, or check responsive layout.
---

# Debug with Playwright

## Goal

Reproduce and **document** a **UI** or **client-side** problem with **evidence** (screens, console, network) so the fix can be reviewed. Prefer the **Playwright MCP** when it is configured; otherwise **degrade** with clear manual steps and optional **Playwright test** skeletons (see `references/mcp-and-fallbacks.md`).

## Ideal dependency

- **[Playwright MCP](https://github.com/microsoft/playwright-mcp)** — connected in Cursor or Claude Code per the project root `README` (MCP table).

If the MCP is **absent**, do **not** pretend you ran the browser; use the fallback path in `references/mcp-and-fallbacks.md`.

## Work with other skills in this repo

- **`pre-pr-review`** — After a fix, run the same **lint / typecheck / tests** the stack defines (see `pre-pr-review/references/stacks/<id>.md` for the **seven** project types). Frontend stacks often list **optional** Playwright steps in `commands/<stack>/pre-pr.md`.
- **`test-writer`** — If the user wants **regression** coverage, add or extend a **Playwright** spec in the project’s e2e layout; follow that skill’s stack files for where e2e usually lives.

## Preconditions

- **URL** — local dev server (e.g. `http://localhost:5173/...`) or a **named** environment; never hit **production** with destructive actions unless the user explicitly says so and scope is read-only.
- **App running** — Start the dev server with the project’s real command (from `package.json` / `CLAUDE.md`) or ask the user to start it. **Do not** guess ports if the repo documents one.
- **Auth** — If the flow needs login, use test credentials the user provides or **stop** and ask; **never** use or store real user passwords in docs.

## Workflow (execute in order)

1. **Clarify** — Expected vs. actual behavior, browser if relevant, viewport (mobile / desktop) if layout-related.
2. **Navigate** — Open the target URL; set **viewport** when the user asked for responsive checks.
3. **Reproduce** — Perform the described steps (clicks, typing, navigation). Wait for **network idle** or framework-specific stability when the MCP allows.
4. **Capture** — **Screenshot(s)** of the bad state; **console** messages (errors/warnings); **network** failures (4xx/5xx, CORS, blocked). Summarize, do not paste megabytes of HAR.
5. **Diagnose** — Map symptoms to **likely** causes: component state, bad CSS, bad API response, auth, hydration, race. Cite **files** you **read** in the repo (stack trace to source when available).
6. **Fix (optional)** — If the user wants a code change, edit the **minimal** surface; avoid unrelated refactors.
7. **Re-verify** — Re-run the same browser steps after the fix; compare screenshot or behavior.
8. **Report** — Use the structure in `references/report-template.md` (repro, evidence, conclusion, follow-ups).

## Safety and privacy

- **No** screenshot or log that contains **secrets**, session tokens, or PII in **deliverable** text—**redact** in descriptions.
- **No** auto-posting to Slack or tickets; the user copies artifacts.

## Anti-patterns

- Claiming “I opened the page” when the MCP was not invoked or failed—state **not run** and why.
- **Vague** fixes without tying to a **file** and **line**-level hypothesis.
- Running **arbitrary** JavaScript in the page context without a clear debugging need and user agreement.

## Reference map

- `references/mcp-and-fallbacks.md` — When MCP is missing; official link; local Playwright CLI patterns.
- `references/report-template.md` — How to write the debug report.

## Exemplar prompts (ai-enablement)

- **Pre-PR** and **ship** flows that use Playwright for UI checks: `commands/<stack>/pre-pr.md` and `ship-jira.md` / `ship-ado.md` (e.g. visual verification steps). Match the **stack** folder to the project type.
