# Secret and credential patterns (for grep / static review)

**Redact** matches in any user-facing report. Use only to **find** issues and point to **file:line** without pasting the secret.

## High-signal string patterns (examples)

- `-----BEGIN` (private keys, certs)
- `api_key`, `apikey`, `api-key`, `secret`, `client_secret`, `password=`, `token=`, `bearer ` (case variations)
- `ghp_`, `github_pat_`, `glpat-` (token prefixes)
- `AKIA` (AWS key id pattern start); `ASIA` (temp)
- Connection strings: `mongodb://`, `postgres://`, `Server=...;Password=`, `DefaultEndpointsProtocol=...;AccountKey=`

## File paths to scan (carefully)

- `**/.env` (often gitignored—**do not** commit; if found in index, **CRITICAL**)
- `**/.env.production`, `appsettings.Production.json` with real secrets
- `docker-compose*.yml` with env inline
- CI: `.github/workflows/*.yml` with hardcoded tokens (use secrets store)
- `**/*secrets*`, `**/*credentials*`, `**/id_rsa`, `**/*.pem`

## JSON / YAML

- `private_key`, `client_secret`, `apiKey` as **values** in committed files (not keys from well-known public docs)

## False positives

- **Example** placeholders: `replace_me`, `xxx`, `changeme`, `your-api-key` in templates—note as **low** / doc issue unless used in live env.

## Fix guidance (generic)

- **Rotate** if a real secret was ever committed; **git history** may retain it.
- **Use** env + secret manager; **pre-commit** hooks for secret scanning (gitleaks, etc.).
