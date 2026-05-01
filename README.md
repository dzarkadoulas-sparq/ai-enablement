# AI Enablement

This repository helps software developers and teams get the most from **AI coding assistants** such as **Claude Code** and **Cursor**. It's a collection of reusable **project context** (`CLAUDE.md`), **slash-style command prompts** (`commands/`), **Agent Skills** (`skills/`), and **course modules** (`modules/`) that you can copy into real repositories or use as teaching examples.

The materials assume you will connect assistants to your toolchain using the **Model Context Protocol (MCP)** so the agent can work with Git hosting, work tracking, chat, browsers, and API tools—not only local files and the terminal.

_This repository ships no code, only templates you copy into your project._

## New to AI coding assistants? Start here

1. **Pick your stack** — find your project type in the table below (React, .NET, Python, etc.).
2. **Copy the matching `claude.md/<stack>/CLAUDE.md`** into your project root — this gives the assistant instant context about your codebase.
3. **Try a command** — paste any file from `commands/<stack>/` into the chat (e.g. `pre-pr.md` before opening a pull request).
4. **Level up with skills** — read [`skills/README.md`](skills/README.md) to learn which skills fit your day-to-day tasks and how to install them.
5. **Go deeper** — work through the `modules/` lessons in order; Module 1 (prompting) and Module 2 (agentic workflows) are the best starting points.

---

## Repository layout

| Path         | Purpose                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `templates/` | Template **CLAUDE.md** files per project type: stack-specific rules, commands, and conventions for the assistant. **Cursor** users can reuse the same content as a root **`.cursorrules`** file (see below).                                                                                                                                                                                                                                                                                                                                                                                             |
| `commands/`  | **Reusable prompts** (markdown) organized by project type, e.g. pre-PR checks, new features, security scans, standup prep, and shipping work (Jira vs Azure DevOps variants).                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `skills/`    | **Claude / Cursor skills** — start with the tutorial [`skills/README.md`](skills/README.md) (triggers, usage, how skills combine, and how to extend the baseline). **Phase 1–3:** `project-bootstrap`, `pre-pr-review`, `test-writer`, `security-audit`, `git-cleanup`, `conflict-resolver`, `impact-analysis`, `api-test-suite`, `refactor-guide`, `ship-it`. **Phase 4 (specialized):** `standup-prep`, `arch-doc-generator`, `debug-with-playwright` — all **13** **recommended** skills are implemented. **Bonus:** `prep-sprint-work` (Modules 3, 6, 7), documented in the same `skills/README.md`. |
| `modules/`   | **Course-style lessons** (prompting, agentic workflows, MCP, Git, testing, refactoring, security).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |

---

## Project types and languages

Content is aligned with **seven project stacks** (see subfolders under `templates/` and `commands/`):

| Folder            | Stack                 | Language / runtime |
| ----------------- | --------------------- | ------------------ |
| `react`           | React (Vite) SPA      | **TypeScript**     |
| `next`            | Next.js app           | **TypeScript**     |
| `angular`         | Angular               | **TypeScript**     |
| `node-express`    | Node.js + Express API | **TypeScript**     |
| `python-fastapi`  | Python + FastAPI      | **Python**         |
| `java-springboot` | Java + Spring Boot    | **Java**           |
| `dotnet`          | .NET                  | **C#** / .NET      |

**TypeScript** is covered in **four** of the seven (`react`, `next`, `angular`, `node-express`). **Python**, **Java**, and **.NET / C#** each have one dedicated stack.

---

## Using `templates/`

Each subdirectory contains a **CLAUDE.md** tailored to that stack (for example `templates/react/CLAUDE.md`).

**What it is:** “Project intelligence” the assistant should treat as binding: how to install and run the app, test and lint commands, architecture rules (folder layout, state, API style), and common pitfalls.

**How to use it:**

1. **Copy** the file that best matches your repo (or merge sections into an existing `CLAUDE.md` at the repository root, or under `.claude/CLAUDE.md` if you prefer that layout in Claude Code).
2. **Edit** paths, package manager, and team-specific rules so they match the real project.
3. In **Claude Code**, keep `CLAUDE.md` where [Claude Code settings](https://docs.anthropic.com/en/docs/claude-code/settings) expect it (project root is typical).
4. In **Cursor**, use the **same** template content at the project root in a file named **`.cursorrules`** (for example, copy the matching `templates/<stack>/CLAUDE.md` and save it as `.cursorrules`). That file supplies project rules to Cursor’s agent.

Referencing a template in documentation or onboarding helps new contributors and keeps assistant behavior consistent across branches.

---

## Using `commands/`

Commands are **markdown prompt recipes**, not executables. Each file describes a workflow in plain steps (shell commands, checks, and what to do when something fails).

**Structure:** One subfolder per project type, mirroring `templates/` (e.g. `commands/react/`, `commands/dotnet/`).

**Examples of patterns you will see:**

- **Quality gates:** `pre-pr.md`, `security-scan.md`
- **Feature work:** `new-feature.md`, `new-component.md` (or framework-specific names like `new-page.md`, `new-action.md`)
- **Debugging:** `debug-ui.md`, `debug-api.md`, `debug-route.md`
- **MCP-friendly flows:** `standup-jira.md` / `standup-ado.md`, `ship-jira.md` / `ship-ado.md`

**How to use them:**

1. Open the file for **your** stack and copy the text into the assistant chat, or register it as a **Custom Command** / **slash command** in your tool if you use that feature.
2. Replace any placeholders (branch names, ticket keys, environment names) with your team’s values.
3. For Jira- vs ADO-specific commands, pick the file suffix that matches **your** work-tracking setup (`*-jira.md` vs `*-ado.md`).

After you configure the **Jira (Atlassian)** or **Azure DevOps** MCP (see below), prompts that reference issues, boards, or pipelines can be executed with live data.

---

## Using `skills/`

**Skills** package repeatable workflows (bootstrap a repo, pre-PR review, test writing, security audit, etc.) with optional `references/` for language- or stack-specific detail.

- **`skills/README.md`** — **Tutorial** for all skills: **when to use** each, example prompts, **what you get**, links to each **`SKILL.md`**, how skills **compose**, **suggested** order to learn them, and how the baseline set can be **extended** for a specific project. Phases: Foundation → Git & analysis → Composed workflows → Specialized, plus the **bonus** `prep-sprint-work` skill. `skills/skill_recommendations.md` is a **stub** that points here for backward compatibility.

**Status:** The **thirteen** recommended skills are **implemented** under `skills/`, plus the **bonus** `prep-sprint-work`. For stack coverage, Phase 1 uses `references/stacks/` and/or `language-specific/` (seven project types) where applicable; `ship-it` and several specialized skills do **not** ship seven stack files in their own folder by design. Copy skill folders into `~/.claude/skills/`, project `.claude/skills/`, or your Cursor skills location, per your tool’s documentation.

---

## MCP integrations (recommended)

These materials are **intended to be used together with MCP** so the assistant can:

- **GitHub** — issues, pull requests, repository context
- **Playwright** — browser automation and UI verification
- **Jira (Atlassian Cloud)** _or_ **Azure DevOps** — work items, sprints, depending on the organization
- **Slack** — team notifications and standup context
- **Postman** — collections, environments, and API testing workflows

Install and configure servers in the host (Cursor: `mcp.json`; Claude Code: `claude mcp` / project `.mcp.json`). Start with a small set of servers and add more as needed—too many tools at once can slow or confuse tool selection.

### Host documentation (install & configure MCP)

| Product         | Official documentation                                                                                                                                                                                                                                                                                     |
| --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Cursor**      | [Model Context Protocol (MCP)](https://cursor.com/docs/mcp) · [MCP integrations](https://cursor.com/help/customization/mcp) (UI: **Settings → Tools & MCP**; project file: `.cursor/mcp.json`)                                                                                                             |
| **Claude Code** | [Connect Claude Code to tools via MCP](https://docs.anthropic.com/en/docs/claude-code/mcp/) (also published at [code.claude.com](https://code.claude.com/docs/en/mcp)) · [Claude Code settings](https://docs.anthropic.com/en/docs/claude-code/settings) (MCP in `~/.claude.json` and project `.mcp.json`) |

### Service-specific references (servers you may add)

| Integration                             | Where to start (official / upstream)                                                                                                                                                                                                                                                                                                          |
| --------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **GitHub**                              | [github/github-mcp-server](https://github.com/github/github-mcp-server) (remote and local options, PAT / OAuth)                                                                                                                                                                                                                               |
| **Playwright (browser automation)**     | [microsoft/playwright-mcp](https://github.com/microsoft/playwright-mcp) (see repo README for Cursor / Claude Code examples)                                                                                                                                                                                                                   |
| **Jira / Confluence (Atlassian Cloud)** | [Atlassian Rovo MCP Server — use Atlassian Rovo MCP Server](https://support.atlassian.com/rovo/docs/atlassian-remote-mcp-server/) · [Setting up IDEs (desktop clients)](https://support.atlassian.com/atlassian-rovo-mcp-server/docs/setting-up-ides/) (includes Cursor)                                                                      |
| **Azure DevOps**                        | [Enable AI assistance with the Azure DevOps MCP Server](https://learn.microsoft.com/azure/devops/mcp-server/mcp-server-overview) · [Set up the remote Azure DevOps MCP Server (preview)](https://learn.microsoft.com/azure/devops/mcp-server/remote-mcp-server) · [microsoft/azure-devops-mcp](https://github.com/microsoft/azure-devops-mcp) |
| **Slack**                               | [Slack MCP server — API docs](https://api.slack.com/ai/slack-mcp-server)                                                                                                                                                                                                                                                                      |
| **Postman**                             | [Postman docs — MCP servers overview](https://learning.postman.com/docs/postman-ai-developer-tools/mcp-servers/overview) · [postmanlabs/postman-mcp-server](https://github.com/postmanlabs/postman-mcp-server) (remote URL, OAuth, and local npm package)                                                                                     |

Exact `mcp.json` / `claude mcp add` snippets change as vendors update servers; use the links above for current transport (stdio vs HTTP), OAuth, and required environment variables.

---

## Course modules

The `modules/` directory contains topic guides (e.g. effective prompting, agentic workflows, **MCP server integration**, Git, testing, refactoring, configuration and security) that align with the commands and skills in `skills/`.

---

## License and usage

This project is created by Dimitris Zarkadoulas and licensed under the MIT License. Co-authored with Claude by Anthropic and with Cursor — see DISCLAIMER.md for details on AI-generated content, warranty, and support.
