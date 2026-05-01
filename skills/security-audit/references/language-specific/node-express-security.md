# Node.js + Express — API security focus

## Injection

- **SQL** / **NoSQL** — no string concat into **Prisma** raw, **TypeORM** query builder with user string, **find** with object injection; **$where** in Mongo if used.
- **Command** — `child_process` with user input; **path** join from user without **normalize** and **jail**.

## AuthN / AuthZ

- **JWT** — `alg: none`, **weak** secret, **long** expiry without refresh; **bearer** in **URL** (log leakage).
- **Session** — **httpOnly** + **secure** + **sameSite**; **regenerate** on login.
- **Routes** — every new route on **default-deny** or explicit **public** list; **role** checks on **mutations**.

## Input

- **Zod** / **Joi** at boundary; **content-type** and **size** limits (body-parser / raw limits).
- **file uploads** — type, size, path, **virus** scan if required by policy.

## Headers

- **Helmet** (or hand-rolled) — **CSP**, **HSTS**, **X-Content-Type-Options**; per `header-recommendations.md`.
- **CORS** — **origin** allowlist, not `*` with **credentials: true**.

## Tools

- `pnpm audit --audit-level=high`; **Semgrep** / **CodeQL** if in CI; **`npm run lint`** with security rules if configured.

## See also

`commands/node-express/security-scan.md` in ai-enablement.
