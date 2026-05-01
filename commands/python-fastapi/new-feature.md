Scaffold a new feature package. Feature name: `$ARGUMENTS`.

Follow the project's package-by-feature convention. Use the existing
`users` feature as the reference — copy patterns and names exactly.

1. **Create** `app/features/<feature>/` with:
   - `__init__.py` — exports the public router only.
   - `router.py` — `APIRouter(prefix="/<feature>s", tags=["<feature>"])`,
     each route with an auth `Depends(get_current_user)` and a response model.
   - `schemas.py` — Pydantic v2 request/response models
     (`model_config = ConfigDict(from_attributes=True)` for read models).
   - `service.py` — async functions. No direct DB session use; depend on
     repository functions.
   - `repository.py` — async SQLAlchemy functions, each taking
     `session: AsyncSession` as the first arg.
   - `models.py` — SQLAlchemy `DeclarativeBase` model.
   - `exceptions.py` — `<Feature>NotFoundError(DomainError)` etc.

2. **Wire the router** in `app/main.py` under the versioned prefix
   (`/api/v1`).

3. **Create the Alembic migration**

   ```
   uv run alembic revision --autogenerate -m "create <feature> table"
   ```

   Review the generated script — autogenerate is a starting point, not gospel.
   Add any indexes, constraints, or backfills it missed. Do NOT
   `alembic upgrade head` — leave it to the developer.

4. **Create tests**
   - `tests/features/<feature>/test_service_unit.py` — pytest, async,
     repository mocked, no DB.
   - `tests/features/<feature>/test_router_integration.py` — `httpx.AsyncClient`
     - real Postgres via Testcontainers, happy path + one auth failure.
   - `tests/factories/<feature>.py` — `polyfactory` factory. No hardcoded UUIDs.

5. **Verify**
   - `uv run ruff check --fix . && uv run ruff format .`.
   - `uv run mypy app` — zero errors on the new files.
   - `uv run pytest tests/features/<feature>/` — all green.

6. **Summary** — list every file created, Alembic revision filename,
   next manual steps (add to OpenAPI tags if custom, wire rate limiter
   if public-facing, add to public-routes test if public).

Respect CLAUDE.md rules: no Pydantic model reuse as ORM model, no direct
import of another feature's `service.py`, routers never touch the DB
session directly.
