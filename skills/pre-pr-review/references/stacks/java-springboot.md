# Pre-PR gate: Java + Spring Boot

## Baseline branch

`origin/develop` or repo default. `git diff origin/develop...HEAD`.

## Gate order (Gradle / Maven: adjust to repo's wrapper)

1. **Format** — e.g. `./gradlew spotlessApply` then `./gradlew spotlessCheck` (or project formatter).
2. **Static analysis** — `./gradlew check` (Checkstyle, SpotBugs, PMD) — new warnings on **changed** files = fail (per project policy).
3. **Build + unit tests** — e.g. `./gradlew build -x integrationTest` (match README).
4. **Integration tests** — `./gradlew integrationTest`. If Docker isn’t running, **stop and report**.
5. **Dependencies** — `./gradlew dependencyCheckAnalyze` (or equivalent); flag HIGH/CRIT on **introduced** dependencies.
6. **Diff hygiene** — see **Diff hygiene (Spring)** below.
7. **Summary**. **Do not push or open PR** unless asked.

## Diff hygiene (Spring)

- `System.out.println` / `e.printStackTrace()` in non-test code.
- `@Ignore` / `@Disabled` without ticket.
- Commented-out code, unrelated file churn, debug configs.
- Hardcoded secrets/URLs/IPs.
- New endpoints without `@PreAuthorize` (or equivalent) if project requires it.
- DTOs without `@Valid` and validation where standard.
- Raw SQL outside the project’s allowed area (e.g. reporting package only).

## Exemplar

`commands/java-springboot/pre-pr.md`
