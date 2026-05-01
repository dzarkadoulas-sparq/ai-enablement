# API test suite: Next.js (App Router)

## Discovery

- **Route** **Handlers:** `app/**/route.ts` **—** **export** `GET`/`POST`/`PUT`/`PATCH`/`DELETE` (and **dynamic** `[[...params]]`).
- **List** **(method,** **path** **template)** from **file** **path** **convention** (`/api/...` **or** **rest** under `app/`).
- **Server** **Actions** are **not** always **HTTP**—**if** the **user** **wants** “API” **tests** **for** **actions**, **treat** **as** **form** **/ RPC** **with** **validation** **tests** in **unit** **layer** (or **E2E** **POST** to **form** **action**—**rare**).

## In-repo tests

- **Vitest** + `Request` / Next test utils (if the project uses `next/experimental/testmode` or a hand-rolled `Request` to the handler)—match the existing pattern in the repo.
- **Integration:** **start** `next` **in** **test** **or** use **Vercel**-style **preview** **URL**—**per** **team** **CI**.

## OpenAPI

- If **generated** **or** **checked** **in**, **use** it; **else** **manual** **table** from **file** **tree** under `app/`.

## Postman

- **Base** **URL** = **deploy** **preview** or **localhost:3000**; **only** **paths** that **are** **true** **Route** **Handlers**.
