Run the full pre-PR gate for this .NET / ASP.NET Core service.

Stop at the first failing step and report; do not continue past a failure.

1. **Format + lint**
   - `dotnet format` then `dotnet format --verify-no-changes`.
   - If format rewrites files, keep those changes.

2. **Compile with warnings-as-errors**
   - `dotnet build -c Release -warnaserror --no-restore`.
   - A new warning on a changed file is a failure.

3. **Unit tests**
   - `dotnet test --no-build --filter "Category!=Integration" --collect:"XPlat Code Coverage"`.

4. **Integration tests**
   - `dotnet test --filter "Category=Integration"`.
   - If Docker isn't running, say so and stop — don't skip.

5. **Coverage gate**
   - Run `reportgenerator` (or the project's coverage tool) against the
     coverage output; enforce 80% on changed lines.

6. **Dependency vulnerabilities**
   - `dotnet list package --vulnerable --include-transitive`.
   - `dotnet list package --deprecated`.
   - Flag any HIGH/CRITICAL CVE on a dependency touched in this branch.

7. **Diff hygiene** — `git diff origin/develop...HEAD` and check for:
   - `Console.WriteLine` / `Debug.WriteLine` left in production code.
   - `[Fact(Skip = ...)]` / `[Theory(Skip = ...)]` without a linked ticket.
   - Commented-out code blocks.
   - Hardcoded URLs, IPs, connection strings, or values that look like secrets.
   - `#pragma warning disable` without a `// Reason:` comment on the line above.
   - Endpoints missing `[Authorize]` (and not `[AllowAnonymous]`).
   - Missing `FluentValidation.AbstractValidator<T>` for new request DTOs.
   - `FromSqlRaw` with string interpolation.

8. **Summary**
   - Commits on the branch + one-line description.
   - Changed-file count, grouped by project.
   - Gate status (PASS / FAIL + reason).
   - 2–3 bullet draft of the PR description.

Do not push, do not open a PR. This command only verifies the branch is ready.
