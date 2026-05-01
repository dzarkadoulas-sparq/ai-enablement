# Impact analysis: Next.js (App Router)

## Tracing

- **Files** under `app/` — **page** / **layout** / `route.ts` **imports**; **Server** **Components** **import** **graph** (no **client** **boundary** **without** **analysis**).
- **Server Actions** — **string**-bound **form** **actions**; **find** **`"use server"`** **file** **exports** **imported** **by** **client** **modules**.
- **Middleware** — **matcher** **config** **affecting** **same** **paths** as **target**.

## Dynamic

- **`revalidatePath` / `revalidateTag`** **string** **literals** referring to **routes** you **rename**.

## Cross-layer

- Changes in **`lib/`**, **`server/`** **used** by **both** **RSC** and **client**—**higher** **blast** **radius**; **list** **both** **sides**.

## Tools

- `next build` (or `tsc`) as **validation** of **import** **graph** for **TypeScript** **files**.
