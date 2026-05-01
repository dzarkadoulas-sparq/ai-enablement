# API test suite: Python + FastAPI

## Discovery

- **OpenAPI** **=** **best** **source:** `GET` **/openapi.json` **in\*\* **dev** **or** **static** **export**; **or** **scan** `APIRouter` **include** **prefixes** in `main` **and** **routers** **under** `app/`.

## Test harness

- **httpx** `AsyncClient(app=app, **base_url="http://test")` **or** `TestClient` for **sync** **style**; **assert** **status** **+** **JSON** **(match** **Pydantic** **model** **or** **jsonschema**).
- **Markers** **—** `pytest -m "not integration"` for **fast** **matrix**; **full** **with** **DB** **in** `integration`.

## Schema

- **Pydantic** **models** **as** **response** **model** `response_model=...`—**derive** **expected** **4xx** **bodies** from **ValidationError** **handlers** **(per** **app** **pattern**).

## Postman

- **Import** **from** **OpenAPI** **(Postman** **import** **URL** **/ file)** **then** **add** **A/V** **examples**; **or** **generate** **collection** **from** **OpenAPI** **in** **CI** **(openapi2postman** **et** **al.)**—**if** **not** **in** **repo**, **suggest** **one**-time\*\* **import** **flow**.

## Coverage

- **Compare** **OpenAPI** **operationId** **/ path** list **to** **tests** **collected** **(pytest** **--collect-only** **+** **naming** **convention**).
