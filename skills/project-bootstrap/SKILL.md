---
name: project-bootstrap
description: >-
  Onboards a developer to an unfamiliar repository and produces a production-grade
  CLAUDE.md (and a short developer onboarding guide). Reads manifests, maps architecture,
  traces main request flows, surfaces TODOs and risks, and tailors output to the stack.
  Use when the user says onboard me, explore this codebase, I just joined, what does
  this project do, generate a CLAUDE.md, or on first touch with a new repo.
---

# Project bootstrap

## Goal

Produce two artifacts:

1. **`CLAUDE.md`** (project root, or `.claude/CLAUDE.md` if the team uses that layout) — load-bearing rules: commands, architecture, style, security, pitfalls.
2. **A short onboarding doc** — e.g. `docs/ONBOARDING.md` or a new section in `README.md` — how to run, test, where things live, and who to ask for what (only what you can infer from the repo).

For **Cursor**, the same content can live in **`.cursorrules`** at the project root instead of or in addition to `CLAUDE.md`.

## When stack is unknown

1. **Detect** the stack using the table below (read `package.json`, `*.csproj`, `pom.xml` / Gradle, `pyproject.toml`, `go.mod`, etc.).
2. **Read exactly one** stack file: `references/stacks/<id>.md` for the matching `<id>`.
3. **Read** `references/architecture-patterns.md` for cross-cutting patterns and red flags.
4. If detection is **ambiguous** (e.g. monorepo with both Next and an API package), ask one short question or document both packages separately in one `CLAUDE.md` with a **Monorepo** top section.

| `<id>`            | Detection signals (examples) |
| ----------------- | ---------------------------- |
| `react`           | Vite + React; `index.html` at root; `vite.config.*`; often no `next` in deps |
| `next`            | `next` dependency; `app/` or `pages/`; `next.config.*` |
| `angular`         | `@angular/core`; `angular.json` |
| `node-express`    | `express` + Node API; `src/index.ts` / `server.ts`; not Next |
| `python-fastapi`  | `fastapi`, `uvicorn` in `pyproject.toml` or requirements |
| `java-springboot` | `spring-boot-starter-*` in Gradle/Maven; `@SpringBootApplication` |
| `dotnet`          | `.sln`, `Microsoft.NET.Sdk`, ASP.NET Core host project |

## Workflow (execute in order)

1. **Scan structure** — list top-level dirs, find apps/packages in monorepos, identify test and config locations.
2. **Read manifests** — lockfiles, build files, CI workflows (`.github/workflows`, `azure-pipelines.yml`). Extract real **install / build / test / lint** commands; prefer scripts that CI runs over guesses.
3. **Identify architecture** — layering, feature folders, public API surface; note framework versions from manifests.
4. **Trace one happy path** — e.g. HTTP request to handler → service → DB, or App Router `page` → data fetch → component. Name specific files you followed.
5. **Surface tribal knowledge** — search for `TODO`, `FIXME`, `HACK`, `XXX`, commented-out code blocks; list high-signal items (not every trivial TODO).
6. **Red flags** — apply stack file + `architecture-patterns.md` (god classes, missing tests for critical paths, abandoned deps, secrets in code).
7. **Write `CLAUDE.md`** — use the **section list** in the stack file. Mirror tone: load-bearing rules, "Read first" list, strict architecture and security blocks. **Do not** invent scripts; if unknown, say "TBD" and point to the file to confirm.
8. **Write onboarding** — quickstart, link to `CLAUDE.md`, branch/PR convention if visible in `CONTRIBUTING` or CI.

## Exemplar shapes (ai-enablement)

If this repo is **ai-enablement**, each stack has a full exemplar at `claude.md/<id>/CLAUDE.md`. When generating for **other** repos, treat those exemplars as **structural** guides only; replace with this repo’s actual tools and paths.

## Anti-patterns

- **Fabricating** npm/pnpm scripts or paths not present in `package.json` or docs.
- **Dropping** security and env handling sections for web stacks that expose client config.
- **One-size-fits-all** rules that contradict the framework (e.g. React patterns on an Angular app).
- **Omitting** "when a rule and a user request conflict, stop and ask" for generated `CLAUDE.md` when the exemplars use it.
- **Dumping** every TODO from the codebase into `CLAUDE.md` — only actionable or risky ones.

## Reference map

- `references/architecture-patterns.md` — monorepos, layers, request tracing, generic red flags.
- `references/stacks/<id>.md` — detection hints, files to read first, flow to trace, required `CLAUDE.md` sections, stack-specific red flags.

Valid `<id>` values: `react`, `next`, `angular`, `node-express`, `python-fastapi`, `java-springboot`, `dotnet`.
