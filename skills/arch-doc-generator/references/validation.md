# Grounding and regeneration metadata

## Metadata block (append to each generated doc)

```markdown
---
generated: <ISO-8601 date, e.g. 2026-04-23>
prompt_summary: <one line from user request>
source_hint: <branch name, tag, or commit if known; else `working tree`>
evidence: <`git`, `ripgrep`, `read` of N files, or `manifests only`>
---
```

Adjust front matter style to match the repo (some teams use `<!-- -->` in Markdown instead).

## Validation checklist (before you finish)

1. **Commands** — install / test / run commands appear in a manifest, README, or CI; **or** they are called out as **unknown** with a request to the user to confirm.
2. **Paths** — file paths in the document were **read** or **listed**; no guessed paths to “`src/Service/Thing.cs`” without a directory check.
3. **Behavior** — “How X works” is backed by **at least** one hop of code or an official config for that stack; otherwise label **assumption** with question marks.
4. **Versions** — framework/runtime version comes from a lockfile or build file, not the training cutoff year.
5. **Secrets** — never paste real tokens, keys, or PII; redact in examples.
6. **Regeneration** — a future run should be able to **diff**; tell the user to re-run the skill after **major** refactors or to pin the commit in the metadata.

## If something cannot be proven

- Write **TBD** or **not found in tree** and suggest what to open or add (a test, an ADR, or a spec file).
