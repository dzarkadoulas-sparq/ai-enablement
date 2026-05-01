# ADR (Architecture Decision Record) shape

This follows a MADR- or Nygard-style **short** record. Titles in your repo can be numbered: `0001-choose-event-bus.md`, etc.

## Suggested file layout

- **Status:** `Proposed` | `Accepted` | `Superseded by <link>` | `Deprecated`
- **Context** — Problem, constraints, and what was known **from the codebase** (paths, versions).
- **Decision** — What the team (or the **current** code) does; one or two short paragraphs.
- **Consequences** — Positive, negative, and **tradeoffs** not resolved.
- **Alternatives** — Rejected options and **why** (or “TBD if reviving a planned migration”).

## If the “decision” is already implemented

- Cite the **key** files, packages, and config. If the history is not in `git log` in scope, write **observed in code** rather than a fake rationale.

## If the user is deciding something new

- Leave **Status: Proposed**; list open questions; avoid accepting until the user confirms.

## Links

- Add related ADRs, tickets, and PRs in a **Links** line at the bottom (optional).
