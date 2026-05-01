# Impact analysis: Python + FastAPI

## Tracing

- **Imports** — `from app.features.x` (package **layout** under `app/`); **re-export** in `__init__.py`.
- **DI** **overrides** in **tests** (`app.dependency_overrides`).

## Dynamic

- **`getattr`**, **importlib** **imports**; **Celery** **task** **name** **strings**; **Pydantic** **discriminated** **unions** **by** **literal** **field** **value**.

## Router

- **APIRouter** **prefix** **in** `include_router`; **changing** **one** **prefix** **hits** **all** **registrations** **in** `main` / **feature** **routers**.

## Data

- **Alembic** **migrations**; **raw** **SQL** **in** **repositories**—**rg** **table** / **column** **names**.

## Tools

- `rg` / **ruff** **check**; **mypy** for **ref** **errors**; **pytest** **collection** **imports**.
