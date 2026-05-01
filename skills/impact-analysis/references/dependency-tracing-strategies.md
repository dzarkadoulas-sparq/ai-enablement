# Dependency tracing strategies

## Static graph (default)

- **Start** from the **export surface** of the **target**: every **public** symbol, **re-export** from `index.ts` / `__init__.py` / `Assembly` that maps to the **same** implementation.
- **Fan-in** — `rg` / IDE **Find References** for:
  - **Exact** symbol name
  - **Path**-based import (`from './foo'`, `import ... from '@/lib/foo'`) including **aliases** (`@/`, `~` read from `tsconfig` / Vite / webpack)
- **Tests** — `**/*.test.*`, `**/*.spec.*`, `*Test.*`, `test_*.py` importing or mocking the target.

## Indirect layers

- **BFS** with a **max depth** (e.g. **3**) unless the user needs full graph; label **hops** in the report.
- **Stop** at **known boundaries**: **public HTTP API** (if change is **server-only**), **published package** version boundary, **separate** **deployable** in a **monorepo**.

## Dynamic and hidden dependencies

| Mechanism | What to search |
| --------- | -------------- |
| **String** **routes** / **action** **names** | Grep the **old** string; **i18n** **keys** if the change renames user-visible keys |
| **Config** (JSON/YAML/env) | Class name, **module** path, **bean** name, **feature** **flag** value |
| **DI** / **locator** | `container.get('Foo')`, **Spring** `@Bean` / **Autowire** by **name** |
| **Reflection** | `Class.forName`, `Assembly.Load`, `importlib`, **MediatR** by **type** in registration |
| **Codegen** | Protobuf, **GraphQL** **schema** **clients**—regenerate, don’t only grep **source**    |
| **Build** / **MSBuild** / **webpack** | **Entry** **points** and **splitChunks** that reference **chunks** by name |

## Monorepos

- **Per-package** **tsconfig** **path**; **project references** in **.NET**; **Gradle** **modules**. Trace **across** packages via **package** name in **imports** (`@org/lib-a`).

## Public API and versioning

- **REST** / **gRPC** **protos** — any **other** **service** or **client** in **other** **repos** (out of **repo** **scope**—**flag** as **external** **risk**).
- **DB** **migrations** — `ALTER` that **assumes** **column** **shape**; **read** path in **app** and **views**.

## Risk rubric (suggested)

- **Low** — **internal** **module** only, **covered** **by** **tests**, **static** **refs** **only** **in** **one** **app**.
- **Medium** — **Multi-feature** use, some **mocks** / **fixtures** to **update**; or **1** **dynamic** **ref** you **found** **and** **can** **fix** **in** **one** **PR**.
- **High** — **Widespread** **public** **API**; **unknown** **dynamic** **refs**; **missing** **tests** on **critical** path; **migrations** with **downtime** or **data** **backfill**.

## Effort (T-shirt to phases)

- **S** — **renames** with **IDE** / **TypeScript** **project** **or** **few** **test** **updates**
- **M** — **multiple** **modules**; **new** **adapter** **layer** **optional**
- **L** — **strangler** **or** **parallel** **implementation** + **switch**; **phased** **rollout** **recommended**

## Phased change (when to suggest)

- **Strangler** — new **code** path **next** to **old**; **move** **callers** in **waves** (by **feature** **flag** or **package**)
- **Adapter** — **temporary** **shim** **mapping** old → new **contract** until **migrated**
- **Big-bang** **only** when **blast** **radius** is **proven** **small** and **test** **safety** **net** **exists**

## Tools (generic)

- **`rg`**, **`git grep`**, **IDE** “Find all references”
- **LSP** / **TypeScript** **`tsserver`**, **OmniSharp**, **Pyright**—when the **agent** **can’t** run them, **instruct** the user to **confirm** **ref** **count**
