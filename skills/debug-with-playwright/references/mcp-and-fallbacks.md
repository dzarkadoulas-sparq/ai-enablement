# Playwright MCP and fallbacks

## Official host + server

- **Server:** [microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) — install and transport (stdio / HTTP) follow the repo README; Cursor and Claude Code examples are usually there.
- **Host docs** — Project root `README` in this training repo: **Cursor** and **Claude Code** MCP configuration links.

## When the MCP is available

- Prefer tool calls the server exposes (navigate, click, type, screenshot, console, network as supported). **Names** and **args** differ by version—read the **descriptor** or server docs for the instance the user runs.

## When the MCP is not available

1. **State clearly** that live browser control was not used.
2. **Give manual steps** — open URL, open DevTools (console, network), repro steps, what to look for.
3. **Optional** — if the repo already has **Playwright** or **Cypress** in `package.json`, suggest: `pnpm exec playwright test` (or the script name in the project) and point to the **`e2e/`** or **`tests/e2e/`** folder if present.
4. **Code-only triage** — read **React/Vue/Angular** components, **styles**, and **data-fetch** code for the route; this **does not** replace a browser for layout or race bugs.

## Local dev server

- Use the **exact** command from the project’s `package.json` / `README` / `CLAUDE.md` (e.g. `pnpm dev`, `npm start`). Confirm **port** from console output or docs—**do not** assume `3000` vs `5173`.

## Auth and environments

- Staging vs. local: **use what the user names**. For **SSO** or **2FA**, say that automated login may be **blocked** and ask for a **session** approach or a **reduced** repro (storybook, isolated page).
