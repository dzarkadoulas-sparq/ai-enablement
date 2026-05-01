# Deliverable types

## Architecture Decision Record (ADR)

- One **decision** per file when possible; use `adr-template.md`.
- Titles must be **searchable** (e.g. “use PostgreSQL for job store” not “database choice” without context).

## Module dependency map

- **Scope** — repo root, a solution, a package, or a monorepo workspace.
- **Nodes** — buildable units (NPM package, .NET project, Gradle module, Python package path).
- **Edges** — dependencies from manifests, project references, or high-confidence static `import`/`include` from public entry points (note **dynamic** imports as dashed or “see runtime”).
- **Output** — List or mermaid `graph` in Markdown; mermaid is optional if the user’s renderer supports it; otherwise a table **From → To → evidence**.

## Data flow trace

- **One** primary scenario per request (e.g. “user submits form X”).
- **Sections:** Entry (route, queue, event) → **Middleware/validation** → **Domain** → **Persistence/IO** → **Response/side effect**.
- **Citations:** File path and symbol or line range where reasonable.

## Technical debt inventory

- **Item** | **location** (path) | **symptom** (pattern, `TODO` key, or metric) | **suggested** fix class (not a promise) | **risk** (H/M/L).
- Prefer **debt the code admits** (comments, `deprecated`, skipped tests) over stylistic nits.
- Distinguish **inherited** (third-party) vs. **our** code.

## API / contract documentation

- **If a machine-readable spec exists:** path to the file, how it is **produced** (script in `package.json`, CI), and a **summary** of resources/methods touched by recent changes (if the user asked for “current” doc).
- **If missing:** a **gap** list: routes discovered from framework registration vs. an explicit spec, and a **small** hand-maintained list for the **public** surface only (avoid inventing 200 endpoints from scratch in one pass).

## Developer onboarding guide

- **Audience:** new dev on this repo, not a generic “how to write React” tutorial.
- **Sections:** prerequisites, clone, **exact** install from README or manifest, run app, run tests, lint, common tasks (“add an endpoint,” “add a feature flag”), where architecture docs live, link to 1–2 key ADRs or the dependency map.
- Reuse `project-bootstrap`’s “short onboarding” spirit; **de-duplicate** if the repo already has `CONTRIBUTING`—**extend** or **summarize** with links.
