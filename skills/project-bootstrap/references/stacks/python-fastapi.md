# Stack: Python + FastAPI

## Detection

- `fastapi` in `pyproject.toml` or `requirements*.txt`.
- `uvicorn` (or `gunicorn` + Uvicorn worker) in dev instructions.

## Read first

`README.md`, `pyproject.toml` (or requirements), `app/main.py` or `src/.../main.py`, `app/core/config` or `settings`, Alembic (`alembic.ini`, `app/db`), `.env.example`.

## Trace one flow

OpenAPI: route decorator → dependency-injected `Depends` (auth, DB session) → **service** → **repository** → models. Note **Pydantic** models used as request/response DTOs.

## `CLAUDE.md` sections to generate

1. **Quick Reference** — `uv sync` / `poetry install`, `uvicorn` dev command, `pytest`, `ruff` / `mypy`, `alembic` from real files.
2. **Read first** — application entry, settings, migrations.
3. **Architecture** — routers → services → repositories; feature packages; no DB in route bodies.
4. **Code style** — Python version, Pydantic v2, `async` discipline, type hints, structlog or logging setup.
5. **Security** — CORS, dependency injection for secrets, `pip-audit` / `bandit` if in scripts, no default `DEBUG=True` in prod.
6. **Testing** — markers, fixtures, integration with Docker/DB.
7. **Pitfalls** — sync ORM in async routes, blocking I/O in async def, schema reuse between ORM and Pydantic where forbidden by team.
8. Conflict / stop-and-ask line.

## Stack red flags

- Raw SQL in routers.
- `Session` or `engine` as globals without lifecycle management.
- Missing health/readiness route when Kubernetes deploy is implied by repo.

## Exemplar (ai-enablement)

`claude.md/python-fastapi/CLAUDE.md`
