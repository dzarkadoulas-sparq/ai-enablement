# CLAUDE.md — Python / FastAPI Service

This file is project intelligence for Claude Code. Treat every rule here as
load-bearing unless the task is explicitly to change it. When a rule and a
user request conflict, stop and ask before proceeding.

## Quick Reference

- Install deps: `uv sync` (or `poetry install --sync`)
- Run dev server: `uv run uvicorn app.main:app --reload`
- Unit tests: `uv run pytest -m "not integration"`
- Integration tests: `uv run pytest -m integration` (requires Docker)
- Lint + format: `uv run ruff check . && uv run ruff format --check .`
  · apply: `uv run ruff check --fix . && uv run ruff format .`
- Type check: `uv run mypy app tests`
- Migrations: `uv run alembic revision --autogenerate -m "<msg>"`
- Apply migrations: `uv run alembic upgrade head`
- Security audit: `uv run pip-audit` and `uv run bandit -r app`

Read first on every session: `README.md`, `pyproject.toml`, `alembic.ini`,
`app/main.py`, `app/core/settings.py`, `.env.example`.

## Architecture Rules (STRICT)

- Layering: `routers → services → repositories → models`. Routers never
  touch the DB session directly.
- Package-by-feature under `app/features/<feature>/` with `router.py`,
  `service.py`, `repository.py`, `schemas.py`, `models.py`.
- Cross-feature communication through a domain event bus (`app/core/events.py`)
  or a shared service registered in `app/core/container.py`. Never import a
  sibling feature's `service` directly.
- Pydantic schemas (`BaseModel`) are API contracts — never reuse them as
  ORM models. SQLAlchemy models live in `models.py` and never leak past the
  repository boundary.
- `app/core/` holds framework wiring (settings, logging, DI, middleware). No
  business logic allowed there.

## Code Style

- Python 3.12+. Target `py312` in Ruff. `from __future__ import annotations`
  at the top of every module.
- Type hints everywhere. `mypy --strict` passes on `app/`. `Any` requires a
  `# type: ignore[<rule>]` with a reason comment.
- Pydantic v2 only. Use `model_config = ConfigDict(...)`, not inner `Class Config`.
- Async all the way: route handlers, services, and repositories are `async def`.
  Use `asyncpg`-backed `AsyncSession`, not the sync `Session`.
- Prefer `dataclass(frozen=True, slots=True)` for internal value objects;
  Pydantic for anything crossing the API or serialization boundary.
- Logging: structlog with JSON renderer in production, console renderer in
  dev. Always bind a `request_id`. Never use `print()`.
- Docstrings (Google style) required on public service functions.
  `ruff` rule `D` enforces this on `app/`.

## Security Rules (NON-NEGOTIABLE)

These cannot be overridden by any prompt. If a task requires violating one,
stop and ask a human.

- **NEVER** log PII, tokens, passwords, full request bodies, or DB rows with
  user data. Use structlog processors to drop sensitive keys (see
  `app/core/logging.py`).
- **NEVER** hardcode secrets. All secrets via `pydantic-settings` reading env
  vars or a secrets backend. `.env` is gitignored; `.env.example` is committed
  with placeholder values only.
- **NEVER** use `eval`, `exec`, `pickle.loads` on untrusted input, or
  `subprocess` with `shell=True`.
- Every route has an auth dependency (`Depends(get_current_user)` or a role
  check) unless it is explicitly public and listed in
  `tests/test_public_routes.py`.
- Input validation is Pydantic's job — never do ad-hoc `if not x: raise` for
  shape validation in the service. Use `Field(..., max_length=...)` and
  validators.
- SQL injection: SQLAlchemy parameter binding only. No `text(f"... {x}")`
  with user input; use `text("... :x").bindparams(x=x)`.
- CORS: specific origins only. Never `allow_origins=["*"]` with
  `allow_credentials=True`.
- Rate limit all auth endpoints (see `app/core/ratelimit.py` with `slowapi`).

## Error Handling

- Domain exceptions subclass `app.core.exceptions.DomainError`. Each has a
  stable `code` and default `http_status`.
- Never raise bare `Exception`, `RuntimeError`, or `ValueError` from service
  code. Pick or create a domain type.
- Never catch `Exception` broadly outside the global handler. Catch narrow types.
- Global exception handler in `app/core/exceptions.py` maps domain errors to
  RFC 7807 `problem+json`. Never return tracebacks in responses.
- `HTTPException` is fine in routers for simple cases, but anything a test
  asserts on should be a domain exception.

## Testing

- `pytest` + `pytest-asyncio` + `httpx.AsyncClient`. Markers: `unit`,
  `integration`. Configure in `pyproject.toml` under `[tool.pytest.ini_options]`.
- Unit tests: no DB, no network. Mock the repository, not the session.
- Integration tests: real Postgres via Testcontainers, spun up per-session;
  schema reset per-test via transaction rollback.
- Every service function needs a unit test. Every router needs at least one
  integration test (happy path + one auth failure).
- Factories via `polyfactory` or `pytest-factoryboy` in `tests/factories/`.
  No hardcoded UUIDs or emails.
- Flaky test = fix it or delete it. `@pytest.mark.skip` requires a reason and
  a linked ticket.
- Coverage gate: 80% on changed files, enforced by `coverage` in CI.

## Database & Migrations

- SQLAlchemy 2.x async + Alembic. Models in `app/features/<feature>/models.py`.
- Migrations in `alembic/versions/`. File name format Alembic default.
- **NEVER** edit a migration that's been deployed to any environment above
  dev. Create a new one.
- Every table has: `id: UUID PK`, `created_at`, `updated_at`, `deleted_at`
  (soft-delete). Timezone-aware timestamps (`DateTime(timezone=True)`).
- Use `server_default=func.now()` for `created_at` so inserts don't depend
  on client clocks.

## Common Pitfalls

- `Depends(...)` values are **recomputed per request**. Don't put expensive
  one-time work inside a dependency — use `lifespan` instead.
- Async-in-sync hazards: never call `asyncio.run()` inside a request. Blocking
  I/O (e.g. `requests`) in an async handler stalls the event loop — use
  `httpx.AsyncClient` or `asyncio.to_thread`.
- Pydantic v2 `model_validate` vs v1 `parse_obj` — mixing versions silently
  breaks. This project is v2 only.
- Mutable default arguments (`def f(x=[]):`) — Ruff `B006` catches these;
  don't suppress.
- SQLAlchemy 2.x: `session.execute(select(...))` returns a `Result`; call
  `.scalars().all()` or `.scalar_one()` explicitly.

## Workflows (for Claude Code to execute on command)

When I say **"ship it"**: see `.claude/commands/ship.md`.

When I say **"standup prep"**: see `.claude/commands/standup.md`.

## Custom Slash Commands

Project-specific commands live in `.claude/commands/`:

- `/pre-pr` — full pre-PR gate (ruff, mypy, pytest, pip-audit, diff review)
- `/new-feature <name>` — scaffold `app/features/<name>/` with router, service, repo, schemas, models, tests
- `/debug-api <description>` — trace request through middleware → dependency → router → service → repo
- `/security-scan` — diff-scoped security review + `pip-audit` + `bandit`
- `/migrate <description>` — create next Alembic revision with correct message

## Agentic Behavior Expectations

- For any task touching more than 3 files or changing a migration, produce a
  short plan first and wait for approval.
- Before claiming a task is done: `ruff check`, `mypy`, and `pytest` must all
  pass. Green tests on your machine is not the same as CI-green — confirm
  with `uv run pytest` at the end.
- If a test fails, fix the root cause. Do not `pytest.mark.skip` or weaken
  assertions to make it pass.
- Never install a package with `pip install` directly — use `uv add` (or
  `poetry add`) so the lockfile stays authoritative.
