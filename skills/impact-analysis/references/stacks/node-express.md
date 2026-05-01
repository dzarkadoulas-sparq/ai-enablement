# Impact analysis: Node.js + Express (TypeScript)

## Tracing

- **Router** **mounts** in `app.use` / `router` **chains**; **import** of **controller** / **service** / **schema**.
- **Zod** / **shared** **types** **imported** **by** **multiple** **features**.

## Dynamic

- **Route** **path** **strings** in **tables** or **config**; **`req.params`** **accessors** with **string** **param** **names**.

## Data

- **Prisma** / **ORM** **model** **rename**—**migrations** + **every** **query** **file**; **`$queryRaw`** **string** **search**.

## Cross-service

- If **HTTP** **client** **calls** **this** **service**’s **paths**, **search** **repo** for **path** **prefix** or **OpenAPI** **tags**.

## Tools

- `rg` **symbol**; **tsc**; **integration** **test** **files** under **`test:integration`**.
