# Multi-step API workflow templates

Parametrize **base URL**, **tokens**, and **ids** with **env** or **test** **fixtures**. Add **assertions** on **each** **step** before **chaining**.

## 1) Auth: token acquisition

- **Client** **credentials** or **password** **(test** **user)** `POST /oauth/token` or `POST /auth/login`
- **Extract** `access_token` (and `refresh_token` if needed) into **var**
- **Next** **requests**: `Authorization: Bearer {{access_token}}`

**Negative:** wrong **client** **secret** → 401/400; **expired** **token** on **subsequent** **call** (optional **separate** **test** with **artificial** **expired** or **short** **TTL** in **test** **env**).

## 2) CRUD lifecycle (resource)

- `POST /{resource}` → 201, **store** `id` from **body** or **Location**
- `GET /{resource}/{id}` → 200, **same** **shape** as **create** **response**
- `PATCH /{resource}/{id}` → 200/204, **read** again **or** assert **eTag**
- `DELETE /{resource}/{id}` → 204
- `GET` **same** `id` → 404 (if **hard** **delete**)

## 3) Idempotent create / upsert (if API supports)

- `POST` **with** **Idempotency-Key** `A` → 201, **id** = X
- `POST` **with** same **key** and **same** **body** → 200/201 **idempotent** (same id X, **or** 409 if **key** **conflict** with **different** **body**—**per** **API** spec)

## 4) Concurrency (optimistic lock)

- `GET` with **ETag**; `PATCH` **with** `If-Match: "etag1"` **→** 200; **concurrent** **second** `PATCH` **with** same **stale** **ETag** → 412 or **per** spec

## 5) Search / list (pagination)

- `GET` **list** with **default** page **size**; **assert** `items.length <= limit`
- **Empty** list **(no** **data)**; **page** **beyond** **end** (empty or 404—**as** spec)

## 6) State machine (order / job)

- `POST` **create** **(draft)**
- **Allowed** **transition** `POST .../submit` **→** **in_progress**
- **Disallowed** **(skip** **step**)` → **C** (409) **or** **C**-style **error**
- **Webhooks** (if **in** **scope**): **mock** **receiver** **or** **poll** `GET /jobs/{id}`

## 7) File upload (when supported)

- **Multipart** **POST**; **file** too **large**; **bad** **MIME** **→** V **or** 413

## Reporting

- **Each** **workflow** **=** one **“scenario”** **in** the **report**; **list** **steps** and **P0** **assertions** **covered** **or** **gaps** **(e.g. no** **412** **test** **because** **no** **ETag** **in** **MVP**).
