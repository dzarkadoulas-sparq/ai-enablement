# Test categories (per request / per endpoint)

Use a **shorthand** in the **report** and **in** test **names**.

| Code  | Category                  | What to assert |
| ----- | ------------------------- | ------------------------- |
| **H** | Happy / success           | 2xx, **valid** **body** **per** **schema** or **contract**; **Location** on **201** if **restful** **create** |
| **V** | Validation / client error | 400 or **422** (or **per** **API** **convention**); **error** **code** and **field** **errors** **for** **invalid** **input** (missing, type, range) |
| **A** | AuthN                     | 401 when **no** or **invalid** **token**; **correct** **challenge** (Bearer, cookie) per **server** |
| **Z** | AuthZ (forbidden)         | 403 (or 404 to **hide** **existence** if **API** **policy** **says** so—**document**) |
| **N** | Not found                 | 404 for **unknown** **id** (valid **format**); 400 for **malformed** **id** if **distinguishing** **policy** **exists** |
| **C** | Conflict / state          | 409, **version** **mismatch**, **duplicate** **(unique)**, **invalid** **transition** |
| **P** | Pagination / list         | `limit/offset` or `cursor` **—** first page, last page, **empty**; **abuse** **(limit** **too** **high)** if **capped** |
| **E** | Idempotency / concurrency | **Idempotency-Key** **replay** **(same** **key** **→** same **outcome)**; **If-Match** / ETag on **update** if **exposed** |
| **I** | Integration edge          | **Rate** **limit** **(429** **+** **Retry-After)** only if **documented** or **in** code |

## Error body shape

- **Standardize** on **one** project **format** (e.g. `{"errors":[{"path","message","code"}]}`, **RFC 7807** `problem+json`). **Test** the **shape** for **4xx/5xx** in **V/N/C** cases.

## Headers to assert (when API promises them)

- `Content-Type`, **pagination** `Link` or `X-Total-Count`, `ETag` on **GET** **if** **documented** **for** **caching** **/ concurrent** **edit** **safety**.

## Naming in code / Postman

- `it('returns 201 with order id and Location')` — **H**
- `it('returns 422 when amount is negative')` — **V**
- `it('returns 401 without Authorization')` — **A**
