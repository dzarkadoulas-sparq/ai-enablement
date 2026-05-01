---
name: impact-analysis
description: >-
  Maps blast radius for a proposed change: finds direct and indirect consumers
  of the target (imports, calls, routes), flags dynamic and config-based references,
  estimates change difficulty per consumer, and gives an overall risk and phased
  recommendation. Use for what will break, blast radius, who depends on this, or
  pre-change risk assessment.
---

# Impact analysis

## Goal

Before changing a **symbol**, **module**, **API surface**, or **type**, answer: **what breaks**, **what must change with it**, and **whether to split** the work. Output a **trace report** and a **recommendation**: proceed, redesign, or **phase** the change.

## Inputs (ask or infer)

- **Target** — file(s), **exported** symbol name, **route** path, **public** type, or **URL** of an endpoint.
- **Scope** — “replace X with Y” vs “rename only” vs “change behavior of Z.”
- **Branch** — compare to **default** or **mainline** you will merge into.

## When stack is unknown

1. **Detect** the repo type (`package.json`, `*.sln`, `pyproject.toml`, etc.) as in `project-bootstrap`.
2. **Read** `references/dependency-tracing-strategies.md` (always).
3. **Read** `references/stacks/<id>.md` for **language-specific** search and **import** rules.

| `<id>`            | Typical signals          |
| ----------------- | ------------------------ |
| `react`           | Vite + React             |
| `next`            | `next` app               |
| `angular`         | `angular.json`           |
| `node-express`    | Node API (Express, etc.) |
| `python-fastapi`  | FastAPI / Python         |
| `java-springboot` | Spring Boot              |
| `dotnet`          | .NET / ASP.NET Core      |

## Workflow (execute in order)

1. **Anchor the target** — **exports** (barrel `index.ts`, `__init__.py`, `public` class), **route** registration, **DI** registration. If ambiguous, list **candidates** and ask one short question.

2. **Direct consumers** — static **imports** / `using` / `import` of the **module** or **symbol**; **call sites**; **subclasses**; **interface** implementers. Use **ripgrep** and **stack** file patterns; use **IDE** / LSP “find references” if available as a user step.

3. **Indirect consumers** — BFS/expand: **consumers of consumers** until **stability** (no new app-level entry) or **cap** (e.g. 3–4 **hops**) to avoid combinatorial blowup—**state** the cap in the report.

4. **Dynamic / hidden edges** — **String** **keys**, **DI** by **name**, **route** **tables**, **config** **JSON/YAML** **pointing** at class names, **reflection** / `getattr` / Spring `@ConditionalOn*`, **OpenAPI** **operationId** **changes**, **feature** **flags**. Search per `dependency-tracing-strategies.md` and the **stack** file.

5. **Per-consumer assessment** — For each **non-trivial** consumer: **what** must change (signature, DTO, test, mock), **effort** (S/M/L), **risk** (low/med/high) if **missed**.

6. **Roll-up** — **Overall risk** (e.g. **low** = localized + tests; **high** = cross-cutting + dynamic refs). **Effort** (person-days order of magnitude if possible). **Recommendation:** **proceed** / **reduce scope** / **strangler** **phases** (see `dependency-tracing-strategies.md`).

## Report format (use this structure)

- **Target** and **assumption**
- **Direct dependers** — table: path, kind (import / call / route / test)
- **Indirect (hops 2–N)** — summary table or **graph** (mermaid **optional**)
- **Dynamic / config** — list with **evidence** (file:line)
- **Tests** that must run or **change**
- **Risk** + **effort** + **recommendation** + **suggested phases** (if any)

## Anti-patterns

- **Declaring** “no usages” from **one** `grep` without **barrels**, **re-exports**, and **framework** wiring.
- **Infinite** expansion—**stop** and **summarize** when **fan-out** is huge; suggest **automated** “find all references” in **IDE** for the user.
- **Ignoring** **migrations** and **public** **API** **versioning** (mobile clients, other repos).

## Reference map

- `references/dependency-tracing-strategies.md` — dynamic refs, config, phasing, risk rubric.
- `references/stacks/<id>.md` — tools and patterns for that stack.

Valid `<id>` values: `react`, `next`, `angular`, `node-express`, `python-fastapi`, `java-springboot`, `dotnet`.
