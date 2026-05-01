# Impact analysis: Angular (v17+)

## Tracing

- **Standalone** **imports** in `@Component` / **route** `loadComponent` **lambdas** (dynamic **import** **paths**).
- **DI** — `inject(Token)`, **string**-based **providers** in **route** `providers: []`.
- **Barrels** — `index.ts` **in** **feature** **folders**.

## Dynamic

- **`loadChildren`** / **lazy** **route** **strings**; **i18n** **pipe** **keys** if **renaming** **exposed** **ids**.

## Templates

- **Selector** **string** in **other** **templates** **if** you **change** **component** **selector** (rare for **refactor**—flag **all** `app-foo` **usages**).

## Tools

- **Angular** **LS** / **ng** **schematic**-style **moves** when **available**; else **rg** **component** **class** **name** and **file** **path**.
