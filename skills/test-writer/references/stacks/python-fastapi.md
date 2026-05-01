# Test writer: Python + FastAPI

## Language references

- `references/python-pytest.md` — async, httpx/ASGI, markers, conftest.
- `references/edge-case-catalog.md`, `references/anti-patterns.md`

## Detect / locate tests

- `tests/` or `app/**/test_*.py`; `pytest.ini` / `pyproject.toml` `[tool.pytest]`.
- **Async** routes → **async** tests with `pytest-asyncio` mode the repo uses.

## Commands

- Unit: e.g. `uv run pytest -m "not integration" --cov=app` (match README).
- Integration: `uv run pytest -m integration` when scope requires DB.

## Focus

- **Pydantic** validation error matrix; **auth** `Depends` with test overrides.
- **Service/repository** unit tests with fakes; **API** tests with in-process client.

## Exemplar context

`templates/python-fastapi/CLAUDE.md`
