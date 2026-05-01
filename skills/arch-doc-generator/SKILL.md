---
name: arch-doc-generator
description: >-
  Produces architecture documentation grounded in the repository: ADRs, module
  dependency maps, data-flow traces, technical-debt inventory, API surface (e.g.
  OpenAPI) when present, and a developer onboarding outline. Adds generation
  metadata and validates every claim with file or symbol references. Use for
  document the architecture, generate an ADR, technical debt report, data flow
  for X, or onboarding doc from code.
---

# Architecture doc generator

## Goal

From **read-only** analysis of the codebase (and **existing** OpenAPI/GraphQL/schema files if present), produce **structured, auditable** documentation. Every non-trivial statement should tie to a **concrete** path, symbol, or config key—or be labeled **inference** with low confidence. **Do not** invent services, env vars, or data stores that are not discoverable in the tree.

## Work with other skills in this repo

- **`impact-analysis`** — When building a **module or package dependency map**, or tracing who imports a package, use `impact-analysis`’s workflow and `references/dependency-tracing-strategies.md` (and `impact-analysis/references/stacks/<id>.md` for language specifics).
- **`project-bootstrap`** — Reuse the **stack detection** table, `references/architecture-patterns.md` for **smells and debt** signals, and the **onboarding/CLAUDE** section shape for “developer quickstart” output when the user wants parity with a generated `CLAUDE.md`—without replacing `project-bootstrap` when the only ask is a full `CLAUDE.md` (suggest that skill for a single load-bearing file).

## Inputs (ask or infer)

- **What to produce** — One or more of: ADR, dependency map, data flow for a named flow, debt inventory, OpenAPI/API doc refresh, onboarding guide.
- **Target** — Whole repo, one service/package, or one user journey (e.g. “checkout” end to end).
- **Depth** — Executive (one page) vs. engineering (file-level citations).

## When stack is unknown

1. **Detect** `<id>` as in `project-bootstrap` (manifests, build files).
2. For **traces, imports, routes**, read `impact-analysis/references/stacks/<id>.md` once.
3. Read **`references/artifacts.md`** in this folder for the shape of each deliverable.

## Workflow (per requested artifact; merge into one report or separate files as the user prefers)

1. **Inventory** — Top-level layout, packages/apps, CI entry points. Record **date** and **user prompt** for the metadata block (`references/validation.md`).
2. **ADR (optional)** — Use `references/adr-template.md`. Base decisions on **code/config** and PR/issue links if the user provides them; otherwise name the **observed** choice and mark alternatives as TBD.
3. **Module dependency map** — Packages/projects and **import/dependency** edges. Prefer build-graph tools when available (`tsc --traceResolution` is not always on—use the strategies in **`impact-analysis`** + grep/search). **Graph** = nodes are modules/packages, edges are “depends on” from manifests or static imports.
4. **Data flow trace** — Pick **entry** (HTTP route, CLI, job) and follow **one** path: handler → service → data access, naming files. If dynamic dispatch hides the callee, **say so** and list candidate files.
5. **Technical debt** — `TODO`/`FIXME`, deprecated APIs, high cyclomatic or god-class heuristics from the stack; cross-check `project-bootstrap/references/architecture-patterns.md`. Score **severity** and **effort** as **S/M/L** only with explicit rationale or mark **unknown**.
6. **API documentation** — If `openapi.json`, `swagger`, FastAPI’s generated schema, or `.yaml` in repo, **synchronize** a summary or the spec path; if generation is a **build** step, document the **command** from `package.json` / `Makefile` / CI. **Do not** hand-write a full OpenAPI for a large API in one go unless the user asked for **drafting** a new spec; prefer “here is what exists; gaps are X.”
7. **Onboarding** — Prereqs, install, test, run, where to add a feature, link to the ADR/dependency map. Mirror **facts** from README/CONTRIBUTING/CI, not ideal practice.
8. **Validate** — Pass the **validation checklist** in `references/validation.md` (evidence, boundary cases, no fabrication).
9. **Output** — Markdown under `docs/architecture/`, `docs/adr/`, or paths the user names. Include the **metadata** footer: generated date, prompt, git commit or branch if available.

## Anti-patterns

- Filling an ADR with **generic** “we use microservices” when the repo is a **monolith**—match reality.
- **Omitting** how you traced a data flow; reviewers need **files** to verify.
- Copying a **huge** OpenAPI into the chat instead of a **link/path** and a **delta** summary.
- Conflating **product roadmap** (unknown) with **debt in code** (evidence-based).

## Reference map

- `references/adr-template.md` — ADR sections and status values.
- `references/artifacts.md` — What each output type should contain.
- `references/validation.md` — Metadata block and “evidence for every claim” rules.

## Exemplar shapes (ai-enablement)

- Full stack templates for assistant rules live in `claude.md/<id>/CLAUDE.md`; this skill’s **onboarding** section can align in tone, but it should still reflect **this** project’s real paths and scripts.
