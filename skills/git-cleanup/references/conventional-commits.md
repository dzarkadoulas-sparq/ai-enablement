# Conventional Commits (1.0.0-style)

Use for **reworded** and **new** commit subjects after cleanup. If `CONTRIBUTING` or **commitlint** defines stricter rules, **those win**.

## Format

```
<type>[optional scope][optional !]: <description>

[optional body]

[optional footer(s)]
```

- **Description:** imperative, lowercase start is common, **~50 chars** for subject line (soft limit).
- **!** after `scope` or `type` indicates a **BREAKING CHANGE** (or spell out in **footer**).

## Common types

| type           | Use                                                            |
| -------------- | -------------------------------------------------------------- |
| `feat`         | new user-visible capability                                    |
| `fix`          | bug fix                                                        |
| `docs`         | documentation only                                             |
| `style`        | format, no code meaning (avoid if “style” is confused with UI) |
| `refactor`     | neither feat nor fix                                           |
| `perf`         | performance                                                    |
| `test`         | add/fix tests only                                             |
| `chore`        | build, CI, deps bump without feature fix                       |
| `build` / `ci` | some teams split from `chore`                                  |

## Scope

- Optional parenthesized **area**: `feat(api):`, `fix(auth):`. Match **monorepo** package or **layer** as the repo does.

## Body and footers

- **Body** explains **why** / **context**, not just **what** (the diff shows what).
- **Footer:** `BREAKING CHANGE: description` or `Closes #123` / `Refs JIRA-456` if the project uses that.

## Multiple commits after cleanup

Typical post-cleanup set:

1. `feat(scope): add …` (main implementation)
2. `test(scope): …` (if team splits tests) **or** fold into one `feat` if one commit is preferred
3. `chore: …` only for unrelated noise removed in the same PR (ideally **avoid** mixing)

## What to avoid

- **Vague** subjects: "fix stuff", "WIP", "update", "PR feedback" without `scope` and concrete verb.
- **One giant** `feat` that combines **unrelated** tickets—prefer **separate** commits for **separable** reverts.
