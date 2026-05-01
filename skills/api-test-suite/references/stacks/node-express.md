# API test suite: Node.js + Express (TypeScript)

## Discovery

- **Router** **files** `router.get/post/...` + mount path in `app.use('prefix', router)`.
- **Zod** **schemas** **in** `schema.ts` or **inline**—**align** **V** **category** with **known** **validation** **errors**.

## Test harness

- **supertest** + **exported** `app` **(no** **listen)**: `request(app).get('/api/...')` **+** **expect** **status** **+** **body**.
- **Integration** **with** **DB:** **use** **test** **DB** / **trans** **per** **test** **as** **in** **repo** `test:integration`.

## OpenAPI

- If **express-openapi** / **tsoa** / **hand**-written **spec**—**import** **paths** **+** **methods** from **spec** **or** **generate** **from** **code** **(note** **drift** **risk**).

## Newman

- **Export** **Postman** **or** **run** **newman** **with** **same** **env** as **Jest** **(baseUrl,** **auth).**

## Coverage

- **Map** **test** **file** **per** **route** **file** or **per** **resource** **folder**; **list** **gaps** for **H/V/A** **on** **each** **path**.
