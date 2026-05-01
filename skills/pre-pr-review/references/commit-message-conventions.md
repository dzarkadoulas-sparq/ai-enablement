# Commit messages (pre-PR)

## Default: Conventional Commits

If the repo does not define another standard, expect:

```
<type>(<optional scope>): <short description>

[optional body]
```

**Types (common):** `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`, `build`.

- **Imperative mood** — "Add login" not "Added" or "Adds".
- **50–72 chars** for the subject line is a common bar; one logical change per commit when possible.

## Detect team rules (read in order)

1. **`CONTRIBUTING.md`** or **`docs/CONTRIBUTING.md`**
2. **Commitlint** — `commitlint.config.*`, `package.json` `commitlint` field
3. **Husky / lefthook** — pre-commit that enforces message format
4. **CI** — a job that lints commit messages
5. **`.gitmessage`** in repo (template)

If the branch has **n commits** and messages are inconsistent, report **P2** (or P1 if CI enforces) with suggested reword/squash guidance; do not rewrite git history without the user asking.

## PR title

Often mirrors the main commit or Conventional-Commits summary; if **GitHub MCP** is used to open a PR, align the PR title with the first commit or the most representative `feat`/`fix` line.
