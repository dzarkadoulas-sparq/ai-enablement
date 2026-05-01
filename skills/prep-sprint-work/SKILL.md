---
name: prep-sprint-work
description: >-
  Prepares development for a new feature or story: pulls work-item details from
  Jira or Azure DevOps, researches related code, plans architecture impact with
  the user, updates ADR and architecture docs, and opens a documentation PR for
  review before implementation. Use for prepare my sprint work, prep sprint work,
  prepare story, or start preparing development for a new feature.
---

# Prep sprint work (bonus)

## Goal

Turn one or more **stories** or **features** into a **reviewed** package: **ingested** requirements (from the tracker), **grounded** codebase findings, **explicit** architecture decisions, **updated** architecture/ADR documents, a **written** implementation plan with a **dependency graph**, and a **pull request** that contains **documentation only** (so the team can approve direction before code).

## Modules

- **Module 3** — Jira or Azure DevOps MCP for assignment, issue details, and links.
- **Module 6** — Codebase understanding, impact and dependencies.
- **Module 7** — Configuration, automation, and safe change process (PRs, conventions).

## Composes other skills in this repo

- **`arch-doc-generator`** — ADR shape (`references/adr-template.md`), `artifacts.md` and `validation.md` metadata, and document tone. **Do not** duplicate long templates; import their rules by path.
- **`impact-analysis`** — Tracing importers, routes, and **module/service** edges for the dependency **graph**.
- **`ship-it`** (or the same **Git** + **GitHub** / `gh` steps) — **Only** for the **docs PR** (branch, commit message, open PR, optional ticket comment with PR link). **Do not** use `ship-it` to ship application code in the same run unless the user **explicitly** changes scope.
- **`standup-prep` / `ship-it` references** — Jira/ADO **fallback** ideas appear in `standup-prep/references/mcp-queries.md` when MCP is missing.

## Work-item selection

1. **If the user names** a story/feature (key, title, or link) — that set is **in scope**. They may name **several** or say **both**, **all** (in context), or a list like **`PROJ-123` and `PROJ-124`** for related work (e.g. shared architecture/ADR updates first). **Confirm** the final key list in one line before heavy research.
2. **If nothing is specified** — use **Jira** or **Azure DevOps** MCP to list work items **assigned to the current user** in a sensible scope (active sprint/iteration, or the board/team the user names). **Present** key, title, status, and a one-line **ask** which to prepare (or which subset). If MCP is unavailable, use **`references/mcp-work-items.md`** (Fallback) and **stop** for keys/titles from the user.
3. **Multi-item runs** — Merge requirements into a **single** plan when stories are coupled; keep **per-key** subsections for acceptance criteria and traceability.

## Ingest work-item content

For each selected key, gather **as available** from the tool (see `references/mcp-work-items.md`):

- Title, description, **acceptance criteria** (field or text), **status** and **issuetype**
- **Comments** (recent first; summarize and cite “comment by @user, date”)
- **Attachments** — list **names and purpose**; **do not** download or open arbitrary binaries in an automated way. Ask the user to **summarize** or paste if the content is **essential** and not text-extractable. **Never** exfiltrate or log secrets from attachments.
- **Links** to Confluence, Figma, or other specs (record URLs; fetch only if a safe read tool exists).

## Codebase research

1. **Detect** stack and layout (`project-bootstrap`-style).
2. Map each acceptance criterion to **likely** code areas (routes, services, models, **bounded contexts**). Use **`impact-analysis`** for **import/route** and config edges.
3. Record **unknowns** and **assumptions**; label inferences in the plan.

## Structured planning (“plan mode”)

Use a **dedicated planning pass** before **application** code changes. If the host (e.g. Cursor) exposes **Plan** mode, run **this** workflow inside it; otherwise, follow the same order **without** committing to implementation until the user has reviewed **architecture** decisions. Phases: **`references/planning-phases.md`**.

In planning:

- Explain **how** the work could affect **architecture** (new boundaries, API contracts, data ownership, feature flags, rollout).
- Propose **decisions** (or **TBD** with options). **Pause** and ask the user to **review and approve** or **revise** these decisions.

## After the user approves architecture decisions

1. **Update documents** under paths the repo already uses (`docs/adr/`, `docs/architecture/`, etc.) or create them following **`arch-doc-generator`** conventions. **Minimum** for this skill’s happy path: **at least one** ADR (if a decision is new or changes prior direction) and **updates** to an architecture overview or **module** map as needed. Use **`validation.md`**-style **metadata** on generated or heavily edited files.
2. **Write the implementation plan** (see `references/implementation-plan.md`) — **phased** tasks, risks, and a **Mermaid** (or table) **dependency graph** of affected **modules / services / components** (name real packages/projects/files where known).
3. **Exit planning** for application code: the default deliverable of this skill is a **docs PR** plus the **implementation plan** in the PR **description** or a linked `docs/.../implementation-plan-*.md` in the same branch. **No** feature code in this PR unless the user explicitly extends scope.

## Open the documentation PR

1. Branch: e.g. `docs/sprint-prep-<KEY>` or `chore/sprint-prep-<KEY1>-<KEY2>`.
2. Conventional Commits, e.g. `docs(arch): align ADR and plan for PROJ-123` (adjust to team style).
3. PR title/body: link **each** work item, summarize **decisions** and where **docs** were updated, **paste** or **link** the **dependency graph**, list **reviewers** (e.g. tech lead) as per `ship-it` / `integrations.md` patterns.
4. If Jira/ADO MCP is available, **comment** the PR URL on the work item(s) and transition only if the user asked.

## Anti-patterns

- **Inventing** work-item fields, assignment, or state when MCP did not return them.
- **Starting** app implementation before **architecture** review when the user wanted **doc-first** prep.
- **Bloating** the docs PR with unrelated refactors or `package.json` lock churn.
- **Omitting** a dependency **graph** when multiple modules or services are touched.
- **Silent** attachment download of untrusted files.

## Reference map

- `references/mcp-work-items.md` — Jira/ADO: assigned work, **fetch** fields, **fallbacks**.
- `references/planning-phases.md` — Plan mode vs. exit, ordered phases.
- `references/implementation-plan.md` — Plan template and **Mermaid** graph for affected parts.

## Exemplar (ai-enablement)

- Same **Jira vs. ADO** **distinction** as `standup-*.md` and `ship-*.md` under `commands/<stack>/` — pick the command family that matches the team’s work tracker for **vocabulary** in prompts, not for stack.
