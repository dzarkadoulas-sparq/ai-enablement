# Framework / runtime upgrade (Next, React, Spring Boot, .NET, etc.)

## When to use

- **EOL** runtime, security patches only on a new major, or a planned org milestone (e.g. LTS .NET).

## Steps

1. Read the **vendor upgrade guide** (breaking changes list, codemods, required config).
2. **Pin** versions on a dedicated branch; prefer **one major risk axis** at a time (framework first vs. dozens of unrelated plugin bumps), when possible.
3. Run **official** codemods or CLIs (Next codemod, `dotnet format`, Angular schematics, etc.); **review** every automated diff—codemods are not always perfect.
4. Fix **type errors** in layers that fail first: often router, build config, then app code.

## Tests

- Match **CI** parity: same build, lint, and test commands as a PR. Add **E2E smoke** (routing, auth, one critical journey) when the upgrade touches the shell.

## Data / database

- ORM and framework steps sometimes require **migrations** at a specific point—Plan a **sub-PR** or sub-phase for Prisma, EF, Flyway, etc.

## Rollback

- **Lockfile** and version **pin** commits in one revert-friendly commit when team policy allows.

## Pitfalls

- **Mixing** a huge version bump with a **feature refactor** in one changeset—keep “bump only” in its own phase/PR so history is **bisect**-able.
