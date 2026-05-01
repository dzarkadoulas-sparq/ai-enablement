# Refactor guide: Python + FastAPI

## Verify

- `uv run ruff check .`, `uv run mypy app tests` (or project paths), `uv run pytest` (markers per README); add **integration** when the phase touches the DB or HTTP app object.

## Refactor-specific notes

- **Routers** — `include_router` prefix + **feature** `router` tags; **OpenAPI** **path** and **tag** **changes** affect **docs** and **generated** **clients**.
- **Pydantic** **v2** **models** — field renames are **breaking** for **API**; version or **deprecate** in **stages** if public.

## Pitfalls

- **Async** vs **sync** I/O: moving code between `async` and sync layers can **block** the **event** **loop** or **change** **semantics** under load.
