# Pytest (FastAPI / async services)

## Layout

- `tests/` mirroring `app/` or colocated; **`conftest.py`** for `session`, `client`, `async_client` fixtures; follow repo.
- **Markers** — `pytest -m "not integration"` for fast runs; `integration` when Docker/DB is up (see `pre-pr` command).

## Async

- `pytest-asyncio` with **`async` tests**; **httpx** `ASGITransport(app=...)` for **FastAPI** in-process calls, or `TestClient` for sync-style if project standard.

## HTTP testing

- `httpx.AsyncClient` + `LifespanManager` if lifespan matters; or Starlette `TestClient` for simpler sync tests.
- Assert **status** + **Pydantic**-compatible body; invalid body → 422; auth → 401/403 as applicable.

## DB

- **Session-scoped** engine + **transaction rollback** per test, or **truncate** script; **alembic** to head before suite if `README` says so.

## Fixtures

- `faker` or factories in `tests/factories/`; keep **deterministic** seeds for snapshots if used.

## Parametrize

- `@pytest.mark.parametrize` for validation matrix (inputs → expected error codes).

## Coverage

- `pytest-cov` with same paths CI uses; `--cov-fail-under` only if team policy (see pre-PR stack).

## Anti-patterns

- `async` tests that **forget** `await` (coroutine not run) — use **pytest** plugins that warn or enable asyncio mode `auto`.
