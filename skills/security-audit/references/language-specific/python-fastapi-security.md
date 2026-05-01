# Python + FastAPI — API security focus

## Injection

- **SQL** — `text(f"...{user}")` or f-strings in SQL; use **bound parameters** / **ORM** with params.
- **SSTI** — Jinja or template engines with **user** content without sandbox.

## Auth

- **OAuth2** / **JWT** — **algorithm** from token validation (deny `none`); **secret** from env; **leeway** and **aud/iss** checks.
- **Depends** — **auth** on every **non-public** route; no **“optional”** auth that still exposes data for wrong user.

## Async + blocking

- **Blocking** I/O in `async` routes can enable **DoS** — `requests.get` in `async def` (flag; use httpx async).

## Files / uploads

- **Upload** path, **content-type** sniffing, **size**; **Path** traversal in stored filenames.

## Deserialization

- **pickle** / **yaml.unsafe_load** on untrusted input = **CRITICAL** if found.

## Tools

- `uv run pip-audit` (or **pip-audit**); `uv run bandit -r app` — **HIGH** threshold per policy; **Safety** if used in CI.
- **MyPy** does not cover security; **Ruff** security plugins if enabled.

## See also

`commands/python-fastapi/security-scan.md` in ai-enablement.
