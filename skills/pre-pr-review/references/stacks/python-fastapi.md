# Pre-PR gate: Python + FastAPI

## Baseline branch

`origin/develop` or repo default. `git diff origin/develop...HEAD`.

## Gate order

1. **Lint + format** — `uv run ruff check --fix .` then `uv run ruff format .`; re-run `ruff check` and `ruff format --check` (or `poetry run` if that’s the repo).
2. **Typecheck** — `uv run mypy app tests` (paths per project). No new `# type: ignore` without a reason.
3. **Unit tests** — e.g. `uv run pytest -m "not integration" --cov=app --cov-branch` (match `pyproject` / README).
4. **Integration tests** — `uv run pytest -m integration`. If Docker isn’t running, **stop and report** (do not skip).
5. **Coverage** — e.g. fail under threshold on changed lines (`coverage` / project script).
6. **Security** — `uv run pip-audit` (flag HIGH/CRIT); `uv run bandit -r app` (flag HIGH).
7. **Diff hygiene** — see **Diff hygiene (FastAPI)** below.
8. **Summary**. **Do not push or open PR** unless asked.

## Diff hygiene (FastAPI)

- `print()` in app code.
- `pytest` skips without reason + ticket.
- Commented-out blocks, hardcoded secrets/URLs/IPs.
- New routes without auth `Depends` unless public by design.
- Pydantic models: request bodies must use `BaseModel` (or project standard).
- `text(f"... {var}")` SQL with user input interpolation.
- Blocking I/O in `async def` handlers (e.g. `requests`, `time.sleep`).

## Exemplar

`commands/python-fastapi/pre-pr.md`
