# Merge strategies and conflict mechanics

## Merge vs rebase: whose is “ours”?

| Operation | `ours` | `theirs` |
| --------- | ------ | -------- |
| **`git merge`** (on **your** branch) | **Current branch** (the one you had checked out) | **Incoming** branch you merged in |
| **`git rebase`** (replaying your commits) | The **commit you’re rebasing onto** (usually **upstream**) | The **commit being replayed** (**your** patch) |

**Practical rule:** in **rebase**, the **deeper** / **first-parent** context (often **upstream** mainline) is **not** “your feature” in the hunk’s labels—**read** the files under `HEAD` and **incoming** in the hunk, not only the label text.

## Reading conflict markers

```
<<<<<<< ours / HEAD / branch name
  content for one side
=======
  content for the other side
>>>>>>> their commit subject or hash
```

- **Three-way** (default with merge) uses **common ancestor**; **diff3** config shows **in-between** the common base:

```text
git config merge.conflictStyle diff3
```

Use **base** to build a result that is **syntactically** the merge of both edits when both are valid.

## Trivial patterns

- **Imports** on both sides — **merge** import lists, **dedupe**, sort as **eslint** / **editorconfig** would.
- **Whitespace**—prefer **one** style (project’s **formatter**); run **format** after resolution when safe.
- **One side** added **end** of file, **other** start—concatenate in **order** per **chronology** of commits if it matters, else top-to-bottom.

## Complex patterns

- **API** / **type** / **function signature** changed on **both** — **unify** to a single signature; may need a **new** name or **overload**; check **call sites** in **non**-conflict files.
- **Rename** on one side, **edit** on other—may need `git add` of **old** and **new** (Git will guide); **read** `git status` renames.
- **Binary** (images, `*.pyc`, some pdfs) — do **not** hand-merge. Prefer **`git checkout --ours` / `--theirs`** for one file after **team** agreement, or **re-export** the asset. **Document** in the report.

## Lockfiles and generated files

- **package-lock.json** / **pnpm-lock.yaml** / **poetry.lock** / **gemfile.lock**: **do not** merge by **typing**. Prefer: **pick one** side, run **`install`** to **regenerate**, **commit** the new lock. Note **policy** in `CONTRIBUTING` if it says **“ours wins”** then **`npm i`**.

## `git log` for intent (complex)

- List commits on **each** side that touched the file:

  `git log --oneline <merge-base>..<branch1> -- path`
  `git log --oneline <merge-base>..<branch2> -- path`

- **Show** the hunk: `git show <commit> -- path`

- Look for: **JIRA/ticket** in message, **revert** chains, **rename** commits, **BREAKING** notes.

## Checkout strategies (when appropriate)

- Single-file take one side:  
  `git checkout --ours -- path` / `git checkout --theirs -- path` (meaning depends on **merge** vs **rebase**; verify **which** content you get with `cat` before `git add`).

- Use only when the **entire** file is correct on one side.

## After resolution

- `git add` each path; **no** conflict markers must remain.
- `git diff --check` to catch **trailing** conflict **artifact** or **whitespace** errors in some projects.

## Escalation

- **Submodules** in conflict: follow **submodule** workflow; may need `git submodule update` in parent after inner resolution.
- **Multiple** nested conflicts: resolve **innermost** files first, then **continue** the outer operation.
