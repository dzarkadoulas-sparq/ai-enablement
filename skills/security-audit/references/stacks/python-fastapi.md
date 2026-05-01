# Security audit: Python + FastAPI

## Read first

- `references/language-specific/python-fastapi-security.md`
- `commands/python-fastapi/security-scan.md` (ai-enablement)

## Dependency scan

- `uv run pip-audit` (or **pip** / **poetry** per repo). `uv run bandit -r app` (or `bandit` path from **pyproject** / CI).

## Code scope (typical)

- `app/routers`, `app/features`, `app/core` (auth), `pyproject.toml` / `requirements*`.

## Focus

- **Pydantic** for bodies; **Depends** auth; **raw SQL**; **pickle**; **async** + blocking; **file** uploads.
- **CORS** — `allow_origins` = **explicit** in prod, not `*` for credentialed use.

## Headers

- **Uvicorn** / **Gunicorn** **behind** Nginx/Traefik: verify **HSTS** / **CSP** at **edge** in prod.
