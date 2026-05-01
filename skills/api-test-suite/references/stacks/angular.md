# API test suite: Angular (SPA)

## Discovery

- **api** **service** **classes** under `features/**/services/*.ts` with **HttpClient** **method+path**—**rg** for `get(` `post(` and **string** **literals** **or** **environment**-**based** **base** **+** **path**.

## In-repo options

- **HttpClientTestingModule** + **expectOne** for **unit** **tests** (mock **backend**); **not** full **HTTP** **stack**.
- **E2E** (Playwright) **vs** **separate** **API** **service** **repo** **—** same **as** **React** **for** **true** **HTTP** **coverage**.

## OpenAPI

- If **client** is **generated** from **OpenAPI** **(ng-openapi)**, **use** **the** **spec** **as** **source** **of** **truth** for **the** **server** **contract** **(may** **live** **in** **another** **package**).

## Report

- **Distinguish** **“** **client** **contract** **(mocked** **HttpClient** **)**” **from** **“** **server** **endpoints** **executed** **”** **in** the **matrix**.
