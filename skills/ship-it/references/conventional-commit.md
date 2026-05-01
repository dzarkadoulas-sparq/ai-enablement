# Conventional commit from the diff

Use the same **Conventional Commits** rules as `pre-pr-review`’s `references/commit-message-conventions.md` and `git-cleanup`’s `references/conventional-commits.md` in this repository. This file is a **short** decision guide for the ship step.

## Subject line

- Pattern: `type(optional-scope): short description`
- Optional ticket suffix or bracket: `fix(api): handle null user [PROJ-123]` or footer `Refs: PROJ-123` per team style.

## Map change to `type`

| Signal                                      | `type`          |
| ------------------------------------------- | --------------- |
| New feature, new API, user-visible behavior | `feat`          |
| Bug fix                                     | `fix`           |
| Tests only                                  | `test`          |
| Docs, comments only                         | `docs`          |
| Refactor, no behavior change                | `refactor`      |
| Build, CI, tooling                          | `chore` or `ci` |
| Performance                                 | `perf`          |

## Scope

- Prefer a real module: `api`, `auth`, `ui`, package name, or folder root the diff touches most.
- If the change is huge, the scope can be broad; the PR description should list areas.

## When the diff mixes types

- Prefer **one** commit with the **dominant** type, or **split** commits (after user agrees) for unrelated changes.

## Body

- Add a body when: breaking change, migration, or complex test plan; list **what** and **why** in two to five lines.

## What not to do

- Vague messages: `update`, `fix stuff`, `wip`, `address comments` without context (unless the team’s intermediate commits are squashed later via `git-cleanup`).
