---
name: refactor-guide
description: >-
  Plans and executes large refactors in phases: scopes the change, requires tests or
  characterization tests first, uses a dependency-aware plan, verifies after each
  phase, and keeps behavior stable unless the goal is an intentional break.
  Composes impact-analysis, test-writer, and pre-pr style checks. Use for refactor,
  extract class, migrate X to Y, framework upgrade, library replacement, or breaking
  up a god class.
---

# Refactor guide

## Goal

Deliver a **safe**, **reviewable** change in **phases**: every phase has a **verifiable** outcome (tests and build green) and a **clear** rollback (commit boundary or feature flag), unless the team’s trunk policy says otherwise.

## Work with other skills in this repo

- **`impact-analysis`** — for **blast radius** and phasing order before big cuts (who imports this, config, dynamic refs).
- **`test-writer`** — add **characterization** tests on critical untested code **before** structural rewrites, unless the user explicitly accepts risk in writing.
- After each phase (or at the end), use the same **checks** as **`pre-pr-review`** (lint, typecheck, test—whatever the project defines).

## Pick a pattern file

Read `references/safety-checklist.md` always, then the pattern that matches the work:

| Situation | Open |
| --------- | ---- |
| Extract / split a class, module, or function          | `references/refactoring-patterns/extract-class.md` |
| Replace one third-party with another in the same role | `references/refactoring-patterns/library-replacement.md`    |
| Major framework or runtime version jump               | `references/refactoring-patterns/framework-upgrade.md` |
| New boundaries, strangler, or module split            | `references/refactoring-patterns/architecture-migration.md` |

Also read **`references/stacks/<id>.md`** for the seven project types to align verify commands and framework quirks (RSC, EF, FastAPI layers, etc.).

## Workflow

1. **Scope** — Goal (behavior preserved vs. intentional break + migration). **In** / **out** list. User-facing vs. internal.
2. **Coverage gate** — If critical code lacks tests, add **characterization** tests first, unless the user waives in writing.
3. **Phased plan** — Order by dependencies (e.g. adapter first, then call sites, then delete legacy). One theme per PR when possible.
4. **Execute one phase** — Minimize diff; no unrelated “cleanup” in the same change.
5. **Verify** — Run the stack’s commands (see `references/stacks/<id>.md` and the repo’s CI / `commands/<id>/pre-pr.md`). Revert or fix the phase if red.
6. **Report** — What was done, what’s left, risks (data, cache, rollout).

**Preserving behavior** — Refactors should not change observable behavior except where the plan **explicitly** allows. If tests or snapshots must change, separate “contract update” from “bug fix” in the report.

## Anti-patterns

- **Big-bang** without tests, or mixing a huge refactor with an unrelated product feature on one branch.
- **Phases** that leave **main** broken for others—use flags or a **long-lived** branch per policy.
- **Removing** code without checking **dynamic** and **config** references (use `impact-analysis` patterns).

## Reference map

- `references/safety-checklist.md`
- `references/refactoring-patterns/*.md`
- `references/stacks/<id>.md` — `react`, `next`, `angular`, `node-express`, `python-fastapi`, `java-springboot`, `dotnet`
