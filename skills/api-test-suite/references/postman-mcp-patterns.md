# Postman MCP (optional)

Use when the user wants **Postman** as the **execution** or **storage** layer for the API test suite. If **MCP** is **unavailable**, **degrade** gracefully: **export** a **v2.1** **Collection** **JSON** and **document** how to **import** it.

## Official resources

- **Postman Learning** — [MCP servers overview](https://learning.postman.com/docs/postman-ai-developer-tools/mcp-servers/overview) and [Postman MCP Server](https://github.com/postmanlabs/postman-mcp-server) (remote **OAuth** URL, local **STDIO** package, **tool** **sets**).
- **Remote** **server** (typical): `https://mcp.postman.com/mcp` (full), `.../minimal` (faster), `.../code` (code-focused). **EU** **region** **variants** if **org** is **in** **EU** (often **API** **key** **only** — see **current** **Postman** **README**).
- **Cursor** / **Claude** **Code** — add as **HTTP** (streamable) **MCP** **in** `mcp.json` / `claude mcp add` per **host** **docs** (see project **root** `README` **MCP** **table**).

## What to ask the MCP (when tools exist)

- **List** **workspaces** and **collections**; **get** or **search** a **collection** by **name**.
- **Create** **/ update** **requests** with **method, URL, headers, body**; **folder** by **resource**.
- **Set** **environment** **variables** for **baseUrl**, **bearer** **token** **(masked** in **logs)**, **tenant** **id**.
- **Run** **if** the **MCP** **exposes** **a** **runner**; **else** **use** **Postman** **CLI** **(newman)** **or** **in-app** **Run** (document **command**: `npx newman run collection.json -e env.json`).

## Conventions in collections

- **Name** `{{method}} {{path}} — {{case}}` e.g. `POST /orders — validation missing amount`.
- **Prerequest** for **auth** only when **sweeping** many **requests**; **per-folder** **token** **refresh** **if** **short** **lived** **access** **token**.
- **Tests** **tab** — `pm.test("status 422", () => pm.response.code === 422)` + **body** **JSON** **schema** or **key** **presence**.

## No MCP

- Output a **portable** **Newman**-runnable **collection** + **example** **environment** with **`{{var}}` placeholders**; **no** real **secrets** in **repo**.
